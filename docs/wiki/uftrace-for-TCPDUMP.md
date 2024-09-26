This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download TCPDUMP**
```
$ git clone https://github.com/the-tcpdump-group/libpcap.git
$ git clone https://github.com/the-tcpdump-group/tcpdump.git
```

**2. Configuration and Build**
```
$ sudo apt-get install flex bison gcc make
$ cd libpcap/ && ./configure && make -j4 && make install
$ cd ../tcpdump/ && ./configure && make -j4 && make install
```

**3. Check the generated binary**
```
$ tcpdump
01:28:34.315311 IP ip-172-18-0-14.ap-northeast-2.compute.internal.ssh > ip-172-18-0-1.ap-northeast-2.compute.internal.55470: Flags [P.], seq 6449552:6449828, ack 3349, win
78, options [nop,nop,TS val 37588058 ecr 37588054], length 276
01:28:34.315352 IP ip-172-18-0-14.ap-northeast-2.compute.internal.ssh > ip-172-18-0-1.ap-northeast-2.compute.internal.55470: Flags [P.], seq 6449828:6450104, ack 3349, win
78, options [nop,nop,TS val 37588058 ecr 37588054], length 276
01:28:34.315391 IP ip-172-18-0-14.ap-northeast-2.compute.internal.ssh > ip-172-18-0-1.ap-northeast-2.compute.internal.55470: Flags [P.], seq 6450104:6450380, ack 3349, win
78, options [nop,nop,TS val 37588058 ecr 37588054], length 276
```

**4. Record tcpdump with uftrace**

uftrace can now record the target program `TCPDUMP`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a tcpdump
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 11415] | nd_init("", 256) {
   7.085 us [ 11415] |   strlcpy("", "", 256) = 0;
 219.616 us [ 11415] | } = 0; /* nd_init */
   0.867 us [ 11415] | strrchr("tcpdump", '/') = "NULL";
   1.880 us [ 11415] | getopt_long(1, 0x7fff390057c8, "aAbB:c:C:dDeE:fF:G:hHi:Ij:JKlLm:M:nNOpqQ:r:s:StT:uUvV:w:W:xXy:Yz:Z:#") = -1;
   3.588 us [ 11415] | isatty(1) = 1;
            [ 11415] | pcap_findalldevs(0x7fff39004308, "") {
            [ 11415] |   pcap_findalldevs_interfaces(0x7fff39004308, "") {
  68.087 us [ 11415] |     getifaddrs();
   0.515 us [ 11415] |     strchr("lo", ':') = "NULL";
            [ 11415] |     add_addr_to_iflist(0x7fff390041f8, "lo", 65609, 0x559a5a8dbcb8, 20, 0, 20, 0, 0, 0, 0, "") {
            [ 11415] |       add_or_find_if(0x7fff39004120, 0x7fff390041f8, "lo", 65609, "NULL", "") {
            [ 11415] |         pcap_create("lo", "<B5>c<C6>X^C") {
            [ 11415] |           can_create("lo", "<B5>c<C6>X^C", 0x7fff39003f84) {
   0.290 us [ 11415] |             strrchr("lo", '/') = "NULL";
   6.509 us [ 11415] |           } = 0; /* can_create */
            [ 11415] |           usb_create("lo", "<B5>c<C6>X^C", 0x7fff39003f84) {
.....
```


**6. uftrace tui output**

```
$ uftrace tui
  TOTAL TIME : FUNCTION
    5.621  s : (1) tcpdump
  219.616 us :  ├─(1) nd_init
    7.085 us :  │ (1) strlcpy
             :  │
    0.867 us :  ├─(1) strrchr
             :  │
    1.880 us :  ├─(1) getopt_long
             :  │
    3.588 us :  ├─(1) isatty
             :  │
   82.156 ms :  ├─(1) pcap_findalldevs
   71.195 ms :  │  ├─(1) pcap_findalldevs_interfaces
   68.087 us :  │  │  ├─(1) getifaddrs
             :  │  │  │
    2.183 us :  │  │  ├─(4) strchr
             :  │  │  │
   71.115 ms :  │  │  ├─(4) add_addr_to_iflist
   71.074 ms :  │  │  │  ├─(4) add_or_find_if
   98.917 us :  │  │  │  │  ├─(2) pcap_create
   22.237 us :  │  │  │  │  │  ├─(2) can_create
    0.656 us :  │  │  │  │  │  │ (2) strrchr
             :  │  │  │  │  │  │
    6.575 us :  │  │  │  │  │  ├─(2) usb_create
    0.371 us :  │  │  │  │  │  │ (2) strrchr
             :  │  │  │  │  │  │
    2.570 us :  │  │  │  │  │  ├─(2) netfilter_create
    0.400 us :  │  │  │  │  │  │ (2) strrchr
             :  │  │  │  │  │  │
...
```

**7. uftrace graph output**

```
$ uftrace graph
# Function Call Graph for 'tcpdump' (session: 3f4fff6e390dee8c)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    5.621  s : (1) tcpdump
  219.616 us :  +-(1) nd_init
    7.085 us :  | (1) strlcpy
             :  |
    0.867 us :  +-(1) strrchr
             :  |
    1.880 us :  +-(1) getopt_long
             :  |
    3.588 us :  +-(1) isatty
             :  |
   82.156 ms :  +-(1) pcap_findalldevs
   71.195 ms :  |  +-(1) pcap_findalldevs_interfaces
   68.087 us :  |  |  +-(1) getifaddrs
             :  |  |  |
    2.183 us :  |  |  +-(4) strchr
             :  |  |  |
   71.115 ms :  |  |  +-(4) add_addr_to_iflist
   71.074 ms :  |  |  |  +-(4) add_or_find_if
   98.917 us :  |  |  |  |  +-(2) pcap_create
   22.237 us :  |  |  |  |  |  +-(2) can_create
...
```