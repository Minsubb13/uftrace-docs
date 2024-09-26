# Overview

uftrace is a tool to trace and analyze execution of a program written in C/C++.  It was inspired by Linux kernel's ftrace (especially function graph tracer), and supports userspace programs on Linux.  In order to use uftrace, the program should be compiled with `-pg` or `-finstrument-functions` option.  (On ARM platforms, you may need to add `-fno-omit-frame-pointer` option also).  The example output looks like below:

    $ cat hello.c
    #include <stdio.h>
    int main(void)
    {
       return printf("%s world\n", "Hello");
    }
    
    $ gcc -pg -o hello hello.c
    
    $ uftrace ./hello
    Hello world
    # DURATION    TID     FUNCTION
                [ 7516] | main() {
     414.023 us [ 7516] |   printf();
     697.310 us [ 7516] | } /* main */

In the above example, uftrace ran the hello program (it printed "Hello world") and showed the execution replay with duration and tid.  As you can see, the uftrace replay output resembles the original source code.  In addition, uftrace provides useful commands and features to analyze behavior of your program.

## Quick install
The uftrace has some optional dependencies but it can be built without any of them.  If you have git installed, below command will download and build it from the source:

    $ git clone https://github.com/namhyung/uftrace.git
    $ cd uftrace

    $ sudo misc/install-deps.sh    # optional for advanced features
    $ ./configure                  # --prefix can be used to change install dir
    $ make
    $ sudo make install

For more information, please refer [INSTALL.md](https://github.com/namhyung/uftrace/blob/master/INSTALL.md) page.

## Tutorials
* [Tutorial in this wiki](Tutorial.md)
* [CppCon 2016 lighting talk](https://github.com/CppCon/CppCon2016/blob/master/Lightning%20Talks%20and%20Lunch%20Sessions/uftrace%20-%20A%20function%20graph%20tracer%20C%20C%2B%2B%20userspace%20programs/uftrace%20-%20A%20function%20graph%20tracer%20C%20C%2B%2B%20userspace%20programs%20-%20Namhyung%20Kim%20and%20Honggyu%20Kim%20-%20CppCon%202016.pdf) - presented by Honggyu Kim
* [Tracing Summit 2016 presentation](http://tracingsummit.org/w/images/3/3d/Uftrace_-_TracingSummit_2016.pdf) - presented by Namhyung Kim
* [CppCon 2017 presentation](https://github.com/CppCon/CppCon2017/blob/master/Tutorials/Understanding%20the%20Runtime%20Behaviors%20of%20C%2B%2B%20Programs%20Using%20uftrace%20Tool/Understanding%20the%20Runtime%20Behaviors%20of%20C%2B%2B%20Programs%20Using%20uftrace%20Tool%20-%20Honggyu%20Kim%20-%20CppCon%202017.pdf) - presented by Honggyu Kim
* [Open Source Summit Europe 2017 tutorial](https://schd.ws/hosted_files/osseu17/52/Good_bye_printf_hands_on_tutorial_uftrace_function_graph_tracer_C_C%2B%2B.pdf) - presented by Taeung Song
* [Open Source Summit North America 2017 presentation](http://schd.ws/hosted_files/ossna2017/a0/uftrace-ossna2017.pdf) - by Namhyung Kim
* [Tracing Summit 2019 presentation](https://tracingsummit.org/w/images/5/5b/Tracingsummit2019-uftrace-kim.pdf) - presented by Namhyung Kim
* [Black Hat Asia 2023 Arsenal](https://uftrace.github.io/talks/uftrace_BHAS23_Arsenal.pdf) - presented by Honggyu Kim

## Man pages
* [uftrace](https://github.com/namhyung/uftrace/blob/master/doc/uftrace.md) - uftrace overview
* [uftrace record](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-record.md) - run and save result
* [uftrace replay](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-replay.md) - show execution flow
* [uftrace report](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-report.md) - analyze execution result
* [uftrace live](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-live.md) - run and show execution flow
* [uftrace info](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-info.md) - show process metadata
* [uftrace dump](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-dump.md) - show and convert raw result
* [uftrace recv](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-recv.md) - save result through network
* [uftrace graph](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-graph.md) - show function call graph
* [uftrace script](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-script.md) - run a script for recorded function trace
* [uftrace tui](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-tui.md) - interactive Text-base User Interface