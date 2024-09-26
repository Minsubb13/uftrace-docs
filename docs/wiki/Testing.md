NOTE: This page is a draft version, so needs more update.

# Testing

There are various test cases in `tests` directory, and `tests/runtest.py` script runs these test cases.

You can simply use `./runtest.py [case] [option]`. In the [case], enter the name or number of the test case.
If [case] is not entered, testing is supported for the entire test case. 

You can test multiple test cases at once and apply different options for one test case (e.g., simultaneously applying optimization steps 1, 2 and 3). Through the `-v` option, you can check how the test command of build compound and `-f` option is to give a compiler sealing plaque. The `-O` option also allows the optimization phase to be adjusted. These functions make it easier and more convenient to test, and in addition to the test case created, the user can create a test case and perform the test.

When a test fails, `runtest.py` will create `failed-test.txt` and write the failed test, so you can see list of failed tests in the file after the `runtest.py` is executed.

## How to add test case

If you do not have the test code you want, you can write the test code and conduct the test.

- Example To add the desired test case:

**1. create s-[TEST CASE NAME].c or s-[TEST CASE NAME].cpp (In your preferred language)**

```c
$ cat s-USER_TEST.c 
#include<stdio.h>

void TEST(void);

int main()
{
	TEST();

	return 0;
}

void TEST(void){
	printf("USER_TEST_CODE\n");
}
```

**2. Apply the default gcc and uftrace options in runtest.py to get the following results:**

- You have to run the following commands in `tests` directory.

```
$ gcc -o t-USER_TEST -fno-inline -fno-builtin -fno-ipa-cp -fno-omit-frame-pointer -D_FORTIFY_SOURCE=0  -finstrument-functions -Os  s-USER_TEST.c
<command-line>:0:0: warning: "_FORTIFY_SOURCE" redefined
<built-in>: note: this is the location of the previous definition

$ uftrace --no-pager --no-event -L.. t-USER_TEST
USER_TEST_CODE
# DURATION     TID     FUNCTION
            [  6053] | main() {
            [  6053] |   TEST() {
 149.720 us [  6053] |     printf();
 151.607 us [  6053] |   } /* TEST */
 152.396 us [  6053] | } /
```

**3. create t[TEST CASE NUMBER]_[TEST CASE NAME].py and paste second result**

```
#!/usr/bin/env python

from runtest import TestBase

class TestCase(TestBase):
    def __init__(self):
        TestBase.__init__(self, 'USER_TEST', """
# DURATION     TID     FUNCTION
   0.858 us [  6053] | __monstartup();
   0.332 us [  6053] | __cxa_atexit();
            [  6053] | main() {
            [  6053] |   TEST() {
 131.151 us [  6053] |     printf();
 132.454 us [  6053] |   } /* TEST */
 132.658 us [  6053] | } /* main */
 """)
```

**4. you can get the following results:**

````
$ ./runtest.py 212 -v -O1
Test case                 pg fi
------------------------: O1 O1
build command: gcc -o t-USER_TEST -fno-inline -fno-builtin -fno-ipa-cp -fno-omit-frame-pointer -D_FORTIFY_SOURCE=0  -pg -O1  s-USER_TEST.c   
test command: ../uftrace --no-pager --no-event -L.. t-USER_TEST
=========== original ===========
USER_TEST_CODE
# DURATION     TID     FUNCTION
   0.658 us [  3344] | __monstartup();
   0.327 us [  3344] | __cxa_atexit();
            [  3344] | main() {
            [  3344] |   TEST() {
   5.075 us [  3344] |     printf();
   5.400 us [  3344] |   } /* TEST */
   5.637 us [  3344] | } /* main */

===========  result  ===========
main() {
   TEST() {
     printf();
   } /* TEST */
 } /* main */
=========== expected ===========
main() {
   TEST() {
     printf();
   } /* TEST */
 } /* main */
build command: gcc -o t-USER_TEST -fno-inline -fno-builtin -fno-ipa-cp -fno-omit-frame-pointer -D_FORTIFY_SOURCE=0  -finstrument-functions -O1  s-USER_TEST.c   
test command: ../uftrace --no-pager --no-event -L.. t-USER_TEST
=========== original ===========
USER_TEST_CODE
# DURATION     TID     FUNCTION
            [  3353] | main() {
            [  3353] |   TEST() {
   7.573 us [  3353] |     printf();
   9.183 us [  3353] |   } /* TEST */
   9.872 us [  3353] | } /* main */

===========  result  ===========
main() {
   TEST() {
     printf();
   } /* TEST */
 } /* main */
=========== expected ===========
main() {
   TEST() {
     printf();
   } /* TEST */
 } /* main */
212 USER_TEST           : OK OK

runtime test stats
====================
total     2  Tests executed (success: 100.00%)
  OK:     2  Test succeeded
  OK:     0  Test succeeded (with some fixup)
  NG:     0  Different test result
  NZ:     0  Non-zero return value
  SG:     0  Abnormal exit by signal
  TM:     0  Test ran too long
  BI:     0  Build failed
  LA:     0  Unsupported Language
  SK:     0  Skipped
````