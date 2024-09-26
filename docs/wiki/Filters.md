# Using Filters
The uftrace tool supports various filters to help users find interesting functions more easily.  Any sizable program produces a vast amount of trace data when using uftrace naively.  This also hurts performance significantly. Therefore, appropriate filters are essential to narrow output scope.  For this reason, it is generally better to apply filters during record time but almost all the filters can also be applied for other commands such as replay, report, graph, script and dump too.  But in this document I'll use filtering at replay since I think it's better for understanding.

## Depth filter
I'd like to start with the depth filter.  The depth filter limits the function call depth so only higher level functions will be shown.  It'll help you to understand the structure of the program easily.  I'd recommend using depth filter with a small number (maybe 5?) when you trace a program first time.

Let's see a simple example.

```
$ cat fib.c
#include <stdlib.h>

int fib(int n)
{
  if (n <= 2)
    return 1;

  return fib(n-1) + fib(n-2);
}

int main(int argc, char *argv[])
{
  int n = 5;

  if (argc > 1)
    n = atoi(argv[1]);

  fib(n);
  return 0;
}

$ gcc -pg -o fib fib.c
```

Now run the `uftrace record` with the 'fib' program.  As I said you can set the depth filter for uftrace record to reduce the data size.  Also it'd be better to record arguments of the fib function, please see [Tutorial: Arguments](https://github.com/namhyung/uftrace/wiki/Tutorial:-Arguments) for details.

```
$ uftrace record -A fib@arg1 fib 10

$ uftrace replay -D 3
# DURATION    TID     FUNCTION
   1.286 us [25706] | __monstartup();
   0.803 us [25706] | __cxa_atexit();
            [25706] | main() {
   1.074 us [25706] |   atoi();
            [25706] |   fib(10) {
  18.171 us [25706] |     fib(9);
  10.762 us [25706] |     fib(8);
  29.345 us [25706] |   } /* fib */
  31.409 us [25706] | } /* main */
```

In the above example, the output only shows up to 3 level in the function call depth.  Note that the fib function was called 109 times but deeper calls were omitted from the output.

```
$ uftrace report
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================================
   31.409 us    0.990 us           1  main
   29.345 us   29.345 us         109  fib
    1.286 us    1.286 us           1  __monstartup
    1.074 us    1.074 us           1  atoi
    0.803 us    0.803 us           1  __cxa_atexit
```

You can also use the depth filter for the report command.  Then it will show data up to depth of 3 only.

```
$ uftrace report -D3
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================================
   31.409 us    0.990 us           1  main
   29.345 us    1.195 us           3  fib
    1.286 us    1.286 us           1  __monstartup
    1.074 us    1.074 us           1  atoi
    0.803 us    0.803 us           1  __cxa_atexit
```

As you can see, the number of calls is changed to 3.

## Function filter
Next type of filter works for name of function.  If you want to focus on a (set of) function, pass the name or a pattern with function filter.  There're two types of function filter - inclusive and exclusive.  For explanation, let's consider another example below:

```
$ cat hello.cpp
#include <iostream>
using namespace std;

int main()
{
  cout << "Hello C++" << endl;
  return 0;
}

$ g++ -pg -o hello hello.cpp
```

The inclusive filter is to filter execution of the given function.  Functions called out of the function will not be traced.  In the above example, normal output without a filter will look like below:

```
$ uftrace record hello
Hello C++

$ uftrace replay
# DURATION    TID     FUNCTION
            [26181] | _GLOBAL__sub_I_main() {
            [26181] |   __static_initialization_and_destruction_0() {
 125.297 us [26181] |     std::ios_base::Init::Init();
 130.521 us [26181] |   } /* __static_initialization_and_destruction_0 */
 131.267 us [26181] | } /* _GLOBAL__sub_I_main */
            [26181] | main() {
  17.510 us [26181] |   std::operator <<();
  18.502 us [26181] |   std::basic_ostream::operator <<();
  37.738 us [26181] | } /* main */
   2.665 us [26181] | std::ios_base::Init::~Init();
```

As you can there're many functions executed automatically.  If you want to trace functions you wrote under 'main' function, you could use `-F` option for the inclusive filter.

