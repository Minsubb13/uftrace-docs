## 1. option system change
* replace argp to new option system  (Done)
* support "on"/"off" pair by providing a single option: [#743](https://github.com/namhyung/uftrace/issues/743)
* this is a first step for Android support: [#433](https://github.com/namhyung/uftrace/issues/433)

## 2. user interface improvement
* field customization (runtime) : report and graph
* report field sorting
* move focus to center

## 3. integration of externally recorded uftrace data
* We can record uftrace data from other programs such as JITed trace from V8.
* We need a way to integrate this kind of record into uftrace data format.

## 4. call graph diff
* We can record twice and compare the summary using `report --diff` as of now.
* We can also compare the call graph diff to identify the specific difference in details.
* We may provide another GUI output for showing the diff efficiently.

## 5. remaining (python) script enhancement
* drawing various charts using `matplotlib`.<br>
\- drawing malloc and free to see memory allocation graph<br>
\- drawing RSS charts by using read trigger<br>
\- many others...
* event handing in (python) script
* collecting useful scripts<br>
\- such as tools in https://github.com/brendangregg/perf-tools

## 6. return-time trigger
* current trigger action is called only at function entry
* reuse `-R` option to allow it called at function exit

## 7. attaching to already running process
* dl-library linked or not?
* work-in-progress : [#360](https://github.com/namhyung/uftrace/pull/360)

## 8. other language support
* Rust : [#594](https://github.com/namhyung/uftrace/issues/594), [rust-lang/rust#57220](https://github.com/rust-lang/rust/pull/57220), [xiph/rav1e/doc/PROFILING.md](https://github.com/xiph/rav1e/blob/master/doc/PROFILING.md#uftrace)
* Go : [#557](https://github.com/namhyung/uftrace/issues/557)
* Python (interpreter + script) : [#436](https://github.com/namhyung/uftrace/issues/436), [#680](https://github.com/namhyung/uftrace/issues/680)
* JavaScript (need to modify engine itself)
* D : [ldc-developers/ldc#1839](https://github.com/ldc-developers/ldc/issues/1839), [ldc-developers/ldc#1845](https://github.com/ldc-developers/ldc/issues/1845)

## 9. filter improvement
* value filter
* size filter : [#780](https://github.com/namhyung/uftrace/issues/780)
* caller filter : [#166](https://github.com/namhyung/uftrace/issues/166)
* (source) location filter

## etc.
* automatic color for each context (kernel, library, event)
* `--help` with pager
* more auto completion


# Completed Items

## 1. nested library tracing
* We can see meaningful enough library functions without user functions.
* We don't miss some important library calls.
* We can remove the restriction of recompiling with `-pg` or `-finstrument-functions`.
* done: [#141](https://github.com/namhyung/uftrace/issues/141)
* merged: [c618ccc](https://github.com/namhyung/uftrace/commit/c618ccc)

## 2. (python) script enhancement
* arguments / retval support for python [#144](https://github.com/namhyung/uftrace/issues/144)
* option handling in python script (in order to make a given script self-contained)
* script filter (e.g. <symbol>@script=<file> ) or in-script filter
* merged: [66d5e54](https://github.com/namhyung/uftrace/commit/66d5e54)

## 3. automatic argument / retval display for known libc functions
* We can do this for some known functions such as `malloc` and `free`.
* Some functions that use `char*` string such as `strcpy` and `execv` can show eye-catch string output.
* We can support some functions at the beginning and incrementally add one by one.
* As a first step, we can simply provide an option file that contains a list of `-A` and `-R` options for specific functions.
* We can also provide library's return value checker. (such as if `malloc` returns NULL in some cases)
* done: [#158](https://github.com/namhyung/uftrace/pull/158)

## 4. make filtering more convenient
* currently regex and glob matching is used in a mix
* support to select regex, glob or simple substring match based on user's chioce
* consolidate similar options so users don't need to type function name multiple times
* merged: [826687d1](https://github.com/namhyung/uftrace/commit/826687d1)

## 5. TUI support
* current console based UI lacks user interaction
* maybe (ncurses-based) TUI can be added
* done: [#326](https://github.com/namhyung/uftrace/issues/326)

## 6. DWARF debug info support
* it can be used to specify argument and return value automatically
* it provides source location info (file:line)
* it might be used as a new filter option - (e.g. filter all functions in this file)
* done: [#174](https://github.com/namhyung/uftrace/issues/174)

## 7. full dynamic tracing
* compiler support free tracing without recompilation.
* It requires to decode each instruction and emulate.
* done: [#281](https://github.com/namhyung/uftrace/issues/281)
* merged: [ee32dbb](https://github.com/namhyung/uftrace/commit/ee32dbb)

## 8. handling daemon programs
* We need to implement a way to trigger tracing on and off using signal or socket.
* I prefer implementing in socket so that it can be used in embedded system.
* It can be used with `recv` command.
* We can trace multiple processes with data file merge feature.
* done: [#340](https://github.com/namhyung/uftrace/issues/340)
* merged: [fa2abec](https://github.com/namhyung/uftrace/commit/fa2abec)

# Misc Items

## 1. more C++ std::* support
* understanding STL containers
* std::string (done!)
* std::vector, std::list, std::shared_ptr, std::unique_ptr, etc...

## 2. builtin chrome trace viewer
* Currently, showing chrome trace output requires many steps.<br>
\- `uftrace record` -> `uftrace dump --chrome` -> goto `chrome://tracing` -> press `Load` to open file
* I would like to make it as the following steps<br>
\- `uftrace record` -> `uftrace chrome` -> it shows a URL link -> goto the link in any other remote machine.
* It requires integrating chrome trace viewer inside uftrace repo.
* It's mainly to make it simple to use for general users.

