This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download Libav**
```
$ git clone git://git.libav.org/libav.git
```

**2. Configuration and Build**
```
$ ./configure --disable-x86asm
$ make && make install
```

**3. Check the generated binary**
```
$ avprobe -v verbose -show_format -of json simple.flac
avprobe version v13_dev0-1649-ge5afa1b55, Copyright (c) 2007-2018 the Libav developers
  built on Oct  5 2019 12:27:36 with gcc 9 (Ubuntu 9.1.0-2ubuntu2~18.04)
  configuration: --disable-x86asm
  libavutil     56.  8. 0 / 56.  8. 0
  libavcodec    58. 12. 1 / 58. 12. 1
  libavformat   58.  2. 0 / 58.  2. 0
  libavdevice   57.  0. 2 / 57.  0. 2
  libavfilter    7.  1. 0 /  7.  1. 0
  libavresample  4.  0. 0 /  4.  0. 0
  libswscale     5.  0. 1 /  5.  0. 1
[flac @ 0x563478469a40] max_analyze_duration 5000000 reached
Input #0, flac, from 'simple.flac':
  Metadata:
    ALBUM           : Bee Moved
    TITLE           : Bee Moved
    album_artist    : Blue Monday FM
    MRAT            : 0
    ARTIST          : Blue Monday FM
  Duration: 00:00:39.87, bitrate: 3340 kb/s
    Stream #0:0: Audio: flac
      96000 Hz, stereo, s32
    Stream #0:1: Video: mjpeg
      yuvj420p, pc, bt470bg/unknown/unknown
      500x500 (0x0) [PAR 1:1 DAR 1:1]
      90k tbn
    Metadata:
      comment         : Cover (front)
      title           : image/jpeg
{   "format" : {
    "filename" : "simple.flac",
    "nb_streams" : 2,
    "format_name" : "flac",
    "format_long_name" : "raw FLAC",
    "start_time" : "N/A",
    "duration" : "39.876000",
    "size" : "16652777.000000",
    "bit_rate" : "3340912.000000",
    "tags" : {
      "ALBUM" : "Bee Moved",
      "TITLE" : "Bee Moved",
      "album_artist" : "Blue Monday FM",
      "MRAT" : "0",
      "ARTIST" : "Blue Monday FM"
    }
  }}

```

**4. Record Libav with uftrace**

uftrace can now record the target program `Libav`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a avprobe -v verbose -show_format -of json simple.flac
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 44161] | main(7, 0x7ffc27330f98) {
            [ 44161] |   av_malloc(4096) {
   1.424 us [ 44161] |     posix_memalign(0x7ffc27330ce0, 32, 4096) = 0;
   5.892 us [ 44161] |   } = 0x55c89b6b69a0; /* av_malloc */
            [ 44161] |   locate_option(7, 0x7ffc27330f98, &real_options, "loglevel") {
   0.530 us [ 44161] |     strchr("v", ':') = "NULL";
   0.193 us [ 44161] |     strlen("v") = 1;
   4.772 us [ 44161] |     strncmp("v", "L", 1) = 42;
   5.624 us [ 44161] |     strncmp("v", "h", 1) = 14;
   0.304 us [ 44161] |     strncmp("v", "?", 1) = 55;
   0.150 us [ 44161] |     strncmp("v", "help", 1) = 14;
   0.150 us [ 44161] |     strncmp("v", "-help", 1) = 73;
   3.767 us [ 44161] |     strncmp("v", "version", 1) = 0;
   0.198 us [ 44161] |     strlen("version") = 7;
   0.297 us [ 44161] |     strncmp("v", "formats", 1) = 16;
   0.142 us [ 44161] |     strncmp("v", "codecs", 1) = 19;
   0.140 us [ 44161] |     strncmp("v", "decoders", 1) = 18;
   0.121 us [ 44161] |     strncmp("v", "encoders", 1) = 17;
   0.130 us [ 44161] |     strncmp("v", "bsfs", 1) = 20;
   0.154 us [ 44161] |     strncmp("v", "protocols", 1) = 6;
.....
```


**6. uftrace tui output**

```
$ uftrace tui
  TOTAL TIME : FUNCTION
  448.928 ms : (1) avprobe
  448.928 ms : (1) main
    5.892 us :  ├─(1) av_malloc
    1.424 us :  │ (1) posix_memalign
             :  │
  208.232 us :  ├─(2) locate_option
    1.037 us :  │  ├─(4) strchr
             :  │  │
    1.552 us :  │  ├─(10) strlen
             :  │  │
   33.615 us :  │  ├─(80) strncmp
             :  │  │
    0.777 us :  │  └─(4) strcmp
             :  │
  119.624 us :  ├─(1) atrac3_init_static_data
   40.638 us :  │  ├─(1) ff_atrac_generate_tables
   18.726 us :  │  │ (64) pow
             :  │  │
   34.204 us :  │  └─(7) ff_init_vlc_sparse
    5.278 us :  │     ├─(7) av_malloc
    1.728 us :  │     │ (7) posix_memalign
             :  │     │
    1.151 us :  │     ├─(7) qsort
             :  │     │
   14.211 us :  │     ├─(7) build_table
...
```

**7. uftrace graph output**

```
$ uftrace graph
# Function Call Graph for 'avprobe' (session: bc688a0325196b62)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
  448.928 ms : (1) avprobe
  448.928 ms : (1) main
    5.892 us :  +-(1) av_malloc
    1.424 us :  | (1) posix_memalign
             :  |
  208.232 us :  +-(2) locate_option
    1.037 us :  |  +-(4) strchr
             :  |  |
    1.552 us :  |  +-(10) strlen
             :  |  |
   33.615 us :  |  +-(80) strncmp
             :  |  |
    0.777 us :  |  +-(4) strcmp
             :  |
  119.624 us :  +-(1) atrac3_init_static_data
   40.638 us :  |  +-(1) ff_atrac_generate_tables
   18.726 us :  |  | (64) pow
             :  |  |
   34.204 us :  |  +-(7) ff_init_vlc_sparse
...
```