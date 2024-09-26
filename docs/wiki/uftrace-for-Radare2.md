This document is written by [Handong Choi](https://github.com/henrychoi7).

#### 1. Download the latest Radare2 source code

```
$ wget https://github.com/radare/radare2/archive/3.9.0.tar.gz
$ tar -zxf 3.9.0.tar.gz
$ cd radare2-3.9.0/
```

#### 2. Building Radare2 from source using Meson

Build faster with meson + ninja. Also, you can add some optional parameters (e.g. `--prefix=/usr`) as well but it's not necessary for now.
```
$ sudo apt-get install python3 python3-pip python3-setuptools \
                       python3-wheel ninja-build
$ CC=clang LDFLAGS=-fuse-ld=gold CFLAGS='-pg' CXXFLAGS='-pg' meson . debug --buildtype=debug
$ ninja -C debug
$ cd debug/binr/radare2/
```

#### 3. Test and check the generated Radare2 binary

```
$ ./radare2 /bin/ls -c "s 0x10"
[0x00000010]> exit()
$ nm ./radare2 | grep mcount
                 U mcount
```

#### 4. Run Radare2 with uftrace

```
$ uftrace -D 4 --no-event ./radare2 /bin/ls -c "s 0x10"
[0x00000010]> exit()
# DURATION     TID     FUNCTION
            [133159] | main() {
   1.262 us [133159] |   strstr();
            [133159] |   r_main_radare2() {
            [133159] |     r_main_radare2() {
   2.100 us [133159] |       r_list_new();
   0.468 us [133159] |       r_list_new();
   0.431 us [133159] |       r_list_new();
   0.428 us [133159] |       r_list_new();
   4.364 us [133159] |       r_sys_get_environ();
   0.178 us [133159] |       r_sys_set_environ();
   7.200 us [133159] |       r_sys_getenv();
  56.118 ms [133159] |       r_core_init();
   9.906 us [133159] |       r_core_task_sync_begin();
   8.830 us [133159] |       set_color_default();
   0.323 us [133159] |       r_list_append();
   2.777 us [133159] |       r_config_get();
   0.403 us [133159] |       r_sys_getenv();
   1.908 us [133159] |       r_config_get_i();
  49.220 us [133159] |       r_core_loadlibs();
   0.316 us [133159] |       run_commands();
   1.001 us [133159] |       r_list_free();
   0.471 us [133159] |       r_bin_force_plugin();
   2.527 us [133159] |       r_config_get();
  23.806 us [133159] |       radare2_rc();
   2.492 us [133159] |       r_config_get_i();
   4.228 us [133159] |       r_file_is_directory();
   2.096 us [133159] |       r_config_get();
   2.080 us [133159] |       r_config_get();
   1.538 us [133159] |       r_config_get();
 137.880 us [133159] |       r_core_file_open();
   1.223 us [133159] |       r_io_desc_get();
 136.734 ms [133159] |       r_core_bin_load();
   0.444 us [133159] |       r_bin_cur();
  25.671 us [133159] |       r_core_cmd0();
   0.352 us [133159] |       r_io_desc_get();
   0.328 us [133159] |       r_bin_cur_object();
   1.561 us [133159] |       r_flag_get();
  12.405 us [133159] |       r_core_seek();
   6.617 us [133159] |       r_core_seek();
   2.038 us [133159] |       r_config_get();
   2.195 us [133159] |       r_config_get();
   0.264 us [133159] |       r_core_project_open();
   0.315 us [133159] |       r_io_desc_get();
   1.459 us [133159] |       r_config_get_i();
   6.385 ms [133159] |       r_bin_file_hash();
   2.023 us [133159] |       r_config_get();
   1.407 us [133159] |       r_flag_space_set();
   1.293 us [133159] |       r_str_newf();
   4.421 us [133159] |       r_file_exists();
   2.138 us [133159] |       r_str_r2_prefix();
   2.095 us [133159] |       r_file_exists();
  35.010 us [133159] |       r_core_cmd_str();
   1.825 us [133159] |       r_config_get_i();
  45.408 us [133159] |       run_commands();
   0.808 us [133159] |       r_list_free();
   0.358 us [133159] |       r_list_free();
   0.350 us [133159] |       r_list_free();
   1.641 us [133159] |       r_config_get_i();
   1.757 us [133159] |       r_config_get_i();
  22.649 us [133159] |       r_core_fortune_print_random();
   1.484 us [133159] |       r_cons_flush();
   0.477 us [133159] |       r_flag_space_set();
 863.790 us [133159] |       r_core_cmd0();
   1.617 us [133159] |       loading_stop();
   3.564  s [133159] |       r_core_prompt();
  19.761 us [133159] |       r_core_prompt_exec();
   3.330 us [133159] |       r_config_get_i();
   0.181 us [133159] |       r_cons_is_interactive();
   3.219 us [133159] |       r_core_task_running_tasks_count();
   2.168 us [133159] |       r_config_get();
   2.335 us [133159] |       r_str_newf();
   1.582 us [133159] |       r_config_get_i();
   1.770 us [133159] |       mustSaveHistory();
 168.359 us [133159] |       r_line_hist_save();
   4.471 us [133159] |       r_core_task_sync_end();
   5.866 ms [133159] |       r_core_fini();
   0.358 us [133159] |       r_cons_set_raw();
   0.526 us [133159] |       r_str_const_free();
   0.211 us [133159] |       r_cons_free();
   0.191 us [133159] |       r_list_free();
   0.192 us [133159] |       r_list_free();
   0.187 us [133159] |       r_list_free();
   0.187 us [133159] |       r_list_free();
   3.770  s [133159] |     } /* r_main_radare2 */
   3.770  s [133159] |   } /* r_main_radare2 */
   3.770  s [133159] | } /* main */
```

#### 5. Generate dump output

```
$ uftrace record -D 4 --no-event ./radare2 /bin/ls -c "s 0x10"
[0x00000010]> exit()
$ uftrace replay
# DURATION     TID     FUNCTION
            [134528] | main() {
...
```

#### 6. Visualized trace output of Radare2

TUI:
```
  TOTAL TIME : FUNCTION
    1.921  s : (1) radare2
    1.921  s : (1) main
    1.141 us :  ├─(1) strstr
             :  │
    1.921  s :  └─(1) r_main_radare2
    1.921  s :   ▶(1) r_main_radare2

uftrace graph: source location is not available [at 0x7fe5cc8dcdda]                100%
```

Graph:
```
$ uftrace graph
# Function Call Graph for 'radare2' (session: f68137622e006528)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    1.921  s : (1) radare2
    1.921  s : (1) main
    1.141 us :  +-(1) strstr
             :  |
    1.921  s :  +-(1) r_main_radare2
    1.921  s :    (1) r_main_radare2
...
```

Chrome:
```
$ uftrace record -t 1ms --no-event ./radare2 /bin/ls -c "s 0x10"
[0x00000010]> exit()
$ uftrace dump --chrome > radare2.json
$ trace2html radare2.json
radare2.html
```

Flame graph:
```
$ uftrace record -t 1ms --no-event ./radare2 /bin/ls -c "s 0x10"
[0x00000010]> exit()
$ uftrace dump --flame-graph | ./flamegraph.pl >radare2.svg
```