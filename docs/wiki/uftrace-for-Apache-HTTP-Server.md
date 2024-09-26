This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download Apache HTTP Server**
```
$ git clone https://github.com/apache/httpd.git
$ git clone https://github.com/apache/apr.git
$ git clone https://github.com/apache/apr-util.git
```

**2. Configuration and Build**
```
$ sudo apt-get install git python openssl gcc autoconf make libtool-bin libpcre3-dev libxml2  libexpat1 libexpat1-dev wget tar
$ mv apr httpd/srclib/
$ mv apr-util httpd/srclib/
$ ./buildconf
$ ./configure
$ make -j4
$ make install
```

**3. Check the generated binary**
```
$ httpd
$ curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

**4. Record httpd with uftrace**

uftrace can now record the target program `Apache HTTP Server`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a httpd
$ curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
  29.420 us [ 33113] | apr_app_initialize();
   0.348 us [ 33113] | apr_pool_create_ex();
   0.161 us [ 33113] | apr_pool_abort_set();
   0.064 us [ 33113] | apr_pool_tag();
   2.340 us [ 33113] | apr_file_open_stderr();
   0.076 us [ 33113] | apr_palloc();
   3.340 us [ 33113] | reset_process_pconf(0x564a53a35ba0);
   5.218 us [ 33113] | apr_filepath_name_get();
   8.746 ms [ 33113] | ap_init_rng(0x564a53a35aa8);
   0.141 us [ 33113] | apr_pool_parent_get();
   0.126 us [ 33113] | apr_pool_abort_set();
   0.460 us [ 33113] | apr_array_make();
   0.116 us [ 33113] | apr_array_make();
   0.076 us [ 33113] | apr_array_make();
 687.054 us [ 33113] | ap_setup_prelinked_modules(0x564a53a35ba0) = "NULL";
   3.534 us [ 33113] | ap_run_rewrite_args(0x564a53a35ba0);
   0.109 us [ 33113] | apr_getopt_init();
   0.077 us [ 33113] | apr_getopt();
   0.140 us [ 33113] | apr_pool_create_ex();
   0.100 us [ 33113] | apr_pool_tag();
   0.321 us [ 33113] | apr_pool_create_ex();
   0.064 us [ 33113] | apr_pool_tag();
  85.501 ms [ 33113] | ap_read_config(0x564a53a35ba0, 0x564a53a6f558, "conf/httpd.conf", &ap_conftree) = 0x564a53a6e048;
   0.140 us [ 33113] | apr_pool_cleanup_register();
  21.299 us [ 33113] | apr_hook_sort_all();
   5.507 us [ 33113] | ap_mutex_init.part.0();
   0.084 us [ 33113] | apr_pool_cleanup_register();
   0.317 us [ 33113] | apr_cpystrn();
  34.508 us [ 33113] | event_pre_config(0x564a53a37ab8, 0x564a53a6b538, 0x564a53a6f558) = 0;
   1.796 us [ 33113] | ap_uname2id("#-1") = -1;
   0.666 us [ 33113] | ap_gname2id("#-1") = -1;
.....
```


**6. uftrace tui output**

```
$ uftrace tui
  TOTAL TIME : FUNCTION
    1.046  m : (1) httpd
   29.420 us :  ├─(1) apr_app_initialize
             :  │
    1.165 us :  ├─(4) apr_pool_create_ex
             :  │
    0.287 us :  ├─(2) apr_pool_abort_set
             :  │
    0.329 us :  ├─(4) apr_pool_tag
             :  │
    2.340 us :  ├─(1) apr_file_open_stderr
             :  │
    0.076 us :  ├─(1) apr_palloc
             :  │
  267.408 us :  ├─(2) reset_process_pconf
    0.259 us :  │  ├─(1) apr_pool_create_ex
             :  │  │
    0.076 us :  │  ├─(1) apr_pool_tag
             :  │  │
    0.421 us :  │  ├─(2) apr_pool_pre_cleanup_register
             :  │  │
  262.374 us :  │  └─(1) apr_pool_clear
    0.352 us :  │     ├─(1) apr_hook_deregister_all
             :  │     │
    1.125 us :  │     ├─(4) ap_pool_cleanup_set_null
             :  │     │
   33.270 us :  │     └─(21) unload_module
   21.204 us :  │       (21) ap_remove_module
    2.570 us :  │       (21) free
             :  │
    5.218 us :  ├─(1) apr_filepath_name_get
             :  │
    8.746 ms :  ├─(1) ap_init_rng
    2.591 us :  │  ├─(1) apr_random_standard_new
             :  │  │
    5.605 ms :  │  ├─(4221) apr_generate_random_bytes
             :  │  │
  951.174 us :  │  ├─(4221) apr_random_add_entropy
             :  │  │
  298.241 us :  │  └─(4221) apr_random_insecure_ready
             :  │
    0.141 us :  ├─(1) apr_pool_parent_get
             :  │
    0.652 us :  ├─(3) apr_array_make
...
```

**7. uftrace graph output**

```
$ uftrace graph

# Function Call Graph for 'httpd' (session: aec00e36b4e17363)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    1.020  m : (1) httpd
   29.420 us :  +-(1) apr_app_initialize
             :  |
    1.165 us :  +-(4) apr_pool_create_ex
             :  |
    0.287 us :  +-(2) apr_pool_abort_set
             :  |
    0.329 us :  +-(4) apr_pool_tag
             :  |
    2.340 us :  +-(1) apr_file_open_stderr
             :  |
    0.076 us :  +-(1) apr_palloc
             :  |
  267.408 us :  +-(2) reset_process_pconf
    0.259 us :  |  +-(1) apr_pool_create_ex
             :  |  |
    0.076 us :  |  +-(1) apr_pool_tag
             :  |  |
    0.421 us :  |  +-(2) apr_pool_pre_cleanup_register
             :  |  |
  262.374 us :  |  +-(1) apr_pool_clear
    0.352 us :  |     +-(1) apr_hook_deregister_all
             :  |     |
    1.125 us :  |     +-(4) ap_pool_cleanup_set_null
             :  |     |
   33.270 us :  |     +-(21) unload_module
   21.204 us :  |       (21) ap_remove_module
    2.570 us :  |       (21) free
             :  |
    5.218 us :  +-(1) apr_filepath_name_get
             :  |
    8.746 ms :  +-(1) ap_init_rng
    2.591 us :  |  +-(1) apr_random_standard_new
             :  |  |
    5.605 ms :  |  +-(4221) apr_generate_random_bytes
...
```