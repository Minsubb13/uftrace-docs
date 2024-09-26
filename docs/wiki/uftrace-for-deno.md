The [deno](https://github.com/denoland/deno) is a secure JavaScript and TypeScript runtime written in Rust.

#### 1. Download deno
```
$ git clone --recurse-submodules https://github.com/denoland/deno.git
$ cd deno
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

# The rust-toolchain.toml file overrides nightly toolchain so should be discarded.
$ mv rust-toolchain.toml rust-toolchain.toml.orig
```

#### 4. Configure and build deno with uftrace
```
$ export RUSTFLAGS="-Z instrument-mcount -C passes=ee-instrument<post-inline>"
$ cargo build
```

#### 5. Check the generated binary
```
$ nm ./target/debug/deno | grep mcount
                 U mcount@@GLIBC_2.2.5
```

#### 6. Run deno interpreter with uftrace
```
$ uftrace record ./target/debug/deno run cli/tests/testdata/run/002_hello.ts
$ uftrace replay -t 80ms -F ^deno -F ^tokio
# DURATION     TID     FUNCTION
            [357474] | deno::main() {
            [357474] |   deno_runtime::tokio_util::run_local() {
            [357474] |     tokio::task::local::LocalSet::block_on() {
            [357474] |       tokio::runtime::runtime::Runtime::block_on() {
            [357474] |         tokio::runtime::scheduler::current_thread::CurrentThread::block_on() {
            [357474] |           tokio::runtime::scheduler::current_thread::CoreGuard::block_on() {
            [357474] |             tokio::runtime::scheduler::current_thread::CoreGuard::enter() {
            [357474] |               tokio::macros::scoped_tls::ScopedKey<T>::set() {
            [357474] |                 tokio::runtime::scheduler::current_thread::CoreGuard::enter::_{{closure}}() {
            [357474] |                   tokio::runtime::scheduler::current_thread::CoreGuard::block_on::_{{closure}}() {
            [357474] |                     tokio::runtime::scheduler::current_thread::Context::enter() {
            [357474] |                       tokio::runtime::scheduler::current_thread::CoreGuard::block_on::_{{closure}}::_{{closure}}() {
            [357474] |                         tokio::runtime::scheduler::current_thread::CoreGuard::block_on::_{{closure}}::_{{closure}}::_{{closure}}() {
            [357474] |                           _<core::pin::Pin<P>>::poll() {
            [357474] |                             tokio::task::local::LocalSet::run_until::_{{closure}}() {
            [357474] |                               _<tokio::task::local::RunUntil<T>>::poll() {
            [357474] |                                 tokio::task::local::LocalSet::with() {
            [357474] |                                   std::thread::local::LocalKey<T>::with() {
            [357474] |                                     std::thread::local::LocalKey<T>::try_with() {
            [357474] |                                       tokio::task::local::LocalSet::with::_{{closure}}() {
            [357474] |                                         _<tokio::task::local::RunUntil<T>>::poll::_{{closure}}() {
  94.719 ms [357474] |                                           deno::main::_{{closure}}();
  94.752 ms [357474] |                                         } /* _<tokio::task::local::RunUntil<T>>::poll::_{{closure}} */
  94.757 ms [357474] |                                       } /* tokio::task::local::LocalSet::with::_{{closure}} */
  94.760 ms [357474] |                                     } /* std::thread::local::LocalKey<T>::try_with */
  94.760 ms [357474] |                                   } /* std::thread::local::LocalKey<T>::with */
  94.760 ms [357474] |                                 } /* tokio::task::local::LocalSet::with */
  94.761 ms [357474] |                               } /* _<tokio::task::local::RunUntil<T>>::poll */
  94.767 ms [357474] |                             } /* tokio::task::local::LocalSet::run_until::_{{closure}} */
  94.803 ms [357474] |                           } /* _<core::pin::Pin<P>>::poll */
  94.806 ms [357474] |                         } /* tokio::runtime::scheduler::current_thread::CoreGuard::block_on::_{{closure}}::_{{closure}}::_{{closure}} */
  94.818 ms [357474] |                       } /* tokio::runtime::scheduler::current_thread::CoreGuard::block_on::_{{closure}}::_{{closure}} */
  94.821 ms [357474] |                     } /* tokio::runtime::scheduler::current_thread::Context::enter */
  94.828 ms [357474] |                   } /* tokio::runtime::scheduler::current_thread::CoreGuard::block_on::_{{closure}} */
  94.831 ms [357474] |                 } /* tokio::runtime::scheduler::current_thread::CoreGuard::enter::_{{closure}} */
  94.844 ms [357474] |               } /* tokio::macros::scoped_tls::ScopedKey<T>::set */
  94.852 ms [357474] |             } /* tokio::runtime::scheduler::current_thread::CoreGuard::enter */
  94.852 ms [357474] |           } /* tokio::runtime::scheduler::current_thread::CoreGuard::block_on */
  94.871 ms [357474] |         } /* tokio::runtime::scheduler::current_thread::CurrentThread::block_on */
  94.918 ms [357474] |       } /* tokio::runtime::runtime::Runtime::block_on */
  94.968 ms [357474] |     } /* tokio::task::local::LocalSet::block_on */
  96.317 ms [357474] |   } /* deno_runtime::tokio_util::run_local */
            [357474] |   /* linux:task-exit */
```

#### 7. Generate chrome dump output
```
# record
$ uftrace record -t 10us ./target/debug/deno run cli/tests/testdata/run/002_hello.ts
Hello World

# dump in json format
$ uftrace dump --chrome > deno.json

# convert json to html
$ trace2html deno.json
deno.html
```

#### 8. Visualized trace output of deno
- chrome: https://uftrace.github.io/dump/deno.html
- flame-graph: https://uftrace.github.io/dump/deno.svg

[![deno.svg](https://uftrace.github.io/dump/deno.svg)](https://uftrace.github.io/dump/deno.svg)