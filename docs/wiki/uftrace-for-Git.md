This document is written by [Handong Choi](https://github.com/henrychoi7).

#### 1. Download the latest Git source code

```
$ wget https://github.com/git/git/archive/v2.23.0.tar.gz
$ tar -zxf v2.23.0.tar.gz
$ cd git-2.23.0/
```

#### 2. Install minimal dependencies for compiling the Git binaries

(Optional) If you need to install required packages:
```
$ sudo apt-get install dh-autoreconf libcurl4-gnutls-dev libexpat1-dev \
  gettext libz-dev libssl-dev install-info
```

#### 3. Building Git from source using CMake

Run `configure` with `-g -pg` options. Also, you can add some optional parameters (e.g. `--prefix=/usr`) as well but it's not necessary for now.
```
$ autoreconf

$ CFLAGS='-pg -g' CXXFLAGS='-pg -g' ./configure
```

Execute `make`, it's recommended to do this in several threads.
```
$ make -j4 all
```

#### 4. Test and check the generated Git binary

```
$ ./git --version
git version 2.23.0

$ nm ./git | grep mcount
                 U mcount@@GLIBC_2.2.5
```

#### 5. Run Git with uftrace

```
$ uftrace -F main -D 2 --no-event ./git -version
git version 2.23.0
# DURATION     TID     FUNCTION
            [ 61730] | main() {
  11.491 us [ 61730] |   trace2_initialize_clock();
  14.944 us [ 61730] |   sanitize_stdfds();
   5.925 us [ 61730] |   restore_sigpipe_to_default();
   0.186 us [ 61730] |   git_resolve_executable_dir();
  87.670 us [ 61730] |   trace2_initialize_fl();
   0.164 us [ 61730] |   trace2_cmd_start_fl();
   7.935 us [ 61730] |   git_setup_gettext();
  28.255 us [ 61730] |   initialize_the_repository();
   1.379 us [ 61730] |   attr_start();
            [ 61730] |   cmd_main() {

uftrace stopped tracing with remaining functions
================================================
task: 61730
[1] cmd_main
[0] main
```

#### 6. Generate dump output

```
$ uftrace record -F main -D 2 --no-event ./git --version
git version 2.23.0
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 61743] | main() {
  16.120 us [ 61743] |   trace2_initialize_clock();
  14.147 us [ 61743] |   sanitize_stdfds();
   5.992 us [ 61743] |   restore_sigpipe_to_default();
   0.174 us [ 61743] |   git_resolve_executable_dir();
  68.137 us [ 61743] |   trace2_initialize_fl();
   0.185 us [ 61743] |   trace2_cmd_start_fl();
  14.070 us [ 61743] |   git_setup_gettext();
  46.961 us [ 61743] |   initialize_the_repository();
   1.675 us [ 61743] |   attr_start();
            [ 61743] |   cmd_main() {

uftrace stopped tracing with remaining functions
================================================
task: 61743
[1] cmd_main
[0] main
```

#### 7. Visualized trace output of Git

TUI:
```
$ uftrace tui
  TOTAL TIME : FUNCTION
  203.549 us : (1) git
  203.549 us : (1) main
   16.120 us :  ├─(1) trace2_initialize_clock
             :  │
   14.147 us :  ├─(1) sanitize_stdfds
             :  │
    5.992 us :  ├─(1) restore_sigpipe_to_default
             :  │
    0.174 us :  ├─(1) git_resolve_executable_dir
             :  │
   68.137 us :  ├─(1) trace2_initialize_fl
             :  │
    0.185 us :  ├─(1) trace2_cmd_start_fl
             :  │
   14.070 us :  ├─(1) git_setup_gettext
             :  │
   46.961 us :  ├─(1) initialize_the_repository
             :  │
    1.675 us :  ├─(1) attr_start
             :  │
             :  └─(1) cmd_main

uftrace graph: session 3ab8aab308fdfb5e (/home/henry/git-2.23.0/git)                             0%
```

Graph:
```
$ uftrace graph
# Function Call Graph for 'git' (session: 3ab8aab308fdfb5e)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
  203.549 us : (1) git
  203.549 us : (1) main
   16.120 us :  +-(1) trace2_initialize_clock
             :  |
   14.147 us :  +-(1) sanitize_stdfds
             :  |
    5.992 us :  +-(1) restore_sigpipe_to_default
             :  |
    0.174 us :  +-(1) git_resolve_executable_dir
             :  |
   68.137 us :  +-(1) trace2_initialize_fl
             :  |
    0.185 us :  +-(1) trace2_cmd_start_fl
             :  |
   14.070 us :  +-(1) git_setup_gettext
             :  |
   46.961 us :  +-(1) initialize_the_repository
             :  |
    1.675 us :  +-(1) attr_start
             :  |
             :  +-(1) cmd_main
```

Chrome:
```
$ uftrace record -t 1ms ./git --version
git version 2.23.0

$ uftrace dump --chrome > git.json

$ trace2html git.json
git.html
```

Flame graph:
```
$ uftrace record -t 1ms ./git --version
git version 2.23.0

$ uftrace dump --flame-graph | ./flamegraph.pl >git.svg
```