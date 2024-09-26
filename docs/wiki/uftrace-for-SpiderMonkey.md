This document is written by [Handong Choi](https://github.com/henrychoi7).

#### 1. Download SpiderMonkey source code

```
$ wget http://ftp.mozilla.org/pub/spidermonkey/prereleases/59/pre1/mozjs-59.0a1.0.tar.bz2
$ tar xvf mozjs-59.0a1.0.tar.bz2
$ cd mozjs-59.0a1.0.tar.bz2
```

#### 2. Install build tools and dependencies

Following command will install all system prerequisites:
```
$ wget -O bootstrap.py https://hg.mozilla.org/mozilla-central/raw-file/default/python/mozboot/bin/bootstrap.py && python bootstrap.py
```

(Optional) If you want to install autoconf-2.13 from source:
```
$ wget https://ftp.gnu.org/gnu/autoconf/autoconf-2.13.tar.gz
$ tar -xvzf autoconf-2.13.tar.gz
$ cd autoconf-2.13/
$ ./configure --program-suffix=2.13
$ make
$ sudo make install
$ cd ..
```

#### 3. Build configuration

SpiderMonkey doesn't support building in the source directory. Configure and build in a separate build directory, as shown below.
```
$ cd js/src

# If it doesn't work, use autoconf2.13
$ autoconf-2.13

$ mkdir build_DBG.OBJ
$ cd build_DBG.OBJ

$ CFLAGS='-pg' CXXFLAGS='-pg' ../configure --enable-debug --disable-optimize
$ make
```

#### 4. Test and check the generated JS interpreter

```
$ cd js/src/shell/
$ ./js -e 'print("Hello")'
Hello
$ nm ./js | grep mcount
                 U mcount
```

#### 5. Run JS interpreter with uftrace

```
$ uftrace -F main -D 1 --no-event ./js -e 'print("Hello")'
Hello
# DURATION     TID     FUNCTION
   1.731  s [ 85571] | main();
```

#### 6. Generate dump output

```
$ uftrace record -F main -D 2 -- ./js -e 'print("Hello")'
Hello
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 85546] | main() {
...
```

#### 7. Visualized trace output of SpiderMonkey

TUI:
```
  TOTAL TIME : FUNCTION
    1.783  s : (1) js
    1.783  s : (1) main
    0.121 us :  ├─(1) PreInit
             :  │
   77.746 us :  ├─(1) setlocale
             :  │
    0.107 us :  ├─(2) js::shell::RCFile::RCFile
             :  │
    0.092 us :  ├─(2) js::shell::RCFile::acquire
             :  │
    0.678 us :  ├─(2) SetOutputFile
             :  │
    2.346 ms :  ├─(1) JS_Init
  688.216 us :  │ (2) linux:schedule
             :  │
    1.040 us :  ├─(3) mozilla::MakeScopeExit
             :  │
    1.104 us :  ├─(1) js::cli::OptionParser::OptionParser
             :  │
    0.045 us :  ├─(1) js::cli::OptionParser::setDescription
             :  │
    0.045 us :  ├─(1) js::cli::OptionParser::setDescriptionWidth
             :  │
    0.048 us :  ├─(1) js::cli::OptionParser::setHelpWidth
             :  │
    0.060 us :  ├─(1) JS_GetImplementationVersion
             :  │
    0.048 us :  ├─(1) js::cli::OptionParser::setVersion
             :  │
   44.457 us :  ├─(4) js::cli::OptionParser::addMultiStringOption
             :  │
  115.763 us :  ├─(42) js::cli::OptionParser::addBoolOption
             :  │
   62.388 us :  ├─(24) js::cli::OptionParser::addStringOption
             :  │
    7.004 us :  ├─(1) js::cli::OptionParser::addOptionalStringArg
             :  │
   10.892 us :  ├─(1) js::cli::OptionParser::addOptionalMultiStringArg
             :  │
...
uftrace graph: source location is not available [at 0x55c509ea30f8]              0%
```

Graph:
```
$ uftrace graph
# Function Call Graph for 'js' (session: 4700ef4f18dc95d1)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    1.783  s : (1) js
    1.783  s : (1) main
    0.121 us :  +-(1) PreInit
             :  |
   77.746 us :  +-(1) setlocale
             :  |
    0.107 us :  +-(2) js::shell::RCFile::RCFile
             :  |
    0.092 us :  +-(2) js::shell::RCFile::acquire
             :  |
    0.678 us :  +-(2) SetOutputFile
             :  |
    2.346 ms :  +-(1) JS_Init
  688.216 us :  | (2) linux:schedule
             :  |
    1.040 us :  +-(3) mozilla::MakeScopeExit
             :  |
    1.104 us :  +-(1) js::cli::OptionParser::OptionParser
             :  |
    0.045 us :  +-(1) js::cli::OptionParser::setDescription
             :  |
    0.045 us :  +-(1) js::cli::OptionParser::setDescriptionWidth
             :  |
    0.048 us :  +-(1) js::cli::OptionParser::setHelpWidth
             :  |
    0.060 us :  +-(1) JS_GetImplementationVersion
             :  |
    0.048 us :  +-(1) js::cli::OptionParser::setVersion
             :  |
   44.457 us :  +-(4) js::cli::OptionParser::addMultiStringOption
             :  |
  115.763 us :  +-(42) js::cli::OptionParser::addBoolOption
             :  |
   62.388 us :  +-(24) js::cli::OptionParser::addStringOption
             :  |
    7.004 us :  +-(1) js::cli::OptionParser::addOptionalStringArg
             :  |
...
```

Chrome:
```
$ uftrace record -t 5ms ./js -e 'print("Hello")'
$ uftrace dump --chrome > spidermonkey.json
$ trace2html spidermonkey.json
spidermonkey.html
```

Flame graph:
```
$ uftrace record -t 5ms ./js -e 'print("Hello")'
$ uftrace dump --flame-graph | ./flamegraph.pl > spidermonkey.svg
```
