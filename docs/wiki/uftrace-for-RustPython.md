This document is written by [MinJeong Kim](https://github.com/rls1004).

#### 1. Download RustPython
```
$ git clone --depth=1 https://github.com/RustPython/RustPython
$ cd RustPython
```
#### 2. Install Rust
Make sure `export PATH="$HOME/.cargo/bin:$PATH"` is added to your environment variables.
```
$ curl https://sh.rustup.rs/ -sSf | sh
$ source $HOME/.cargo/env
```
#### 3. Install & set nightly as default compiler
Nightly compiler is required for using -Z flag options.
```
$ rustup install nightly
$ rustup default nightly
```
#### 4. Configure and build RustPython with uftrace
```
$ export RUSTFLAGS="-Z instrument-mcount -C passes='ee-instrument<post-inline>'"
$ cargo build
```
#### 5. Check the generated binary
```
$ nm ./target/debug/rustpython | grep mcount
                 U mcount@@GLIBC_2.2.5
```
#### 6. Get demangled version of Rust symbol names
More readable calling convention is needed to trace a specific function. `rustfilt` could pipe to stdout using `grep` or `less`.
```
$ cargo install rustfilt
$ nm ./target/debug/rustpython | rustfilt
```
#### 7. Run RustPython interpreter with uftrace
```
$ uftrace -F rustpython_vm::obj::objrange::PyRange::getitem -D 2 ./target/debug/rustpython -c 'range(10)[0:5]'
# DURATION     TID     FUNCTION
            [ 10285] | rustpython_vm::obj::objrange::PyRange::getitem() {
   5.697 us [ 10285] |   rustpython_vm::obj::objslice::_<impl rustpython_vm..pyobject..PyRef<rustpyth
   0.127 us [ 10285] |   _<core..result..Result<T,E> as core..ops..try..Try>::into_result();
  20.885 us [ 10285] |   rustpython_vm::obj::objrange::PyRange::get();
   0.139 us [ 10285] |   rustpython_vm::obj::objint::PyInt::new();
   1.971 us [ 10285] |   rustpython_vm::pyobject::PyValue::into_ref();
   0.332 us [ 10285] |   core::ptr::real_drop_in_place();
   2.709 us [ 10285] |   rustpython_vm::obj::objslice::_<impl rustpython_vm..pyobject..PyRef<rustpyth
   0.094 us [ 10285] |   _<core..result..Result<T,E> as core..ops..try..Try>::into_result();
  18.016 us [ 10285] |   rustpython_vm::obj::objrange::PyRange::get();
   0.114 us [ 10285] |   rustpython_vm::obj::objint::PyInt::new();
   3.267 us [ 10285] |   rustpython_vm::pyobject::PyValue::into_ref();
   0.659 us [ 10285] |   core::ptr::real_drop_in_place();
   0.672 us [ 10285] |   rustpython_vm::obj::objslice::_<impl rustpython_vm..pyobject..PyRef<rustpyth
   0.094 us [ 10285] |   _<core..result..Result<T,E> as core..ops..try..Try>::into_result();
   0.494 us [ 10285] |   _<rustpython_vm..pyobject..PyRef<T> as core..clone..Clone>::clone();
   0.038 us [ 10285] |   core::ptr::real_drop_in_place();
   1.695 us [ 10285] |   rustpython_vm::pyobject::PyValue::into_ref();
   0.041 us [ 10285] |   rustpython_vm::pyobject::PyRef<T>::into_object();
   0.533 us [ 10285] |   core::ptr::real_drop_in_place();
  62.415 us [ 10285] | } /* rustpython_vm::obj::objrange::PyRange::getitem */
```
#### 8. Generate dump output
```
$ uftrace record -F rustpython_vm::obj::objrange::PyRange::getitem -D 2 ./target/debug/rustpython -c 'range(10)[0:5]'
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 10304] | rustpython_vm::obj::objrange::PyRange::getitem() {
...
```
#### 9. Visualized trace output of RustPython
Graph : 
```
$ uftrace graph
# Function Call Graph for 'rustpython' (session: 20cd9b6b768a352d)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
   58.425 us : (1) rustpython
   58.425 us : (1) rustpython_vm::obj::objrange::PyRange::getitem
    4.933 us :  +-(1) rustpython_vm::obj::objslice::_<impl rustpython_vm..pyobject..PyRef<rustpython_vm..obj
             :  | 
    0.298 us :  +-(3) _<core..result..Result<T,E> as core..ops..try..Try>::into_result
             :  | 
   37.394 us :  +-(2) rustpython_vm::obj::objrange::PyRange::get
             :  | 
    0.258 us :  +-(2) rustpython_vm::obj::objint::PyInt::new
             :  | 
    5.669 us :  +-(3) rustpython_vm::pyobject::PyValue::into_ref
             :  | 
    1.637 us :  +-(4) core::ptr::real_drop_in_place
             :  | 
    2.708 us :  +-(1) rustpython_vm::obj::objslice::_<impl rustpython_vm..pyobject..PyRef<rustpython_vm..obj
             :  | 
    0.624 us :  +-(1) rustpython_vm::obj::objslice::_<impl rustpython_vm..pyobject..PyRef<rustpython_vm..obj
             :  | 
    0.501 us :  +-(1) _<rustpython_vm..pyobject..PyRef<T> as core..clone..Clone>::clone
             :  | 
    0.041 us :  +-(1) rustpython_vm::pyobject::PyRef<T>::into_object
```
Chrome : 
```
$ uftrace record -t 5ms ./target/debug/rustpython -c 'range(10)[0:5]'
$ uftrace dump --chrome > uftrace-dump-rustpython.json
$ trace2html uftrace-dump-rustpython.json
```
Flame-Graph : 
```
$ uftrace record -t 5ms ./target/debug/rustpython -c 'range(10)[0:5]'
$ uftrace dump --flame-graph | ./flamegraph.pl > uftrace-dump-rustpython.svg
```