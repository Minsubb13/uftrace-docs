**1. Download telegram**
```
$ git clone --recursive https://github.com/vysheng/tg.git
$ cd tg
```

**2. Configuration**
```
# install required packages
$ sudo apt install libevent-dev libreadline-dev libconfig-dev libjansson-dev

# do configuration
$ ./configure --disable-liblua
```

**3. Compilation**
```
$ make CFLAGS="-pg -g" CXXFLAGS="-pg -g" -j9
$ cd bin
```

**4. Check the generated binary**
```
$ nm telegram-cli | grep mcount
                 U mcount@@GLIBC_2.2.5
```

**5. Run `telegram-cli` with uftrace**
```
$ uftrace -a -D 3 ./telegram-cli --help
telegram-cli Usage
  --phone/-u                           specify username (would not be asked during authorization)
  --rsa-key/-k                         specify location of public key (possible multiple entries)
  --verbosity/-v                       increase verbosity (0-ERROR 1-WARNIN 2-NOTICE 3+-DEBUG-levels)
...
  --permanent-msg-ids                  use permanent msg ids
  --permanent-peer-ids                 use permanent peer ids
# DURATION     TID     FUNCTION
            [134448] | main(2, 0x7ffc15d4c4f8) {
   3.493 us [134448] |   signal(SIGSEGV, &termination_signal_handler) = 0x7f4d9c408b00;
   1.153 us [134448] |   signal(SIGABRT, &termination_signal_handler) = 0x7f4d9c408b00;
   1.004 us [134448] |   signal(SIGBUS, &termination_signal_handler) = 0;
   1.027 us [134448] |   signal(SIGQUIT, &termination_signal_handler) = 0;
   1.027 us [134448] |   signal(SIGFPE, &termination_signal_handler) = 0;
   1.170 us [134448] |   signal(SIGPIPE, 0x1) = 0;
   1.010 us [134448] |   signal(SIGTERM, &sig_term_handler) = 0;
   1.120 us [134448] |   signal(SIGINT, &sig_term_handler) = 0;
            [134448] |   args_parse(2, 0x7ffc15d4c4f8) {
  29.050 us [134448] |     tgl_state_alloc() = 0xeba5240;
 477.485 us [134448] |     getopt_long(2, 0x7ffc15d4c4f8, "u:hk:vNl:fEwWCRAdL:DU:G:qP:S:e:I6bc:p:") = 104;
            [134448] |     usage() {
            [134448] |       /* linux:task-exit */

uftrace stopped tracing with remaining functions
================================================
task: 134448
[2] usage
[1] args_parse
[0] main
```

**6. Login telegram with a phone number**
```
$ ./telegram-cli -q
Telegram-cli version 1.4.1, Copyright (C) 2013-2015 Vitaly Valtman
Telegram-cli comes with ABSOLUTELY NO WARRANTY; for details type `show_license'.
This is free software, and you are welcome to redistribute it
under certain conditions; type `show_license' for details.
Telegram-cli uses libtgl version 2.1.0
Telegram-cli includes software developed by the OpenSSL Project
for use in the OpenSSL Toolkit. (http://www.openssl.org/)
I: config dir=[/home/honggyu/.telegram-cli]
[/home/honggyu/.telegram-cli] created
[/home/honggyu/.telegram-cli/downloads] created
phone number: +8210********
code ('CALL' for phone code): *1*4*
User honggyu online (was online [2019/09/12 00:18:31])
> quit
halt
```

**7. Trace telegram displaying dialog list**
```
# record
$ uftrace record -a ./telegram-cli -e dialog_list
Telegram-cli version 1.4.1, Copyright (C) 2013-2015 Vitaly Valtman
Telegram-cli comes with ABSOLUTELY NO WARRANTY; for details type `show_license'.
...
User Telegram: 3 unread
> All done. Exit
halt

