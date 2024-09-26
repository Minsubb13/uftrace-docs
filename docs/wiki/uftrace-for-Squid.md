This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download Squid**
```
$ git clone https://github.com/squid-cache/squid.git
```

**2. Configuration and Build**
```
$ sudo apt-get install build-essential autoconf pkg-config liblzo2-dev libtool libssl-dev libpam0g-dev libssl1.0.0 openssl libsystemd-dev
$ ./bootstrap.sh
$ ./configure
$ make -j4
$ make install
```

**3. Check the generated binary**
```
$ squid
$  netstat -na | grep 3128
tcp        0      0 0.0.0.0:3128            0.0.0.0:*               LISTEN
$ export http_proxy=localhost:3128
$ export https_proxy=localhost:3128
$  wget https://github.com
--2019-10-02 02:38:41--  https://github.com/
Resolving localhost (localhost)... 127.0.0.1, ::1
Connecting localhost (localhost)|127.0.0.1|:3128... connected.
Proxy request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html’

index.html                       [ <=>                                                 ] 131.32K  --.-KB/s    in 0.05s

2019-10-02 02:38:41 (2.75 MB/s) - ‘index.html’ saved [134475]

```

**4. Record httpd with uftrace**

uftrace can now record the target program `Squid`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a squid
$ wget https://github.com
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
 130.862 us [118012] | std::ios_base::Init::Init();
 171.792 us [118012] | getenv("MEMPOOLS") = "NULL";
  16.061 us [118012] | log();
  10.917 us [118012] | socket(AF_INET6, SOCK_STREAM, 0) = 4;
  11.120 us [118012] | std::__ostream_insert();
  65.109 us [118012] | localtime();
  15.092 us [118012] | __vfprintf_chk();
 104.612 us [118012] | close(4) = 0;
  14.675 us [118012] | regcomp();
  20.440 us [118012] | regcomp();
  13.318 us [118012] | fopen("/workspace/karas/squid/build/etc/squid.conf", "r") = 0x56189edd0f10;
  11.434 us [118012] | free(0x56189f0ef7c0);
  10.939 us [118012] | regcomp();
  10.529 us [118012] | calloc(1, 12592) = 0x56189f0f4420;
 423.216 us [118012] | getaddrinfo("goorm", "NULL", 0, 0x7ffe9f363168) = 0;
 518.161 us [118012] | getpwnam();
 324.640 us [118012] | initgroups();
  21.363 us [118012] | fopen("/workspace/karas/squid/build/var/logs/cache.log", "a+") = 0x56189edd0f10;
  27.460 us [118012] | tzset();
  35.912 us [118012] | initgroups();
  12.451 us [118012] | openlog();
 245.796 us [118012] | fork() = 118014;
.....
```

**6. uftrace graph output**

```
$ uftrace graph
# Function Call Graph for 'squid' (session: 328ae985dcbfa595)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
   16.100 ms : (1) squid
  165.865 us :  +-(439) std::ios_base::Init::Init
             :  |
   53.869 us :  +-(573) __cxa_atexit
             :  |
  258.646 us :  +-(1427) malloc
             :  |
   12.733 us :  +-(10) __dynamic_cast
             :  |
   10.461 us :  +-(86) std::_Rb_tree_insert_and_rebalance
             :  |
   10.154 us :  +-(41) __cxa_guard_acquire
             :  |
    4.425 us :  +-(41) __cxa_guard_release
             :  |
  171.792 us :  +-(1) getenv
             :  |
   35.367 us :  +-(486) time
             :  |
  406.057 us :  +-(1976) strlen
             :  |
   99.876 us :  +-(621) memmove
             :  |
   24.769 us :  +-(29) calloc
             :  |
  120.470 us :  +-(755) free
             :  |
...
```