**1. Download ffmpeg**
```
$ git clone --depth=1 https://github.com/FFmpeg/FFmpeg.git
$ cd FFmpeg
```

**2. Configure and Build**
```
$ ./configure --disable-x86asm
$ make -j8
```

**3. Get Samples**

Some media files can be downloaded in `fate-suite` directory with the following command. [fate](https://ffmpeg.org/fate.html) document explains the details.
```
$ make fate-rsync SAMPLES=fate-suite/
```

**4. Record ffmpeg with uftrace**

uftrace can now record the target program `ffmpeg_g`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.
```
$ uftrace record -P. -a ./ffmpeg_g -i ./fate-suite/qpeg/Clock.avi out.mp4
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [  8994] | main(4, 0x7ffcc0f90318) {
   2.689 us [  8994] |   setvbuf();
            [  8994] |   locate_option(4, 0x7ffcc0f90318, &options, "loglevel") {
   1.560 us [  8994] |     strchr("i", ':') = "NULL";
   0.773 us [  8994] |     strlen("i") = 1;
   9.294 us [  8994] |     strncmp("i", "L", 1) = 29;
   0.660 us [  8994] |     strncmp("i", "h", 1) = 1;
   0.513 us [  8994] |     strncmp("i", "?", 1) = 42;
   0.450 us [  8994] |     strncmp("i", "help", 1) = 1;
...
            [  8994] |   transcode() {
            [  8994] |     transcode_init() {
            [  8994] |       av_opt_set_int(0x557eba704e00, "refcounted_frames", 1, 0) {
            [  8994] |         av_opt_find2(0x557eba704e00, "refcounted_frames", "NULL", 0, 0, 0x7ffcc0f8f450) {
   0.726 us [  8994] |           strcmp("b", "refcounted_frames") = -16;
   0.500 us [  8994] |           strcmp("ab", "refcounted_frames") = -17;
...
            [  8994] |       av_dict_set(0x557eba7074a8, "threads", "auto", 0) {
   0.333 us [  8994] |         strlen("threads") = 7;
   0.333 us [  8994] |         realloc(0, 8) = 0x557eba7148a0;
   0.330 us [  8994] |         memcpy(0x557eba7148a0, &sample_fmts.8033, 8);
   0.330 us [  8994] |         strlen("auto") = 4;
   0.356 us [  8994] |         realloc(0, 5) = 0x557eba714de0;
   0.343 us [  8994] |         memcpy(0x557eba714de0, &on2avc_swb_start_long, 5);
   0.747 us [  8994] |         realloc(0x557eba714e20, 32) = 0x557eba711490;
   7.152 us [  8994] |       } = 0; /* av_dict_set */
            [  8994] |       hw_device_setup_for_decode(0x557eba707400) {
   1.607 us [  8994] |         avcodec_get_hw_config(&ff_qpeg_decoder, 0) = 0;
   4.669 us [  8994] |       } = 0; /* hw_device_setup_for_decode */
            [  8994] |       avcodec_open2(0x557eba704e00, &ff_qpeg_decoder, 0x557eba7074a8) {
   0.487 us [  8994] |         avcodec_is_open(0x557eba704e00) = 0;
            [  8994] |         av_dict_copy(0x7ffcc0f8f228, 0x557eba7b6e00, 0) {
            [  8994] |           av_dict_set(0x7ffcc0f8f228, "sub_text_format", "ass", 0) {
...
```

**6. uftrace tui output**
```
$ uftrace tui -t 1ms
```
The following output can be seen if `l` key is kept pressing after moving the cursor to `main` function.
```
  TOTAL TIME : FUNCTION
   34.192  s : (1) ffmpeg_g
    1.062  s :  ├─(1) main
   41.113 ms :  │  ├▶(1) ffmpeg_parse_options
             :  │  │
  952.731 ms :  │  ├─(1) transcode
    3.406 ms :  │  │  ├▶(1) transcode_init
             :  │  │  │
  324.810 ms :  │  │  ├▶(101) process_input_packet
             :  │  │  │
  525.401 ms :  │  │  ├─(179) reap_filters
   10.683 ms :  │  │  │  ├▶(2) init_output_stream.constprop.23
             :  │  │  │  │
  121.236 ms :  │  │  │  ├▶(76) avcodec_send_frame
             :  │  │  │  │
  366.530 ms :  │  │  │  └─(100) do_video_out
  345.805 ms :  │  │  │    (100) avcodec_send_frame
  345.401 ms :  │  │  │    (100) do_encode
  344.777 ms :  │  │  │    (100) avcodec_encode_video2
  341.777 ms :  │  │  │    (100) ff_mpv_encode_picture
  164.214 ms :  │  │  │    (111) thread_execute
  163.957 ms :  │  │  │    (111) avpriv_slicethread_execute
    9.515 ms :  │  │  │     ├▶(7) pthread_cond_wait
             :  │  │  │     │
   44.198 ms :  │  │  │     └─(32) worker_func
   42.089 ms :  │  │  │       (30) encode_thread
             :  │  │  │
    2.548 ms :  │  │  ├▶(1) avcodec_send_frame
             :  │  │  │
    3.933 ms :  │  │  └▶(1) av_write_trailer
             :  │  │
    7.686 ms :  │  ├▶(1) ffmpeg_cleanup
             :  │  │
   58.668 ms :  │  └─(1) exit
             :  │
   33.130  s :  └─(36) thread_worker
   31.281  s :     ├─(1315) pthread_cond_wait
   31.243  s :     │ (1308) linux:schedule
             :     │
  311.953 ms :     └─(243) worker_func
  281.761 ms :        ├─(225) encode_thread
             :        │
   14.225 ms :        └─(6) ff_estimate_p_frame_motion
    7.129 ms :           ├─(3) ff_epzs_motion_search
             :           │
    2.520 ms :           ├─(1) linux:schedule
             :           │
    2.264 ms :           └─(1) sad_hpel_motion_search
```