# replay
$ uftrace replay -t 10ms -T event_base_loop@depth=1 -T conn_try_read@depth=30
# DURATION     TID     FUNCTION
            [134883] | main(3, 0x7ffd6093e6f8) {
            [134883] |   inner_main() {
            [134883] |     loop() {
            [134883] |       net_loop() {
  88.575 ms [134883] |         event_base_loop();
  86.490 ms [134883] |         event_base_loop();
  86.835 ms [134883] |         event_base_loop();
  74.651 ms [134883] |         event_base_loop();
  74.667 ms [134883] |         event_base_loop();
  81.255 ms [134883] |         event_base_loop();
  92.920 ms [134883] |         event_base_loop();
 233.156 ms [134883] |         event_base_loop();
  79.405 ms [134883] |         event_base_loop();
            [134883] |         event_base_loop() {
            [134883] |           conn_try_read(7, 2, 0xf4fb840) {
            [134883] |             try_read(0xf4fb840) {
            [134883] |               try_rpc_read(0xf4fb840) {
            [134883] |                 rpc_execute(0xf4c2210, 0xf4fb840, 0xe37e576b, 23576) {
            [134883] |                   process_rpc_message(0xf4c2210, 0xf4fb840, 0x460ce80, 23576) {
            [134883] |                     rpc_execute_answer(0xf4c2210, 0xf4fb840, 0x5d79118bbcfd1001) {
            [134883] |                       work_rpc_result(0xf4c2210, 0xf4fb840, 0x5d79118bbcfd1001) {
            [134883] |                         tglq_query_result(0xf4c2210, 0x5d79118b318bb800) {
            [134883] |                           skip_type_any(&__compound_literal.16) {
            [134883] |                             skip_type_messages_dialogs(&__compound_literal.16) {
  10.039 ms [134883] |                               skip_constructor_messages_dialogs_slice(&__compound_literal.16) = 0;
  10.043 ms [134883] |                             } = 0; /* skip_type_messages_dialogs */
  10.046 ms [134883] |                           } = 0; /* skip_type_any */
            [134883] |                           fetch_ds_type_any(&__compound_literal.16) {
            [134883] |                             fetch_ds_type_messages_dialogs(&__compound_literal.16) {
  25.914 ms [134883] |                               fetch_ds_constructor_messages_dialogs_slice(&__compound_literal.16) = 0xf4fbd80;
  25.918 ms [134883] |                             } = 0xf4fbd80; /* fetch_ds_type_messages_dialogs */
  25.921 ms [134883] |                           } = 0xf4fbd80; /* fetch_ds_type_any */
  26.074 ms [134883] |                           get_dialogs_on_answer(0xf4c2210, 0xf4fc560, 0xf4fbd80) = 0;
            [134883] |                           free_ds_type_any(0xf4fbd80, &__compound_literal.16) {
            [134883] |                             free_ds_type_messages_dialogs(0xf4fbd80, &__compound_literal.16) {
  14.045 ms [134883] |                               free_ds_constructor_messages_dialogs_slice(0xf4fbd80, &__compound_literal.16);
  14.047 ms [134883] |                             } /* free_ds_type_messages_dialogs */
  14.049 ms [134883] |                           } /* free_ds_type_any */
  76.695 ms [134883] |                         } = 0; /* tglq_query_result */
  76.700 ms [134883] |                       } = 0; /* work_rpc_result */
  76.702 ms [134883] |                     } = 0; /* rpc_execute_answer */
  77.707 ms [134883] |                   } = 0; /* process_rpc_message */
  77.845 ms [134883] |                 } = 0; /* rpc_execute */
  77.853 ms [134883] |               } /* try_rpc_read */
  77.865 ms [134883] |             } /* try_read */
  77.867 ms [134883] |           } /* conn_try_read */
  77.945 ms [134883] |         } /* event_base_loop */
            [134883] |         do_halt(0) {
            [134883] |           exit(0) {
            [134883] |             /* linux:task-exit */

uftrace stopped tracing with remaining functions
================================================
task: 134883
[5] exit
[4] do_halt
[3] net_loop
[2] loop
[1] inner_main
[0] main
```
**8. TUI output**
```
$ uftrace tui -t 300us
  TOTAL TIME : FUNCTION
    1.012  s : (1) telegram-cli
    1.012  s : (1) main
  530.726 us :  ├▶(1) args_parse
             :  │
    1.011  s :  └─(1) inner_main
    1.011  s :    (1) loop
  610.787 us :     ├▶(1) set_interface_callbacks
             :     │
  545.207 us :     ├▶(1) tgl_login
             :     │
    1.009  s :     └─(1) net_loop
  994.845 ms :        ├─(14) event_base_loop
  907.891 ms :        │  ├─(12) linux:schedule
             :        │  │
    2.480 ms :        │  ├▶(2) conn_try_write
             :        │  │
   83.204 ms :        │  └─(6) conn_try_read
   83.190 ms :        │    (6) try_read
   82.990 ms :        │    (6) try_rpc_read
   82.944 ms :        │    (6) rpc_execute
  943.647 us :        │     ├▶(1) tgln_read_in
             :        │     │
   80.981 ms :        │     └─(4) process_rpc_message
   79.790 ms :        │        ├─(4) rpc_execute_answer
   79.781 ms :        │        │ (4) work_rpc_result
   79.765 ms :        │        │ (4) tglq_query_result
   27.157 ms :        │        │  ├─(3) fetch_ds_type_any
    1.223 ms :        │        │  │  ├▶(2) fetch_ds_type_config
             :        │        │  │  │
   25.918 ms :        │        │  │  └─(1) fetch_ds_type_messages_dialogs
   25.914 ms :        │        │  │    (1) fetch_ds_constructor_messages_dialogs_slice
   25.895 ms :        │        │  │    (4) fetch_ds_type_vector
   25.886 ms :        │        │  │    (4) fetch_ds_constructor_vector
    3.087 ms :        │        │  │    (8) fetch_ds_type_any
    3.082 ms :        │        │  │    (8) fetch_ds_type_message
    3.067 ms :        │        │  │    (8) fetch_ds_constructor_message
    1.369 ms :        │        │  │    (4) fetch_ds_type_message_media
    1.361 ms :        │        │  │    (4) fetch_ds_constructor_message_media_web_page
    1.345 ms :        │        │  │    (4) fetch_ds_type_web_page
    1.038 ms :        │        │  │    (3) fetch_ds_constructor_web_page
             :        │        │  │
   14.734 ms :        │        │  ├▶(3) free_ds_type_any
             :        │        │  │
  571.949 us :        │        │  ├▶(1) tgl_inflate
             :        │        │  │
   10.046 ms :        │        │  ├▶(1) skip_type_any
             :        │        │  │
   26.074 ms :        │        │  └▶(1) get_dialogs_on_answer
             :        │        │
  888.590 us :        │        └▶(1) tgl_pad_aes_decrypt
             :        │
   12.526 ms :        └▶(1) do_halt
```