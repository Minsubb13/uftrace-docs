This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download cURL and Dependencies**
```
$ git clone https://github.com/curl/curl.git
$ sudo apt install autoconf
$ sudo apt-get install libtool
```

**2. Configuration and Build**
```
$ cd curl
$ ./buildconf
$ ./configure
$ make -j4
$ make install
```

**3. Check the generated binary**
```
$ curl -L -k -s -o /dev/null -w "%{http_code}\n"  https://github.com
200
```

**4. Record curl with uftrace**

uftrace can now record the target program `curl`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a ./curl -L -k -s -o /dev/null -w "%{http_code}\n"  https://github.com
200
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [  9508] | main() {
   9.247 us [  9508] |   pipe();
   4.270 us [  9508] |   close(4) = 0;
   5.333 us [  9508] |   close(5) = 0;
   2.550 us [  9508] |   signal(SIGPIPE, 0x1) = 0;
   1.582 us [  9508] |   malloc(1208) = 0x133c300;
   1.703 ms [  9508] |   curl_global_init();
            [  9508] |   get_libcurl_info() {
   0.685 us [  9508] |     curl_version_info();
   6.379 us [  9508] |     curl_strequal();
   0.490 us [  9508] |     curl_strequal();
   0.193 us [  9508] |     curl_strequal();
   0.307 us [  9508] |     curl_strequal();
   0.160 us [  9508] |     curl_strequal();
   0.167 us [  9508] |     curl_strequal();
   0.167 us [  9508] |     curl_strequal();
   0.167 us [  9508] |     curl_strequal();
            [  9508] |     tool_setopt() {
   0.325 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_SSL_VERIFYPEER") = -13;
   0.317 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_SSL_VERIFYHOST") = -13;
   0.356 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_SSL_ENABLE_NPN") = -13;
   0.319 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_SSL_ENABLE_ALPN") = -13;
   0.314 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_TCP_NODELAY") = -14;
   0.318 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_PROXY_SSL_VERIFYPEER") = -10;
   0.323 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_PROXY_SSL_VERIFYHOST") = -10;
   0.317 us [  9508] |       strcmp("CURLOPT_FAILONERROR", "CURLOPT_SOCKS5_AUTH") = -13;
   0.258 us [  9508] |       curl_msnprintf();
   0.272 us [  9508] |       curl_easy_setopt();
   0.199 us [  9508] |       free(0);
   6.870 us [  9508] |     } /* tool_setopt */
.....
```


**6. uftrace tui output**

```
$ uftrace tui
  TOTAL TIME : FUNCTION
  612.844 ms : (1) curl
  612.844 ms : (1) main
    9.247 us :  ├─(1) pipe
             :  │
    9.603 us :  ├─(2) close
             :  │
    2.550 us :  ├─(1) signal
             :  │
    1.582 us :  ├─(1) malloc
             :  │
    1.703 ms :  ├─(1) curl_global_init
             :  │
  101.969 us :  ├─(1) get_libcurl_info
    0.685 us :  │  ├─(1) curl_version_info
             :  │  │
   42.277 us :  │  └─(208) curl_strequal
             :  │
    0.466 us :  ├─(1) config_init
             :  │
  610.614 ms :  ├─(1) operate
  299.910 us :  │  ├─(1) setlocale
    8.017 us :  │  │ (1) linux:schedule
             :  │  │
    0.615 us :  │  ├─(2) curl_strequal
             :  │  │
   16.687 us :  │  ├─(1) parseconfig
    5.863 us :  │  │  ├─(1) homedir
    1.838 us :  │  │  │  ├─(2) getenv
             :  │  │  │  │
    1.078 us :  │  │  │  └─(1) __strdup
             :  │  │  │
    1.589 us :  │  │  ├─(1) curl_maprintf
...
```

**7. uftrace graph output**

```
$ uftrace graph

# Function Call Graph for 'curl' (session: 7a24aa2c5c4c49d3)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
  612.844 ms : (1) curl
  612.844 ms : (1) main
    9.247 us :  +-(1) pipe
             :  |
    9.603 us :  +-(2) close
             :  |
    2.550 us :  +-(1) signal
             :  |
    1.582 us :  +-(1) malloc
             :  |
    1.703 ms :  +-(1) curl_global_init
             :  |
  101.969 us :  +-(1) get_libcurl_info
    0.685 us :  |  +-(1) curl_version_info
             :  |  |
   42.277 us :  |  +-(208) curl_strequal
             :  |
    0.466 us :  +-(1) config_init
             :  |
  610.614 ms :  +-(1) operate
  299.910 us :  |  +-(1) setlocale
    8.017 us :  |  | (1) linux:schedule
             :  |  |
    0.615 us :  |  +-(2) curl_strequal
             :  |  |
   16.687 us :  |  +-(1) parseconfig
    5.863 us :  |  |  +-(1) homedir
    1.838 us :  |  |  |  +-(2) getenv
             :  |  |  |  |
    1.078 us :  |  |  |  +-(1) __strdup
             :  |  |  |
    1.589 us :  |  |  +-(1) curl_maprintf
             :  |  |  |
    1.433 us :  |  |  +-(2) free
             :  |  |  |
    4.763 us :  |  |  +-(1) fopen
             :  |  |
   13.148 us :  |  +-(1) parse_args
    2.193 us :  |  |  +-(1) new_getout
...
```