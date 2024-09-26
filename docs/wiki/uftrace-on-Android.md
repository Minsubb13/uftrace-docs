# uftrace on Android (WIP)

**Currently uftrace is not running properly under Android.**  
Things that don't work -> Argument Tracing, Dynamic Tracing

This document provides a brief introduction to how uftrace can be built and run on Android.

## Introduction
Running uftrace on Android can be done in several ways (context).

  - chroot'ed(?) env. (eg. inside termux app)
    - install uftrace package on Termux
    - compile uftrace from source on Termux

  - system shell env. (ADB)
    - run uftrace binary under chroot'ed env
    - build uftace as NDK binary

## Testing Environment
  - AOSP 10 - Android Q

  - Google Pixel (aarch64)
    - Android: aosp_sailfish-eng 10 (android-10.0.0_r1)  
    - Kernel: 3.18.137 (fixed to 3.18.x)
  - Virtual Device [cuttlefish] (x86)
    - Android: aosp_cf_x86_phone (aosp_master@eef9850)
    - Kernel: 5.4.6 (stock kernel. custom kernel available)

## General preparation
### Get AOSP image

To compile AOSP, follow instruction under [AOSP Document].   
For sudo permission in adb shell, build AOSP with `userdebug` or `eng` variants.

If you're trying to test on cuttlefish Virtual Device, you don't have to compile it. Jump to [Setting up Cuttlefish](#with-cuttlefish-virtual-device).

### Compile kernel with ftrace (function)

The stock kernel included in Android doesn't support with ftrace function tracer.    
In order to use kernel tracing with uftrace, you should compile kernel with FUNCTION_TRACER option enabled.

 - CONFIG_FUNCTION_TRACER=y
 - CONFIG_DYNAMIC_FTRACE=y
 
Follow instruction for compiling [AOSP kernel]  
For more information: [AOSP Dynamic Ftrace]


## Set up environment
### With your phone

For every Android phone in the world, there are different procedures for installing AOSP (unlock phone) and there might be even impossible phones. So in this section  will only briefly cover the concepts.

1. Unlock your phone's bootloader : [Search Here][unlock bootloader]
2. Get or build AOSP image
3. Boot your device with fastboot : adb reboot fastboot
4. Flash image to your phone with fastboot : fastboot flash <partition> <image>

To find the proper way to prepare AOSP on your device, Google it.

### With cuttlefish Virtual Device

1. Build cuttlefish host tool (like emulator)
```sh
git clone https://github.com/google/android-cuttlefish
cd android-cuttlefish
sudo apt install build-essential devscripts debhelper
debuild -i -us -uc -b
sudo apt install cdbs config-package-dev
sudo apt-get install -f
sudo dpkg -i ../cuttlefish-common_*_amd64.deb
```
2. Get cuttlefish AOSP image from [Android CI]
    1. From `aosp-master` branch, click aosp_cf_x86_phone `userdebug` variant
    2. At Artifacts tab, download below files
        - `aosp_cf_x86_phone-img-xxxxxx.zip` 
        - `cvd-host_package.tar.gz`
3. Extract downloaded files
```sh
mkdir cf && cd cf
tar xvf /path/to/cvd-host_package.tar.gz
unzip /path/to/aosp_cf_x86_phone-img-xxxxxx.zip
```
4. Run cuttlefish virtual machine
```sh
HOME=$PWD ./bin/launch_cvd
# To boot with custom compiled kernel, use -kernel-path option
# HOME=$PWD ./bin/launch_cvd -kernel_path /path/to/kernel/../x86/boot/bzImage
# If the graphic is slow, try below option
# HOME=$PWD ./bin/launch_cvd -gpu_mode=drm_virgl
```
5. Connect cuttlefish with VNC   
To view the screen, use VNC client ([Tiger VNC]) to access localhost 6444 port. 
```sh
java -jar tightvnc-jviewer.jar -ScalingFactor=50 -Tunneling=no -host=localhost -port=6444
```

For more information: [cuttlefish git]

## Install Uftrace on Android
### Install Termux App

1. Download Termux APK: [F-droid Store]
2. Install APK with ADB
```sh
adb install com.termux*.apk
```
3. Open termux app

### Install uftrace termux package

There is already [Uftrace Package] in the Termux package repo.

```sh
pkg update
pkg install uftrace
uftrace --version
# uftrace v0.9.3 ( python tui perf sched dynamic )
```

### Compile Termux App (Not Working, WIP)

For using latest feature of uftrace, compile inside termux.   
Since termux doesn't have `gcc`, use `clang` instead. (gcc available with unofficial repo).

To meet the dependency requirements, install below packages.
- `pkg-config`
- `libelf`
- `python`
- `capstone`
- `libluajit`

```sh
pkg update
pkg install git make clang
pkg install pkg-config libelf python capstone libluajit
./configure --prefix=$PREFIX
make -j 8
# WILL FAIL! Source code and LDFLAGS modification needed. 
# libandroid-spawn, argp, shmem etc..
```

Since there is no `libdw` package support from termux, separate compilation is required.   
Unfortunately, `elfutils` doesn't support compiling with clang.  (It has to be done with some hacky way)

Refer to the following link to find out how to do it: [libelf termux building]

Thanks to contributor @gonapps, there's a demo for compiling and running uftrace on Android.  
(The demo below is different from the environment in this document)

[![asciicast](https://asciinema.org/a/275617.svg)](https://asciinema.org/a/275617)

For custom compile, refer to Termux package build script:    
https://github.com/termux/termux-packages/tree/master/packages/uftrace

## Running Uftrace on Android
### Inside chroot'ed env (termux)

From termux app, run uftrace.

```sh
uftrace --version
# uftrace v0.9.3 ( python tui perf sched dynamic )
```

### Inside system shell (ADB)

From the Host side, elevate to root permission and execute uftrace under termux directory.  
(`su` is allowed only on AOSP `phonedebug`, `eng` variant.)

```sh
# From Host side, type:
adb shell
# Connect into android system shell
# vsoc_x86:/ $ 
whoami                      # shell
su
whoami                      # root
export PREFIX=/data/data/com.termux/files/usr
$PREFIX/bin/uftrace -L $PREFIX/lib --version
# uftrace v0.9.3 ( python tui perf sched dynamic )
```

### Tracing sample program with uftrace
Since termux environment doesn't have symbols for `mcount`, compile with `-pg` option will fail.   
When compiling sources inside termux, **compile with `-lmcount` option**.

```sh
# clang is binded to gcc
gcc -pg $PREFIX/uftrace/tests/s-abc.c
# ============= Result Truncated =============
# $PREFIX/bin/aarch64-linux-android-ld: $PREFIX/tmp/s-abc-833b6b.o: in function `c':
# s-abc.c:(.text+0xcc): undefined reference to `_mcount'
# clang-9: error: linker command failed with exit code 1 (use -v to see invocation)

gcc -pg -lmcount $PREFIX/uftrace/tests/s-abc.c
uftrace record a.out
uftrace replay
## DURATION    TID     FUNCTION
#            [ 2163] | main() {
#            [ 2163] |   a() {
#            [ 2163] |     b() {
#   0.880 us [ 2163] |       c();
#   3.475 us [ 2163] |     } /* b */
#   4.448 us [ 2163] |   } /* a */
#   5.631 us [ 2163] | } /* main */
```


### Kernel Tracing with uftrace
**Custom kernel with ftrace - function graph support is required!**   
(If no symbol name is shown, set `kptr_restrict` value to 0.)

Termux environment doesn't support for root permission natively.  
So in here, we'll running uftrace from system shell (ADB).

If you can get elevated permission inside (e.g. rooted phone) just try with su instead of ADB.

```sh
# 0. Prepare environment with ftrace - function graph supported kernel
# In here, I'll show how to boot custom kernel with Phone (Pixel 1, aarch64, sailfish, 3.18.xx)
# For booting custom kernel with cuttlefish, look [4. Run cuttlefish virtual machine] section.
adb reboot fastboot
fastboot boot Image.gz-dtb             # Your own custom kernel image.

# 1. Before start tracing, compile s-mmap.c inside termux
gcc -pg -lmcount $PREFIX/uftrace/tests/s-mmap.c -o t-mmap

# 2. From Host side, type and connect into android system shell
adb shell                              # sailfish:/ $ 
su
whoami                                 # root

# 3. Check current kernel supports ftrace - function graph.
cat /sys/kernel/debug/tracing/available_tracers
# blk function_graph function nop

# (optional) check kernel pointer restrict is set to 0.
cat /proc/sys/kernel/kptr_restrict
echo 0 > /proc/sys/kernel/kptr_restrict

# 4. Start Kernel trace inside Android
export PREFIX=/data/data/com.termux/files/usr
cd $PREFIX/../home/uftrace/tests
$PREFIX/bin/uftrace -L $PREFIX/lib record -K 10 t-mmap
# DURATION     TID     FUNCTION
#            [  3635] | main() {
#            [  3635] |   foo() {
#            [  3635] |     sys_openat() {
#            [  3635] |       handle_IPI() {
#            [  3635] |         irq_enter() {
#   1.719 us [  3635] |           rcu_irq_enter();
#   1.355 us [  3635] |           irqtime_account_irq();
...

```



### DalvikVM Tracing with uftrace in host

There is currently no way to tracing the Android subsystem via uftrace. Instead, it is possible to tracing how Dalvikvm, one of Android's subsystems, works. 

Try the following steps to trace dalvikvm:

1. Get the Android Source Code 
2. Run the `lunch` and choose the host environment to compile.
```
$ lunch

You're building on Linux

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_mips-eng
     4. aosp_mips64-eng
     5. aosp_x86-eng
     6. aosp_x86_64-eng
     7. full_fugu-userdebug
     8. aosp_fugu-userdebug
     ...
```
3. Run the `m` and wait until done compiling.
4. Move `init.environ.rc` in the compiled output. It is located in the following path : `out/target/product/generic_x86_64/root/init.environ.rc`. Find the path that matches your environment.
5. modify `init.environ.rc` like followed:

[ Original ]
```
# set up the global environment
on init
    export ANDROID_BOOTLOGO 1
    export ANDROID_ROOT /system
    export ANDROID_ASSETS /system/app
    export ANDROID_DATA /data
    export ANDROID_STORAGE /storage
    export EXTERNAL_STORAGE /sdcard
    export ASEC_MOUNTPOINT /mnt/asec
    export BOOTCLASSPATH /system/framework/core-oj.jar:/system/framework/core-libart.jar:/system/framework/conscrypt.jar:/system/framework/okhttp.jar:/system/framework/legacy-test.jar:/system/framework/bouncycastle.jar:/system/framework/ext.jar:/system/framework/framework.jar:/system/framework/telephony-common.jar:/system/framework/voip-common.jar:/system/framework/ims-common.jar:/system/framework/apache-xml.jar:/system/framework/org.apache.http.legacy.boot.jar:/system/framework/android.hidl.base-V1.0-java.jar:/system/framework/android.hidl.manager-V1.0-java.jar
    export SYSTEMSERVERCLASSPATH /system/framework/services.jar:/system/framework/ethernet-service.jar:/system/framework/wifi-service.jar
```

[ Modified ]
```
# set up the global environment
export ANDROID_BOOTLOGO=1
export ANDROID_ROOT=/system
export ANDROID_ASSETS=/system/app
export ANDROID_DATA=/data
export BOOTCLASSPATH=/system/framework/core-oj.jar:/system/framework/core-libart.jar:/system/framework/conscrypt.jar:/system/framework/okhttp.jar:/system/framework/legacy-test.jar:/system/framework/bouncycastle.jar:/system/framework/ext.jar:/system/framework/framework.jar:/system/framework/telephony-common.jar:/s
ystem/framework/voip-common.jar:/system/framework/ims-common.jar:/system/framework/apache-xml.jar:/system/framework/org.apache.http.legacy.boot.jar:/system/framework/android.hidl.base-V1.0-java.jar:/system/framework/android.hidl.manager-V1.0-java.jar
```

6. apply it to your environment
```
$ source init.environ.rc
```

7. run dalvikvm to see followed message
```
$ dalvikvm
dalvikvm F 02-18 20:47:17 11445 11445 utils.cc:761] Failed to find ANDROID_ROOT directory /system
Runtime aborting...
(Runtime does not yet exist!)
```

8. copy `/system` from compiled output
```
$ sudo cp -R out/target/product/generic_x86_64/system /
```
now you can see different error message from dalvikvm.
```
$ dalvikvm
dalvikvm I 02-18 20:48:45 11477 11477 image_space.cc:270] RelocateImage: /system/bin/patchoat --input-image-location=/system/framework/boot.art --output-image-file= --instruction-set=x86_64 --base-offset-delta=-335872
dalvikvm F 02-18 20:48:45 11477 11477 utils.cc:761] Failed to find ANDROID_DATA directory /data
Runtime aborting...
```

9. make directory `/data`

10. run uftrace with dalvikvm
```
$ uftrace record -P.@libart.so dalvikvm -cp classes.zip HellWorld
```

finished it. just taste results!

```
$ uftrace info
# system information
# ==================
# program version     : v0.8.2-1319-g52f6 ( dwarf python luajit tui perf sched dynamic )
# recorded on         : Tue Feb 18 00:19:55 2020
# cmdline             : uftrace record -P.@libart.so dalvikvm -cp classes.zip HellWorld
# cpu info            : AMD Ryzen 5 3600 6-Core Processor
# number of cpus      : 12 / 12 (online / possible)
# memory info         : 0.5 / 31.3 GB (free / total)
# system load         : 0.10 / 0.11 / 0.09 (1 / 5 / 15 min)
# kernel version      : Linux 5.0.0-23-generic
# hostname            : ubuntu
# distro              : "Ubuntu 18.04.3 LTS"
#
# process information
# ===================
# number of tasks     : 6
# task list           : 99798(dalvikvm64), 99802(dalvikvm64), 99803(dalvikvm64), 99804(dalvikvm64), 99805(dalvikvm64), 99806(dalvikvm64)
# exe image           : /home/m/AOSP8/out/host/linux-x86/bin/dalvikvm64
# pattern             : regex
# exit status         : exited with code: 0
# elapsed time        : 0.558800594 sec
# cpu time            : 0.166 / 0.392 sec (sys / user)
# context switch      : 35 / 1 (voluntary / involuntary)
# max rss             : 35268 KB
# page fault          : 61 / 40106 (major / minor)
# disk iops           : 1256 / 8 (read / write)
```



Example demo recorded file: [mmap_f-AOSP-kernel-aarch64.data.tar.xz](https://cloud.d-tl.com/index.php/s/b7mSidiZdaLJ5mC)

   [cuttlefish]: <https://drive.google.com/file/d/1DuSFffKSVwbX39m5wGGMnBTObqUOweV3/view/>
   [AOSP Document]: <https://source.android.com/setup/build/downloading>
   [AOSP Kernel]: <https://source.android.com/setup/build/building-kernels>
   [AOSP Dynamic Ftrace]: <https://source.android.com/devices/tech/debug/ftrace#dftrace>
   [unlock bootloader]: <https://www.getdroidtips.com/category/unlock-bootloader/>
   [Android CI]: <https://ci.android.com/builds/branches/aosp-master/grid?>
   [Tiger VNC]: <https://www.tightvnc.com/download.php>
   [cuttlefish git]: <https://android.googlesource.com/device/google/cuttlefish/>
   [F-droid Store]: <https://f-droid.org/en/packages/com.termux/>
   [Uftrace Package]: <https://github.com/termux/termux-packages/tree/master/packages/uftrace>
   [libelf termux building]: <https://github.com/termux/termux-packages/blob/master/packages/libelf/build.sh#L12>