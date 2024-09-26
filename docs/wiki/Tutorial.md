# Basic usage
The uftrace tool consists of several subcommands.  We'll see how to use them with simple examples in this document. Note that this document is still a work in progress and is based on the behavior of the latest version of uftrace.

## Slide Tutorial
It is recommended to read the [slide tutorial](https://uftrace.github.io/slide) as a first step as it includes up-to-date features.

## Getting started
The first subcommand to look at is `live`.  It's a default subcommand and will be used if you don't specify a different subcommand when running uftrace ("uftrace xxx" and "uftrace live xxx" are equivalent commands).  It's basically same as running `record` and then `replay` subcommands in a row.  That means it'd show the output after a program (given on the command line) finishes.  Below is the familiar "hello world" program.
```c
$ cat hello.c
#include <stdio.h>

int main(void) {
  printf("Hello world\n");
  return 0;
}
```
To analyze this program with uftrace, you need to enable the compiler instrumentation.  The gcc provides `-pg` and `-finstrument-functions` options for this.  We prefer to use `-pg` option as it's more light-weight in terms of compiler optimization, but in some cases it has not worked well.  Currently function arguments are only accessible if compiled with the `-pg` option.
```
$ gcc -o hello -pg hello.c
```
Now you can run the hello program with uftrace as shown:
```
$ uftrace hello
Hello world
# DURATION    TID     FUNCTION
   1.337 us [26639] | __monstartup();
   0.897 us [26639] | __cxa_atexit();
            [26639] | main() {
   6.696 us [26639] |   puts();
   7.582 us [26639] | } /* main */
```
The first line is, as expected, the output from the hello program.  The following lines are from uftrace and show function execution time, process/thread id and function names.  As with the function graph tracer in the Linux kernel, the functions are shown as source code of the C programming language.  You can see that the "puts" function was called during "main" instead of during "printf".  This is an optimization by the compiler (gcc) to replace it when the format string has no conversion specifier (e.g. %d) and ends with a new line character.

Note that it showed "puts," which is a library function not written by the user (the same goes for "__monstartup" and "__cxa_atexit") as well as your own functions in the program.  Of course it doesn't show the internals of "puts" but it can be useful to see how long "puts" takes.  It uses a technique called "PLT hooking" which redirects functions called from a dynamically-linked program.  While it's very powerful (so it's enabled by default), it comes with overhead.  If you don't want to see them and/or don't want to take the overhead, you can use `--no-libcall` option for `uftrace record` to disable it.  As `live` subcommand can take all options the `record` takes, you can use the following command:
```
$ uftrace --no-libcall hello
Hello world
# DURATION    TID     FUNCTION
   7.534 us [26714] | main();
```
Now you can only see the "main" function - the only one you wrote. The "Hello world" in the output shows that it actually calls the "puts" function but it didn't show up in the output from uftrace.  Also, you may notice that the "main" function is shown on a single line now.  This is because uftrace shows leaf functions which don't call other functions (from uftrace's perspective) in the compact format.

## Recording trace data
Uftrace needs to collect trace data in order to analyze the program execution.  Many subcommands in uftrace require this data.  The `record` subcommand saves the trace data in the "uftrace.data" directory by default, where other subcommands can use it.  You can use `-d` or `--data` option to use different a name.

Note that you should give uftrace options before the (path of) your program.  Uftrace stops processing command line options when it sees a non-option argument and passes the rest to the program as is.

As you already know, you can give the program a name on the command line.
```
$ uftrace record pwd
uftrace: /home/honggyu/work/uftrace/cmds/record.c:1618:check_binary
 ERROR: Can't find 'mcount' symbol in the '/bin/pwd'.
        It seems not to be compiled with -pg or -finstrument-functions flag
        which generates traceable code.  Please check your binary file.
```
This shows the error message and exits.  This is because the program (pwd) was not built with the compiler instrumentation so there's nothing uftrace can trace.  However you might remember that it can also trace library functions.  If you want to trace calls to library functions even though the program itself was not built to be traced, you can use `--force` option:
```
$ uftrace record --force pwd
/home/honggyu/work/uftrace
```
The output would be a series of library calls which look like a very simple version of ltrace.  You can use the `replay` subcommand to see the recorded program execution.
```
$ uftrace replay
# DURATION     TID     FUNCTION
   1.580 us [ 10536] | getenv();
   0.907 us [ 10536] | strrchr();
  80.071 us [ 10536] | setlocale();
   2.207 us [ 10536] | bindtextdomain();
   1.480 us [ 10536] | textdomain();
   0.894 us [ 10536] | __cxa_atexit();
   8.097 us [ 10536] | getopt_long();
   7.600 us [ 10536] | getcwd();
  15.993 us [ 10536] | puts();
   1.170 us [ 10536] | free();
   1.243 us [ 10536] | __fpending();
   0.890 us [ 10536] | fileno();
   0.920 us [ 10536] | __freading();
   0.154 us [ 10536] | __freading();
   0.984 us [ 10536] | fflush();
   2.417 us [ 10536] | fclose();
   0.180 us [ 10536] | __fpending();
   0.144 us [ 10536] | fileno();
   0.147 us [ 10536] | __freading();
   0.126 us [ 10536] | __freading();
   0.270 us [ 10536] | fflush();
   1.094 us [ 10536] | fclose();
```
You can also record function arguments and return values with `-a` or `--auto-args` option.
```
$ uftrace record --force -a pwd
/home/honggyu/work/uftrace
```
The replay output displays the recorded functions with arguments and return values.
```
$ uftrace replay
# DURATION     TID     FUNCTION
 314.286 us [ 10562] | getenv("POSIXLY_CORRECT") = "NULL";
   1.590 us [ 10562] | strrchr("pwd", '/') = "NULL";
  96.312 us [ 10562] | setlocale(LC_ALL, "") = "en_US.UTF-8";
   2.743 us [ 10562] | bindtextdomain("coreutils", "/usr/share/locale");
   1.670 us [ 10562] | textdomain("coreutils");
   1.010 us [ 10562] | __cxa_atexit();
   2.770 us [ 10562] | getopt_long(1, 0x7fff78bb8568, "LP") = -1;
   7.624 us [ 10562] | getcwd(0, 0) = "/home/honggyu/work/uftrace";
  15.960 us [ 10562] | puts("/home/honggyu/work/uftrace") = 27;
   1.283 us [ 10562] | free(0x19ea200);
   1.470 us [ 10562] | __fpending();
   1.203 us [ 10562] | fileno(&_IO_2_1_stdout_) = 1;
  11.760 us [ 10562] | __freading();
   0.177 us [ 10562] | __freading();
   1.040 us [ 10562] | fflush(&_IO_2_1_stdout_) = 0;
   2.663 us [ 10562] | fclose(&_IO_2_1_stdout_) = 0;
   0.210 us [ 10562] | __fpending();
   0.306 us [ 10562] | fileno(&_IO_2_1_stderr_) = 2;
   0.187 us [ 10562] | __freading();
   0.137 us [ 10562] | __freading();
   0.364 us [ 10562] | fflush(&_IO_2_1_stderr_) = 0;
   1.150 us [ 10562] | fclose(&_IO_2_1_stderr_) = 0;
```
Basically uftrace will save the trace of every single function call (and return).  However, this can be impossible for long-running and/or heavy-weight programs.  So, uftrace provides a couple of filtering options to control the trace data.  Although some of the filtering can work at later processing (like replay), it'd be better to reduce the amount of data at record time.

One is function-level filters and works on the name of functions.  You can use `-F` or `--filter` option to specify a function to trace.  With this option, all functions called during the function will be recorded and *NO* functions called outside of the function will be recorded.  Also there are `-N` or `--notrace` option to do the opposite.  All functions called during the function will *not* be recorded, and all functions called outside of the function *will* be recorded.  You can also use these options together and more than once.  In the hello world example, you can use it to see the function called under "main" only.
```
$ uftrace -F main hello
Hello world
# DURATION    TID     FUNCTION
            [27132] | main() {
   6.522 us [27132] |   puts();
   8.744 us [27132] | } /* main */
```
The next option is a function-depth filter.  You can use `-D` or `--depth` to limit function call depth to be recorded (or replayed).  The below example applies a depth-1 filter which shows functions only called in a top level:
```
$ uftrace -D 1 hello
# DURATION    TID     FUNCTION
   1.630 us [27091] | __monstartup();
   0.917 us [27091] | __cxa_atexit();
   7.493 us [27091] | main();
```
The last type of filter is time-based.  It will show functions running longer than the specified time.  Usually short-running functions are not the target of program execution analysis so it's useful to remove these function in one go.  You can use `-t` or the `--time-filter` option as shown:
```
$ uftrace -t 5us hello
# DURATION    TID     FUNCTION
            [27154] | main() {
   5.027 us [27154] |   puts();
   6.280 us [27154] | } /* main */
```
The above shows functions which run longer than 5 micro-second (us).  Note that the output when using time filter can vary for each run due to various timing issues.  You can also give other units like ns, ms, s or m for nano-second, milli-second, second and minute (respectively).  If you omit the unit, it defaults to "ns".  Please do not put a whitespace between the number and the unit.

In addition, it can also access function arguments and return value.  You can use the `-A` or `--argument` option to access the arguments and likewise, `-R` or the `--return` option for return value.  (Currently) it needs to pass the function name and argument/return value specifier(s).
```
$ uftrace -A puts@arg1/s -R main@retval hello
Hello world
# DURATION    TID     FUNCTION
   2.619 us [27961] | __monstartup();
   2.014 us [27961] | __cxa_atexit();
            [27961] | main() {
   8.256 us [27961] |   puts("Hello world");
   9.996 us [27961] | } = 0; /* main */
```
The first argument of "puts" function is the string so it needs to add the "/s" format specifier at the end.  By default integer type is assumed so retval has no format specifier.  For more information please refer to the manual page.

## Replaying the trace
Once you recorded the trace data of your program, you can use it to see the execution of the program.  As uftrace saved all the information to replay the trace, it's also easy to do it on a different machine by simply copying the data directory (uftrace.data) and running `uftrace replay` on it.  Most of the filtering (except the time filter) also works for replay.

Let's take a look at the following silly program.
```c
$ cat foobar.c
#include <pthread.h>

void *bar(void) {
  return NULL;
}

void *foo(void *unused) {
  return bar();
}

int main(int argc, char *argv[]) {
  pthread_t th;

  foo(argv);
  pthread_create(&th, NULL, foo, NULL);
  pthread_join(th, NULL);
  return 0;
}
```
It creates a thread and calls foo (and bar) function from the two thread each.  We compile and record the program as shown:
```
$ gcc -o foobar -pg -pthread foobar.c
$ uftrace record foobar
```
Now `replay` shows the execution of program.  Note that it uses a pager program (usually "less" - use can set the "PAGER" environment variable to change) to control the terminal output easily.  If you don't want this to happen, you can use the `--no-pager` option or set the the "PAGER" env. to "cat".
```
$ uftrace replay
# DURATION    TID     FUNCTION
   2.217 us [22071] | __monstartup();
   2.274 us [22071] | __cxa_atexit();
            [22071] | main() {
            [22071] |   foo() {
   0.234 us [22071] |     bar();
   1.264 us [22071] |   } /* foo */
  68.900 us [22071] |   pthread_create();
            [22071] |   pthread_join() {
            [22073] | foo() {
   0.241 us [22073] |   bar();
   1.819 us [22073] | } /* foo */
 204.783 us [22071] |   } /* pthread_join */
 278.739 us [22071] | } /* main */
```
You can see only specific thread(s) by using the `--tid` option.
```
$ uftrace replay --tid 22073
# DURATION    TID     FUNCTION
            [22073] | foo() {
   0.241 us [22073] |   bar();
   1.819 us [22073] | } /* foo */
```
There's also the `--column-view` option to make it easy to distinguish different threads/processes as shown:
```
$ uftrace replay --column-view
# DURATION    TID     FUNCTION
   2.217 us [22071] | __monstartup();
   2.274 us [22071] | __cxa_atexit();
            [22071] | main() {
            [22071] |   foo() {
   0.234 us [22071] |     bar();
   1.264 us [22071] |   } /* foo */
  68.900 us [22071] |   pthread_create();
            [22071] |   pthread_join() {
            [22073] |                 foo() {
   0.241 us [22073] |                   bar();
   1.819 us [22073] |                 } /* foo */
 204.783 us [22071] |   } /* pthread_join */
 278.739 us [22071] | } /* main */
```
## Analyzing the trace
With the trace data you recorded above, uftrace provides a couple of subcommands to analyze the program execution.  The `report` subcommand shows statistics of each function in the program sorted by (total) time executed.
```
$ uftrace report
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================================
  278.739 us    3.792 us           1  main
  204.783 us  204.783 us           1  pthread_join
   68.900 us   68.900 us           1  pthread_create
    3.083 us    2.608 us           2  foo
    2.274 us    2.274 us           1  __cxa_atexit
    2.217 us    2.217 us           1  __monstartup
    0.475 us    0.475 us           2  bar
```
The total time is a duration of the function including (child) functions called in it.  The self time is a duration excluding the child functions.  If the function is called multiple times, it's a sum of the durations.  You can use `-s` or the `--sort` option to change the sorting behavior.  Available sort keys are: "total" (default), "self" and "call".  Below is the same trace but sorted by the self time.
```
$ uftrace report -s self
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================================
  204.783 us  204.783 us           1  pthread_join
   68.900 us   68.900 us           1  pthread_create
  278.739 us    3.792 us           1  main
    3.083 us    2.608 us           2  foo
    2.274 us    2.274 us           1  __cxa_atexit
    2.217 us    2.217 us           1  __monstartup
    0.475 us    0.475 us           2  bar
```
Sometimes functions are called more than once and you may want to know the average execution time of the function.  In the above example, the "foo" and "bar" functions are called twice - once from the main thread another from the newly created thread.  For this, uftrace provides `--avg-total` and `--avg-self` option to see the average of the total or self time.
```
$ uftrace report --avg-total
   Avg total   Min total   Max total  Function
  ==========  ==========  ==========  ====================================
  278.739 us  278.739 us  278.739 us  main
  204.783 us  204.783 us  204.783 us  pthread_join
   68.900 us   68.900 us   68.900 us  pthread_create
    2.274 us    2.274 us    2.274 us  __cxa_atexit
    2.217 us    2.217 us    2.217 us  __monstartup
    1.541 us    1.264 us    1.819 us  foo
    0.237 us    0.234 us    0.241 us  bar
```
In this average mode, the available sort keys are different: "min", "max" and "avg" (default).

Note that the `live` subcommand also has a `--report` option to show this style of output before the replay.

Sometimes you might want to focus on a single function, like how it was called (backtrace) and what functions it calls (children).  The `graph` command shows you such information.  For example, let's see the function call graph of "foo":
```
$ uftrace graph foo
# Function Call Graph for 'foo' (session: 45427d15c74df0b8)
=============== BACKTRACE ===============
 backtrace #0: hit 1, time   1.270 us
   [0] foo (0x400833)

 backtrace #1: hit 1, time   0.713 us
   [0] main (0x40084b)
   [1] foo (0x400833)

========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    1.983 us : (2) foo
    0.344 us : (2) bar
```
It first shows the backtrace of "foo", one is from "main" and the other is called directly (from the new thread).  Also it shows that "foo" is called twice and then it also calls "bar" twice.  Below is a graph of "main" function:
```
$ uftrace graph
# Function Call Graph for 'foobar' (session: 45427d15c74df0b8)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
  280.574 us : (1) foobar
    1.100 us :  +-(1) __monstartup
             :  |
    1.204 us :  +-(1) __cxa_atexit
             :  |
  277.000 us :  +-(1) main
    0.713 us :  |  +-(1) foo
    0.157 us :  |  | (1) bar
             :  |  |
  134.616 us :  |  +-(1) pthread_create
             :  |  |
  140.232 us :  |  +-(1) pthread_join
             :  |
    1.270 us :  +-(1) foo
    0.187 us :    (1) bar
```
As you can see, if you omit the command line argument (function name), it'll show the full graph mode by default.  It has no backtrace and shows the function call graph.  The function "foo" and "bar" were shown twice, once is from the "main" function and the other is from a new thread shown its parent as the process name "foobar".  In both cases, the "bar" was called from "foo".

Finally, uftrace can show various information about the system or the program during the execution which might helpful for developers.  The output of the `info` subcommand will look like following:
```
$ uftrace info
# system information
# ==================
# program version     : uftrace v0.8.2-70-g587e
# recorded on         : Sun Dec 31 11:02:10 2017
# cmdline             : uftrace record foobar
# cpu info            : Intel(R) Core(TM) i7-2640M CPU @ 2.80GHz
# number of cpus      : 4 / 4 (online / possible)
# memory info         : 7.8 / 15.5 GB (free / total)
# system load         : 0.06 / 0.11 / 0.18 (1 / 5 / 15 min)
# kernel version      : Linux 4.7.3-2-ARCH
# hostname            : danjae
# distro              : "Arch Linux"
#
# process information
# ===================
# number of tasks     : 2
# task list           : 22073(foobar), 22071(foobar)
# exe image           : /home/namhyung/tmp/foobar
# build id            : 5c748f191162cee663c15a424e3ea979a9a04b6d
# exit status         : exited with code: 0
# elapsed time        : 0.005073929 sec
# cpu time            : 0.003 / 0.000 sec (sys / user)
# context switch      : 2 / 1 (voluntary / involuntary)
# max rss             : 3172 KB
# page fault          : 0 / 198 (major / minor)
# disk iops           : 0 / 16 (read / write)
```