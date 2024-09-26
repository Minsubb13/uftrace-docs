This document is written by [Sangyun Han](https://github.com/sangyun-han).

## What is LXC?
LXC(LinuX Container) is well-known low level linux container runtime. It used to be a part of Docker runtime, but not now.

**Prerequisite(Environment)**
```
$ uname -r
5.0.0-29-generic

$ lsb_release -a
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.3 LTS
Release:	18.04
Codename:	bionic
```

**1. Download LXC source code**
```
$ git clone https://github.com/lxc/lxc.git
$ cd lxc/
```

**2. Install dependencies of build system**
```
$ sudo apt-get install automake
$ sudo apt-get install autotools-dev
$ sudo apt-get install libcap-dev
```
if occurs some errors, please install these modules

**3. Build LXC**
```
$ ./autogen.sh
$ ./configure CFLAGS='-pg -g'
$ make
$ sudo make install
$ sudo cp /usr/local/lib/liblxc* /usr/lib/x86_64-linux-gnu
```


**4. Download container image(rootfs) and Create a container based on the image**
Need root permission for controlling LXC system.
```
# If you need, please enter this.
$ sudo su
```

```
$ lxc-create -t download -n my-container
Setting up the GPG keyring
Downloading the image index

---
DIST	RELEASE	ARCH	VARIANT	BUILD
---
...
ubuntu	xenial	amd64	default	20190921_07:42
ubuntu	xenial	arm64	default	20190921_07:42
ubuntu	xenial	armhf	default	20190921_08:07
ubuntu	xenial	i386	default	20190921_07:42
ubuntu	xenial	ppc64el	default	20190921_07:42
ubuntu	xenial	s390x	default	20190921_07:42
---

Distribution:
ubuntu
Release:
xenial
Architecture:
amd64
```
Please enter the desired image information. 

**5. Check a list of containers**
```
$ lxc-ls
my-container
```
Using uftrace
```
$ uftrace lxc-ls
container1   my-container
# DURATION     TID     FUNCTION
            [ 11605] | main() {
            [ 11605] |   lxc_arguments_parse() {
   0.389 us [ 11605] |     build_shortopts();
   1.109 us [ 11605] |     getopt_long();
            [ 11605] |     lxc_get_global_config_item() {
 448.864 us [ 11605] |       /* linux:schedule */
            [ 11605] |       lxc_get_global_config_item() {
            [ 11605] |         lxc_global_config_value() {
            [ 11605] |           fopen_cloexec() {
  15.393 us [ 11605] |             /* linux:schedule */
 240.392 us [ 11605] |             /* linux:schedule */
 278.504 us [ 11605] |           } /* fopen_cloexec */
   1.160 us [ 11605] |           remove_trailing_slashes();
 289.914 us [ 11605] |         } /* lxc_global_config_value */
 292.817 us [ 11605] |       } /* lxc_get_global_config_item */
 778.150 us [ 11605] |     } /* lxc_get_global_config_item */
            [ 11605] |     lxc_arguments_lxcpath_add() {
   0.139 us [ 11605] |       realloc();
   0.564 us [ 11605] |     } /* lxc_arguments_lxcpath_add */
 781.691 us [ 11605] |   } /* lxc_arguments_parse */
            [ 11605] |   ls_get() {
            [ 11605] |     lxc_append_paths() {
.....
```

**5-1. Check a list of containers**
```
$ uftrace tui
  TOTAL TIME : FUNCTION
    2.913 ms : (1) lxc-ls
    2.913 ms : (1) main
   31.399 us :  ├─(1) lxc_arguments_parse
    0.446 us :  │  ├─(1) build_shortopts
             :  │  │
    4.062 us :  │  ├─(1) getopt_long
             :  │  │
   24.466 us :  │  ├─(1) lxc_get_global_config_item
   20.197 us :  │  │ (1) lxc_get_global_config_item
   16.241 us :  │  │ (1) lxc_global_config_value
    6.009 us :  │  │  ├─(1) fopen_cloexec
             :  │  │  │
    0.659 us :  │  │  └─(1) remove_trailing_slashes
             :  │  │
    0.666 us :  │  └─(1) lxc_arguments_lxcpath_add
    0.214 us :  │    (1) realloc
             :  │
    2.010 ms :  ├─(1) ls_get
    2.797 us :  │  ├─(1) lxc_append_paths
    2.331 us :  │  │ (1) lxc_append_paths
             :  │  │
    4.496 us :  │  ├─(1) dir_exists
    4.081 us :  │  │ (1) dir_exists
.....
```
