# Debug levels
uftrace has various debug messages to help investigating problems.  The `-v` option is to enable these messages and you can use it multiple times to increase the debug level.  The following is a guideline to choose an appropriate debug level when you add new messages.

1. level 1 (`-v`): Important events for global state changes.  Normally not repeated or at a very low frequency.  We can check this regularly to see what's happening at the very high level.  (Please add something to the level 1 with care!)  For example, it can include what uftrace version is, what's the name of binary it's tracing, which libmcount it uses, dynamic tracing stats and so on.
2. level 2 (`-vv`): Important events for per-entity state changes like task, module, filter and so on.  Or detailed info for the global events.  It can have multiple events of same kind but for different entities.  Recommended for per-entity stats like how many symbols in a module.  It should not generate overwhelmingly large number of messages.
3. level 3 (`-vvv`): Detailed info (possibly repeated for many entries).  It should give enough information for debugging in the most cases.  Recommended for enumerating entries in a table.  For example, per-function dynamic patch info, symbol and debug info entries, libmcount rstack changes.
4. level 4 (`-vvvv`): (Optional) very detailed info.  Anything not included in the previous levels should fall into this category.

# Debug domains
uftrace has a number of debug domains to focus on a specific subsystem easily.  The `-v` or `--debug` option sets the same debug level for each domain.  Users can use `--debug-domain` option to set the different debug level for each domain.  The debug domains are defined in the [utils/utils.h](https://github.com/namhyung/uftrace/blob/v0.13/utils/utils.h#L70-L87) header file.  Each source file may define `PR_DOMAIN` and `PR_FMT`(before including the `utils/utils.h`) to setup its debug domain and prefix for the debug messages.  If not, it'd use `uftrace` domain by default.

This is an example debug message for level 1.
```
$ uftrace -v abc
uftrace: running uftrace v0.13.1 ( x86_64 dwarf python3 luajit tui perf sched dynamic )
uftrace: checking binary abc
uftrace: using /usr/local/lib/libmcount.so library for tracing
uftrace: creating 1 thread(s) for recording
mcount: initializing mcount library
plthook: setup PLT hooking "/home/user/tmp/abc"
mcount: mcount setup done
mcount: new session started: e17921e01196f791: abc
mcount: mcount trace finished
mcount: exit from libmcount
uftrace: child terminated with exit code: 0
uftrace: reading /tmp/uftrace-live-aX2fVu/task.txt file
uftrace: flushing /uftrace-e17921e01196f791-16642-000
uftrace: live-record finished.. 
uftrace: start live-replaying...
uftrace: reading /tmp/uftrace-live-aX2fVu/task.txt file
fstack: fixup for some special functions
uftrace: add field "duration"
uftrace: add field "tid"
...
```

You can only read one message for `plthook` domain.  It'll be different when you pass `plthook:2`.
```
$ uftrace --debug-domain uftrace:1,plthook:2 abc
uftrace: running uftrace v0.13.1 ( x86_64 dwarf python3 luajit tui perf sched dynamic )
uftrace: checking binary abc
uftrace: using /usr/local/lib/libmcount.so library for tracing
uftrace: creating 1 thread(s) for recording
plthook: setup PLT hooking "/home/user/tmp/abc"
plthook: setup plthook data for /home/user/tmp/abc (offset: 55a92e0ce000)
plthook: opening executable image: /home/user/tmp/abc
plthook: "abc" is loaded at 0x55a92e0ce000
plthook: update module id to 0x55a92f2b35c0
plthook: found GOT at 0x55a92e0d1f98 (base_addr + 0x3f98)
plthook: module id = 0x55a92f2b35c0, PLT resolver = 0
plthook: restore GOT[3] from "getpid"(0x7f17cbb2e0c0) to PLT(base + 0x1030)
plthook: restore GOT[4] from "__monstartup"(0x7f17cbc505e0) to PLT(base + 0x1040)
plthook: restore GOT[5] from "__cxa_atexit"(0x7f17cba90de0) to PLT(base + 0x1050)
plthook: restore GOT[6] from "atoi"(0x7f17cba8e5b0) to PLT(base + 0x1060)
plthook: destroy plthook special function index
uftrace: child terminated with exit code: 0
uftrace: reading /tmp/uftrace-live-M7EDck/task.txt file
uftrace: flushing /uftrace-aae280c61a22624e-16706-000
uftrace: live-record finished.. 
uftrace: start live-replaying...
uftrace: reading /tmp/uftrace-live-M7EDck/task.txt file
uftrace: add field "duration"
uftrace: add field "tid"
...
```