```
$ uftrace replay -F main
# DURATION    TID     FUNCTION
            [26181] | main() {
  17.510 us [26181] |   std::operator <<();
  18.502 us [26181] |   std::basic_ostream::operator <<();
  37.738 us [26181] | } /* main */
```

The exclusive filter does the opposite.  You can see functions not called from 'main' with `-N` option.

```
$ uftrace replay -N main
# DURATION    TID     FUNCTION
            [26181] | _GLOBAL__sub_I_main() {
            [26181] |   __static_initialization_and_destruction_0() {
 125.297 us [26181] |     std::ios_base::Init::Init();
 130.521 us [26181] |   } /* __static_initialization_and_destruction_0 */
 131.267 us [26181] | } /* _GLOBAL__sub_I_main */
   2.665 us [26181] | std::ios_base::Init::~Init();
```

Note that you can use regular expressions (regex) to specify functions you want.  The uftrace detects any special character used for regex in the option argument, then try to match function name using the pattern.  Otherwise, it should match to full function name.  For example, following example shows functions in the 'std' namespace (or functions name started with 'std').

```
$ uftrace replay -F ^std
# DURATION    TID     FUNCTION
 125.297 us [26181] | std::ios_base::Init::Init();
  17.510 us [26181] | std::operator <<();
  18.502 us [26181] | std::basic_ostream::operator <<();
   2.665 us [26181] | std::ios_base::Init::~Init();
```

The `^` character is a special character used by regex, and it matches to the start of a function name.  It could be useful for C++ programs to filter by namespace or class which has same prefix.


## Time filter
Sometimes it might be hard to identify specific functions or depth to filter.  Or you just want to exclude noise from uninterested functions.  The time filter can be handy for this case since it can omit functions whose execution time is less than a given value.

Here is a new example:
```
$ cat usleep.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#define SIZE  1000

void foo(void)
{
  char *buf = malloc(SIZE);
  FILE *fp = fopen("/dev/null", "r+");
  fread(buf, SIZE, 1, fp);
  usleep(1000);
  fwrite(buf, SIZE, 1, fp);
  fclose(fp);
  free(buf);
}

int main(void)
{
  foo();
  return 0;
}

$ gcc -pg -o usleep usleep.c
```

The 'foo' function does many things and say you really don't want to see all the details.  Maybe what you want to know is what makes it run so slow.  If you have some standard regarding to the execution time, you can set a time filter to find out where it goes slow.  Let's assume that you want to find functions run more than 1 milli-second.  Then you can use following command line to do it:

```
$ uftrace -t 1ms usleep
# DURATION    TID     FUNCTION
            [26536] | main() {
            [26536] |   foo() {
   1.066 ms [26536] |     usleep();
   1.102 ms [26536] |   } /* foo */
   1.103 ms [26536] | } /* main */
```

You need to give time unit properly - allowed ones are "ns", "us", "ms", "s", "m" and "h" for nano-, micro-, milli-, seconds, minutes and hours respectively.


## Advanced usage
The above filters can be used at the same time.  For example, following command is to trace Node.js JavaScript engine focusing on the initialization phase.  It shows maximum depth of 3 in the 'node::Init' function excluding small functions run less than 10 micro-seconds.

```
$ uftrace replay -F node::Init -D 3 -t 10us -d node.data
# DURATION    TID     FUNCTION
            [ 9738] | node::Init() {
            [ 9738] |   uv_default_loop() {
  58.674 us [ 9738] |     uv__loop_init();
  59.437 us [ 9738] |   } /* uv_default_loop */
  19.952 us [ 9738] |   uv_disable_stdio_inheritance();
            [ 9738] |   v8::V8::SetFlagsFromString() {
  17.059 us [ 9738] |     v8::internal::FlagList::SetFlagsFromString();
  23.955 us [ 9738] |   } /* v8::V8::SetFlagsFromString */
            [ 9738] |   v8::Isolate::New() {
 641.705 us [ 9738] |     v8::internal::Isolate::Isolate();
 658.744 us [ 9738] |   } /* v8::Isolate::New */
            [ 9738] |   v8::Isolate::Scope::Scope() {
  57.548 us [ 9738] |     v8::Isolate::Enter();
  57.874 us [ 9738] |   } /* v8::Isolate::Scope::Scope */
 876.860 us [ 9738] | } /* node::Init */
```
