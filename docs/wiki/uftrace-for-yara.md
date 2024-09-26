This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download yara**
```
$ git clone https://github.com/VirusTotal/yara.git
```

**2. Configuration and Build**
```
$ sudo apt-get install automake libtool make gcc flex bison -y
$ ./bootstrap.sh
$ ./configure --disable-shared
$ make -j4
```

**3. Check the generated binary**
```
$ echo "rule dummy { condition: true }" > my_first_rule
$ ./yara my_first_rule my_first_rule
dummy my_first_rule
```

**4. Record yara with uftrace**

uftrace can now record the target program `yara`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a ./yara my_first_rule my_first_rule
dummy my_first_rule
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 12887] | main() {
   0.513 us [ 12887] |   args_parse();
   4.069 us [ 12887] |   time();
   3.218 us [ 12887] |   srand();
   1.316 us [ 12887] |   __ctype_tolower_loc();
            [ 12887] |   yr_thread_storage_create() {
   1.000 us [ 12887] |     pthread_key_create();
   2.414 us [ 12887] |   } /* yr_thread_storage_create */
            [ 12887] |   yr_thread_storage_create() {
   0.200 us [ 12887] |     pthread_key_create();
   0.819 us [ 12887] |   } /* yr_thread_storage_create */
   1.553 us [ 12887] |   CRYPTO_num_locks();
   1.694 us [ 12887] |   CRYPTO_malloc();
   0.189 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   2.209 us [ 12887] |     pthread_mutex_init(0x73b110, 0) = 0;
   3.516 us [ 12887] |   } /* yr_mutex_create */
   0.178 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.306 us [ 12887] |     pthread_mutex_init(0x73b138, 0) = 0;
   1.025 us [ 12887] |   } /* yr_mutex_create */
   0.159 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.279 us [ 12887] |     pthread_mutex_init(0x73b160, 0) = 0;
   0.871 us [ 12887] |   } /* yr_mutex_create */
   0.159 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.236 us [ 12887] |     pthread_mutex_init(0x73b188, 0) = 0;
   0.794 us [ 12887] |   } /* yr_mutex_create */
   0.160 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.228 us [ 12887] |     pthread_mutex_init(0x73b1b0, 0) = 0;
   0.736 us [ 12887] |   } /* yr_mutex_create */
   0.160 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.223 us [ 12887] |     pthread_mutex_init(0x73b1d8, 0) = 0;
   0.714 us [ 12887] |   } /* yr_mutex_create */
   0.158 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.220 us [ 12887] |     pthread_mutex_init(0x73b200, 0) = 0;
   0.730 us [ 12887] |   } /* yr_mutex_create */
   0.158 us [ 12887] |   CRYPTO_num_locks();
            [ 12887] |   yr_mutex_create() {
   0.216 us [ 12887] |     pthread_mutex_init(0x73b228, 0) = 0;
...
```

**6. uftrace tui output**

```
$ uftrace tui
  TOTAL TIME : FUNCTION
    3.567 ms : (1) yara
    3.567 ms : (1) main
    0.512 us :  ├─(1) args_parse
             :  │
    3.768 us :  ├─(1) time
             :  │
    3.365 us :  ├─(1) srand
             :  │
    1.440 us :  ├─(1) __ctype_tolower_loc
             :  │
    3.405 us :  ├─(2) yr_thread_storage_create
    1.427 us :  │ (2) pthread_key_create
             :  │
   15.455 us :  ├─(85) CRYPTO_num_locks
             :  │
    1.865 us :  ├─(1) CRYPTO_malloc
             :  │
   33.033 us :  ├─(41) yr_mutex_create
   11.205 us :  │ (41) pthread_mutex_init
             :  │
    1.924 us :  ├─(2) CRYPTO_THREADID_set_callback
             :  │
    1.568 us :  ├─(2) CRYPTO_set_locking_callback
             :  │
    1.544 us :  ├─(1) yr_modules_initialize
             :  │
    0.476 us :  ├─(2) yr_set_configuration
             :  │
   62.812 us :  ├─(1) yr_compiler_create
    2.377 us :  │  ├─(1) calloc
             :  │  │
   25.515 us :  │  ├─(3) yr_hash_table_create
    2.035 us :  │  │  ├─(3) malloc
             :  │  │  │
   19.543 us :  │  │  └─(3) memset
             :  │  │
   27.925 us :  │  ├─(10) yr_arena_create
   16.298 us :  │  │ (30) malloc
             :  │  │
    1.792 us :  │  └─(1) yr_ac_automaton_create
    0.577 us :  │    (2) malloc
             :  │
    0.277 us :  ├─(1) yr_compiler_set_callback
...
```

**7. uftrace graph output**

```
$ uftrace graph
# Function Call Graph for 'yara' (session: ec91dda37180cfb3)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    3.805 ms : (1) yara
    3.805 ms : (1) main
    0.600 us :  +-(1) args_parse
             :  |
    3.415 us :  +-(1) time
             :  |
    3.392 us :  +-(1) srand
             :  |
    1.302 us :  +-(1) __ctype_tolower_loc
             :  |
    3.286 us :  +-(2) yr_thread_storage_create
    1.229 us :  | (2) pthread_key_create
             :  |
   15.674 us :  +-(85) CRYPTO_num_locks
             :  |
    1.745 us :  +-(1) CRYPTO_malloc
             :  |
   33.017 us :  +-(41) yr_mutex_create
   11.573 us :  | (41) pthread_mutex_init
             :  |
    1.741 us :  +-(2) CRYPTO_THREADID_set_callback
             :  |
    1.597 us :  +-(2) CRYPTO_set_locking_callback
             :  |
    1.380 us :  +-(1) yr_modules_initialize
             :  |
    0.511 us :  +-(2) yr_set_configuration
             :  |
   62.039 us :  +-(1) yr_compiler_create
    2.516 us :  |  +-(1) calloc
             :  |  |
   23.920 us :  |  +-(3) yr_hash_table_create
    2.216 us :  |  |  +-(3) malloc
             :  |  |  |
   17.732 us :  |  |  +-(3) memset
             :  |  |
   28.166 us :  |  +-(10) yr_arena_create
   16.076 us :  |  | (30) malloc
             :  |  |
    1.839 us :  |  +-(1) yr_ac_automaton_create
    0.596 us :  |    (2) malloc
             :  |
...
```