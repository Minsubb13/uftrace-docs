If we are using v0.15.2 release uftrace for your yocto BSP in a glibc-based environment, we can write bitbake file.

I have done BSP and tested on Digilent ZYBO 77-20 Board.
- https://digilent.com/reference/programmable-logic/zybo-z7/reference-manual
![image](https://github.com/namhyung/uftrace/assets/43713967/2c6d67f6-a27a-480a-ad96-c8762e0bdf55)

I writed `bb` on `project-spec/meta-user/recipes-apps/uftrace/uftrace_0.15.2.bb`
- https://github.com/paranlee/Zybo-Z7-OS/blob/20/Petalinux/master/project-spec/meta-user/recipes-apps/uftrace/uftrace_0.15.2.bb
```bb
SUMMARY = "Trace and analyze execution of a program written in C, C++, Rust and Python"
HOMEPAGE = "https://github.com/namhyung/uftrace"
BUGTRACKER = "https://github.com/namhyung/uftrace/issues"
SECTION = "devel"
LICENSE = "GPL-2.0-only"
LIC_FILES_CHKSUM = "file://COPYING;md5=b234ee4d69f5fce4486a80fdaf4a4263"

DEPENDS = "elfutils"

inherit autotools

SRCREV = "9d8657e90b918994d7d2bcf6dd2cc7354c35a1b4"
SRC_URI = "git://github.com/namhyung/${BPN};branch=master;protocol=https"
S = "${WORKDIR}/git"

def set_target_arch(d):
    import re
    arch = d.getVar('TARGET_ARCH')
    if re.match(r'i.86', arch, re.I):
        return 'i386'
    elif re.match('armeb', arch, re.I):
        return 'arm'
    else:
        return arch

EXTRA_UFTRACE_OECONF = "ARCH=${@set_target_arch(d)} \
                        with_elfutils=/use/libelf/from/sysroot"

do_configure() {
    ${S}/configure ${EXTRA_UFTRACE_OECONF}
}

FILES_SOLIBSDEV = ""
FILES:${PN} += "${libdir}/*.so"

COMPATIBLE_HOST = "(i.86|x86_64|aarch64|arm|riscv64)"

# uftrace supports armv6 and above
COMPATIBLE_HOST:armv4 = 'null'
COMPATIBLE_HOST:armv5 = 'null'
```

After performs the BSP on ZYBO Z7-20, we can test uftrace it works well.

NOTE: If someone using `musl` c library, We can reference https://github.com/openembedded/meta-openembedded/tree/master/meta-oe/recipes-devtools/uftrace

```
$ uname -a   
Linux Petalinux-2022 5.15.19-xilinx-v2022.1 #1 SMP PREEMPT Wed Jul 20 20:30:06 UTC 2022 armv7l armv7l armv7l GNU/Linux

$ uftrace --version
uftrace v0.15.2 ( arm dwarf perf sched )
```

We can test with `yavta`.
- https://git.ideasonboard.org/yavta.git/

```
$ uftrace record --force yavta -c1 -f YUYV -s "$width"x"$height" -F /dev/video0
Device /dev/video0 opened.
Device `vcap_mipi_csi2_rx_subsystem_0 o' on `platform:vcap_mipi_csi2_rx_subs' is a video output (without mplanes) device.
Video format set: YUYV (56595559) 1920x1080 field none, 1 planes: 
 * Stride 3840, buffer size 4147200
Video format: YUYV (56595559) 1920x1080 field none, 1 planes: 
 * Stride 3840, buffer size 4147200
8 buffers requested.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 0/0 mapped at address 0xb6366000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 1/0 mapped at address 0xb5f71000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 2/0 mapped at address 0xb5b7c000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 3/0 mapped at address 0xb5787000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 4/0 mapped at address 0xb5392000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 5/0 mapped at address 0xb4f9d000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 6/0 mapped at address 0xb4ba8000.
length: 1 offset: 3199264864 timestamp type/source: mono/EoF
Buffer 7/0 mapped at address 0xb47b3000.
0 (0) [-] none 0 0 B 2071.275698 2071.275824 0.501 fps ts mono/EoF
Captured 1 frames in 1.996645 seconds (0.500840 fps, 0.000000 B/s).
8 buffers released.
```

```
$ uftrace replay
# DURATION     TID     FUNCTION
   3.150 us [   820] | memset();
   7.512 us [   820] | getopt_long();
   3.552 us [   820] | strtol();
   2.406 us [   820] | getopt_long();
   2.730 us [   820] | strcasecmp();
   1.974 us [   820] | strcasecmp();
   1.872 us [   820] | strcasecmp();
   1.938 us [   820] | strcasecmp();
   1.896 us [   820] | strcasecmp();
   2.046 us [   820] | strcasecmp();
   1.950 us [   820] | strcasecmp();
   1.878 us [   820] | strcasecmp();
   1.908 us [   820] | strcasecmp();
   2.094 us [   820] | strcasecmp();
   2.064 us [   820] | strcasecmp();
   1.974 us [   820] | strcasecmp();
   1.962 us [   820] | strcasecmp();
   1.902 us [   820] | strcasecmp();
   1.902 us [   820] | strcasecmp();
   2.142 us [   820] | strcasecmp();
   2.382 us [   820] | getopt_long();
   2.718 us [   820] | strtol();
   2.250 us [   820] | strtol();
   2.346 us [   820] | getopt_long();
   2.220 us [   820] | getopt_long();
  56.724 us [   820] | open();
  78.876 us [   820] | __fprintf_chk();
   2.346 us [   820] | memset();
  26.640 us [   820] | ioctl();
...
```