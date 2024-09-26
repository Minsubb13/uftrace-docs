**1. Download gcc source and decompress**
```
$ wget https://ftp.gnu.org/gnu/gcc/gcc-9.1.0/gcc-9.1.0.tar.gz
$ tar xfz gcc-9.1.0.tar.gz
```

**2. Check and download the support libraries**
```
$ cd gcc-9.1.0
$ ./contrib/download_prerequisites
$ cd ..
```

**3. Build configuration**
```
$ mkdir -p gcc-9.1.0-build sysroot.pg
$ cd gcc-9.1.0-build
$ ../gcc-9.1.0/configure --prefix=`realpath ../sysroot.pg` \
                         --enable-languages=c,c++ \
                         --disable-multilib \
                         --disable-bootstrap \
                         CFLAGS="-pg -g -fno-omit-frame-pointer" \
                         CXXFLAGS="-pg -g -fno-omit-frame-pointer"
```

**4. Compile and build**
```
$ make -j8 CFLAGS="-pg -g -fno-omit-frame-pointer" CXXFLAGS="-pg -g -fno-omit-frame-pointer"
$ make install
```

**5. Test and check the generated gcc binary**
```
$ cd ../sysroot.pg/bin

$ ./gcc
gcc: fatal error: no input files
compilation terminated.

$ nm gcc | grep mcount
                 U mcount@@GLIBC_2.2.5
```

**6. Run gcc compiler with uftrace**
```
$ uftrace -F main -D 3 -- ./gcc
gcc: fatal error: no input files
compilation terminated.
# DURATION     TID     FUNCTION
            [139913] | main() {
            [139913] |   driver::driver() {
   0.166 us [139913] |     option_proposer::option_proposer();
   0.161 us [139913] |     env_manager::init();
   1.637 us [139913] |   } /* driver::driver */
            [139913] |   driver::main() {
   4.529 us [139913] |     driver::set_progname();
   5.651 us [139913] |     driver::expand_at_files();
 104.448 us [139913] |     driver::decode_argv();
 214.672 us [139913] |     driver::global_initializations();
   9.213 us [139913] |     driver::build_multilib_strings();
 827.357 us [139913] |     driver::set_up_specs();
   6.835 us [139913] |     driver::putenv_COLLECT_GCC();
  16.127 us [139913] |     driver::maybe_putenv_COLLECT_LTO_WRAPPER();
   0.597 us [139913] |     driver::maybe_putenv_OFFLOAD_TARGETS();
   0.203 us [139913] |     driver::handle_unrecognized_options();
   0.382 us [139913] |     driver::maybe_print_and_exit();
            [139913] |     driver::prepare_infiles() {
            [139913] |       /* linux:task-exit */

uftrace stopped tracing with remaining functions
================================================
task: 139913
[2] driver::prepare_infiles
[1] driver::main
[0] main
```

**7. Run gcc compiler with uftrace --auto-args**
```
$ uftrace -F main -D 3 --auto-args -- ./gcc
gcc: fatal error: no input files
compilation terminated.
# DURATION     TID     FUNCTION
            [139945] | main(1, 0x7fff78a5e428) {
            [139945] |   driver::driver(0x7fff78a5e300, 0, 0) {
   0.267 us [139945] |     option_proposer::option_proposer(0x7fff78a5e318);
   0.381 us [139945] |     env_manager::init(&env, 0, 0);
   2.609 us [139945] |   } /* driver::driver */
            [139945] |   driver::main(0x7fff78a5e300, 1, 0x7fff78a5e428) {
 244.683 us [139945] |     driver::set_progname(0x7fff78a5e300, "./gcc");
   6.402 us [139945] |     driver::expand_at_files(0x7fff78a5e300, 0x7fff78a5e2c4, 0x7fff78a5e2b8);
 160.592 us [139945] |     driver::decode_argv(0x7fff78a5e300, 1, 0x7fff78a5e428);
 260.046 us [139945] |     driver::global_initializations(0x7fff78a5e300);
   8.955 us [139945] |     driver::build_multilib_strings(0x7fff78a5e300);
            [139945] |     driver::set_up_specs(0x7fff78a5e300);
  10.164 us [139945] |     driver::putenv_COLLECT_GCC(0x7fff78a5e300, "./gcc");
  19.760 us [139945] |     driver::maybe_putenv_COLLECT_LTO_WRAPPER(0x7fff78a5e300);
   0.794 us [139945] |     driver::maybe_putenv_OFFLOAD_TARGETS(0x7fff78a5e300);
   0.292 us [139945] |     driver::handle_unrecognized_options(0x7fff78a5e300);
   0.536 us [139945] |     driver::maybe_print_and_exit(0x7fff78a5e300) = 1;
            [139945] |     driver::prepare_infiles(0x7fff78a5e300) {
            [139945] |       /* linux:task-exit */

uftrace stopped tracing with remaining functions
================================================
task: 139945
[2] driver::prepare_infiles
[1] driver::main
[0] main
```

**8. Generate dump output**
```
$ uftrace record -F main -D 3 -- ./gcc
$ uftrace replay
# DURATION     TID     FUNCTION
            [139968] | main() {
...
```

**9. Visualized trace output of gcc**
- chrome: https://uftrace.github.io/dump/gcc.html
- flame-graph: https://uftrace.github.io/dump/gcc.svg

[![gcc.svg](https://uftrace.github.io/dump/gcc.svg)](https://uftrace.github.io/dump/gcc.svg)