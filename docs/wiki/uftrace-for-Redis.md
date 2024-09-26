**1. clone redis repositories**
```
$ git clone https://github.com/redis/redis.git
```

**2. build redis with option -pg -g**
```
$ cd ./redis
$ make CFLAGS="-pg -g"
```

**3. record redis-server with uftrace -a**
```
$ cd ./src
$ uftrace record -a ./redis-server
...
```
using option "-a" to record arguments and return values of known functions
<br/>
(detailed description: please check -a, \--auto-args description in https://github.com/namhyung/uftrace/blob/master/doc/uftrace-record.md)

**4. save and get data using redis-cli**
```
$ cd ./redis/src
$ ./redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> SET mykey hello
OK
127.0.0.1:6379> GET mykey
"hello"
127.0.0.1:6379> shutdown
not connected> exit
```
open the another terminal to using redis-cli

**5. uftrace replay result**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [   313] | jemalloc_constructor() {
            [   313] |   malloc_init_hard() {
   6.800 us [   313] |     pthread_mutex_trylock(0x55e642d7b9c0) = 0;
   0.700 us [   313] |     pthread_self();
            [   313] |     malloc_init_hard_a0_locked() {
   0.500 us [   313] |       pthread_self();
            [   313] |       je_sc_boot() {
   7.200 us [   313] |         je_sc_data_init(0x7fff454faf10);
   9.300 us [   313] |       } /* je_sc_boot */
   3.600 us [   313] |       je_bin_shard_sizes_boot(0x7fff454fae70);
            [   313] |       malloc_conf_init_helper(0, 0, 1, 0x7fff454fae50, "<B0><CE>OE^B") {
   0.800 us [   313] |         __errno_location();
   3.500 us [   313] |         readlink();
   0.900 us [   313] |         secure_getenv();
 175.700 us [   313] |       } /* malloc_conf_init_helper */
   1.000 us [   313] |       malloc_conf_init_helper(0x7fff454faf10, 0x7fff454fae70, 0, 0x7fff454fae50, "NULL");
  26.400 us [   313] |       je_sz_boot(0x7fff454faf10);
   2.500 us [   313] |       je_bin_boot(0x7fff454faf10, 0x7fff454fae70);
            [   313] |       je_pages_boot() {
   1.000 us [   313] |         sysconf();
 217.500 us [   313] |         syscall(2) = 4;
  56.200 us [   313] |         syscall(0) = 1;
   2.400 us [   313] |         syscall(3) = 0;
  39.200 us [   313] |         syscall(2) = 4;
   3.300 us [   313] |         syscall(0) = 23;
   2.300 us [   313] |         syscall(3) = 0;
   1.300 us [   313] |         strncmp("[always] madvise never\n", "always [madvise] never\n", 23) = -6;
   0.800 us [   313] |         strncmp("[always] madvise never\n", "[always] madvise never\n", 23) = 0;
```

**6. uftrace report result**
```
$ uftrace report
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================
    5.024  m    5.024  m         635  linux:schedule
    3.016  m    3.484 ms           3  bioProcessBackgroundJobs
    1.005  m  543.200 us           1  main
    1.005  m    1.141 ms           1  background_thread_entry
    1.005  m    1.769 ms           1  aeMain
    1.005  m   17.099 ms         628  aeProcessEvents
    1.002  m   17.283 ms         628  epoll_wait
    2.973  s   21.272 ms         621  serverCron
    2.813  s    6.748 ms         621  cronUpdateMemoryStats
    2.699  s    7.203 ms         621  zmalloc_get_allocator_info
    2.697  s   10.290 ms        2485  je_mallctl
    2.687  s   14.885 ms        2485  je_ctl_byname
    2.522  s    4.576 ms         621  epoch_ctl
    2.517  s   39.938 ms         622  ctl_refresh
    2.258  s    2.421 ms         622  ctl_arena_stats_amerge
    2.256  s    1.225  s         622  je_arena_stats_merge
  444.569 ms  444.569 ms      371334  je_extents_nextents_get
  442.639 ms  442.625 ms      371334  je_extents_nbytes_get
  381.158 ms   41.800 us           7  connSocketEventHandler
  380.931 ms   80.000 us           5  readQueryFromClient
  380.459 ms   75.200 us           5  processInputBuffer
  380.015 ms   22.200 us           5  processCommandAndResetClient
  379.759 ms   94.800 us           5  processCommand
  379.425 ms  160.100 us           5  call
  353.185 ms  716.100 us           1  commandDocsCommand
  345.139 ms    3.031 ms         366  addReplyCommandDocs
  292.545 ms   19.730 ms        9226  addReplyBulkCString
```

**7. uftrace tui result**
```
  TOTAL TIME : FUNCTION
    5.028  m : (1) redis-server                                                                                             
    9.118 ms :  ├─(1) jemalloc_constructor
    9.113 ms :  │ (1) malloc_init_hard
    7.400 us :  │  ├─(2) pthread_mutex_trylock
             :  │  │
    0.700 us :  │  ├─(1) pthread_self
             :  │  │
    6.171 ms :  │  ├─(1) malloc_init_hard_a0_locked
    0.500 us :  │  │  ├─(1) pthread_self
             :  │  │  │
    9.300 us :  │  │  ├─(1) je_sc_boot
    7.200 us :  │  │  │ (1) je_sc_data_init
             :  │  │  │
    3.600 us :  │  │  ├─(1) je_bin_shard_sizes_boot
             :  │  │  │
  176.700 us :  │  │  ├─(2) malloc_conf_init_helper
    0.800 us :  │  │  │  ├─(1) __errno_location
             :  │  │  │  │
    3.500 us :  │  │  │  ├─(1) readlink
             :  │  │  │  │
    0.900 us :  │  │  │  └─(1) secure_getenv
             :  │  │  │
   26.400 us :  │  │  ├─(1) je_sz_boot
             :  │  │  │
    2.500 us :  │  │  ├─(1) je_bin_boot
             :  │  │  │
  350.300 us :  │  │  ├─(1) je_pages_boot
    1.000 us :  │  │  │  ├─(1) sysconf
```

**8. uftrace graph result**
```
$ uftrace graph
# Function Call Graph for 'redis-server' (session: 4f527f0760296c24)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    5.028  m : (1) redis-server
    9.118 ms :  +-(1) jemalloc_constructor
    9.113 ms :  | (1) malloc_init_hard
    7.400 us :  |  +-(2) pthread_mutex_trylock
             :  |  |
    0.700 us :  |  +-(1) pthread_self
             :  |  |
    6.171 ms :  |  +-(1) malloc_init_hard_a0_locked
    0.500 us :  |  |  +-(1) pthread_self
             :  |  |  |
    9.300 us :  |  |  +-(1) je_sc_boot
    7.200 us :  |  |  | (1) je_sc_data_init
             :  |  |  |
    3.600 us :  |  |  +-(1) je_bin_shard_sizes_boot
             :  |  |  |
  176.700 us :  |  |  +-(2) malloc_conf_init_helper
    0.800 us :  |  |  |  +-(1) __errno_location
             :  |  |  |  |
    3.500 us :  |  |  |  +-(1) readlink
             :  |  |  |  |
    0.900 us :  |  |  |  +-(1) secure_getenv
             :  |  |  |
   26.400 us :  |  |  +-(1) je_sz_boot
             :  |  |  |
    2.500 us :  |  |  +-(1) je_bin_boot
             :  |  |  |
```