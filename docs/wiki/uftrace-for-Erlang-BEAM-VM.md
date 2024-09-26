# uftrace for Erlang BEAM VM

This document is written by [Yunseong Kim](https://github.com/yskelg) known as [Paran Lee](https://github.com/paranlee)

Erlang is a programming language and runtime system for building massively scalable soft real-time systems with requirements on high availability.

OTP is a set of Erlang libraries, which consists of the Erlang runtime system, a number of ready-to-use components mainly written in Erlang, and a set of design principles for Erlang programs.

BEAM is Virtual Machine that run Erlang and Elixr Programming language.

## 1. clone Erlang/OTP source code

```
git clone https://github.com/erlang/otp.git
```

## 2. build with pg option Erlang/OTP

```
cd otp

export CFLAGS="-g3 -O0 -fno-omit-frame-pointer -pg"
export CXXFLAGS="-g3 -O0 -fno-omit-frame-pointer -pg"
export ERL_TOP=$(pwd)

./configure

time make -j96 | tee make-build.log

make -j96 -C erts/emulator clean
time make -j96 -C erts/emulator debug asan gprof 2>&1 | tee erts_emulator-build.log

make -j96 -C erts/lib_src clean
time make -j96 -C erts/lib_src debug asan gprof 2>&1 | tee erts_lib_src-build.log

make -j96 -C lib/crypto/c_src clean
time make -j96 -C lib/crypto/c_src debug asan gprof 2>&1 | tee lib_crypto_c_src-build.log
```

## 3. Do uftrace!

`escript`

Test code `factorial.erl`
```
 otp$ cat factorial.erl 
#!/usr/bin/env escript
%% -*- erlang -*-
%%! -sname factorial -mnesia debug verbose
main([String]) ->
    try
        N = list_to_integer(String),
        F = fac(N),
        io:format("factorial ~w = ~w\n", [N,F])
    catch
        _:_ ->
            usage()
    end;
main(_) ->
    usage().

usage() ->
    io:format("usage: factorial integer\n"),
    halt(1).

fac(0) -> 1;
fac(N) -> N * fac(N-1).
```
Trace the factorial erlang program.
```
$ uftrace record --no-sched --no-libcall -N ^std -t 20us bin/escript factorial.erl 3

$ uftrace replay
# DURATION     TID     FUNCTION
            [ 22055] | main() {
  28.099 ms [ 22055] |   start_epmd_daemon();
            [ 22055] |   /* linux:task-name (comm="beam.smp") */
            [ 22055] |   _GLOBAL__sub_I_BeamGlobalAssembler::emitPtrs() {
 253.384 us [ 22055] |     __static_initialization_and_destruction_0();
 256.612 us [ 22055] |   } /* _GLOBAL__sub_I_BeamGlobalAssembler::emitPtrs */
            [ 22055] |   main() {
            [ 22055] |     erl_start() {
            [ 22055] |       early_init() {
            [ 22055] |         erts_pre_early_init_cpu_topology() {
            [ 22055] |           erts_cpu_info_create() {
            [ 22055] |             erts_cpu_info_update() {
            [ 22055] |               read_cpu_quota() {
            [ 22055] |                 get_cgroup_path() {
  47.290 us [ 22055] |                   get_cgroup_child_path();
 102.354 us [ 22055] |                 } /* get_cgroup_path */
 126.461 us [ 22055] |               } /* read_cpu_quota */
 198.301 us [ 22055] |             } /* erts_cpu_info_update */
 199.631 us [ 22055] |           } /* erts_cpu_info_create */
 202.694 us [ 22055] |         } /* erts_pre_early_init_cpu_topology */
            [ 22055] |         erts_sys_pre_init() {
            [ 22055] |           erts_thr_init() {
            [ 22055] |             ethr_init() {
            [ 22055] |               ethr_init_common__() {
            [ 22055] |                 erts_cpu_info_create() {
            [ 22055] |                   erts_cpu_info_update() {
            [ 22055] |                     read_cpu_quota() {
  38.797 us [ 22055] |                       get_cgroup_path();
  51.421 us [ 22055] |                     } /* read_cpu_quota */
  79.093 us [ 22055] |                   } /* erts_cpu_info_update */
  79.429 us [ 22055] |                 } /* erts_cpu_info_create */
  86.339 us [ 22055] |               } /* ethr_init_common__ */
  96.058 us [ 22055] |             } /* ethr_init */
  97.334 us [ 22055] |           } /* erts_thr_init */
 126.595 us [ 22055] |         } /* erts_sys_pre_init */
            [ 22055] |         erts_early_init_scheduling() {
  62.984 us [ 22055] |           aux_work_timeout_early_init();
  70.929 us [ 22055] |         } /* erts_early_init_scheduling */
            [ 22055] |         erts_alloc_init() {
 419.831 us [ 22055] |           erts_mseg_init();
            [ 22055] |           start_au_allocator() {
            [ 22055] |             erts_gfalc_start() {
```

# `Perfetto` for dumped json file

We can use `Perfetto`.

```
alias uftrace_json="$HOME/uftrace/uftrace dump --libmcount-path=$HOME/uftrace --chrome > dump.json"
```
- [dump.zip](https://github.com/namhyung/uftrace/files/14156453/dump.zip)
- https://ui.perfetto.dev/

![image](https://github.com/namhyung/uftrace/assets/43713967/fae7e3f4-1638-4344-859d-4e0d8d1b38d4)
