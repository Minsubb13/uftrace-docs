This document is written by [GwanYeong Kim](https://github.com/gy741).

**1. Download Wireshark**
```
$ git clone https://github.com/wireshark/wireshark.git
```

**2. Configuration and Build**
```
$ sudo apt install build-essential pkg-config ninja-build bison flex qt5-default qttools5-dev-tools qtcreator ninja-build libpcap-dev cmake libglib2.0-dev libgcrypt20-dev qttools5-dev qtmultimedia5-dev libqt5svg5-dev
$ mkdir build && cd build && cmake .. && make && make install
```

**3. Check the generated binary**
```
$ tshark -r test/captures/http.pcap
Usage: invalid enum spec: e:5views_c_5f
Running as user "root" and group "root". This could be dangerous.
    1   0.000000     10.0.0.5 → 207.46.134.94 HTTP 207 HEAD /v4/iuident.cab?0307011208 HTTP/1.1
```

**4. Record Wireshark with uftrace**

uftrace can now record the target program `Wireshark`, which is not compiled with `-pg` or `-finstrument-functions`, but it can be possible with [full-dynamic tracing](https://uftrace.github.io/slide/#full-dynamic-tracing) feature. Please note that the following command uses `-P.` option.

```
$ uftrace record -P. -a tshark -r test/captures/http.pcap
Usage: invalid enum spec: e:5views_c_5f
Running as user "root" and group "root". This could be dangerous.
    1   0.000000     10.0.0.5 → 207.46.134.94 HTTP 207 HEAD /v4/iuident.cab?0307011208 HTTP/1.1
```

**5. uftrace replay output**
```
$ uftrace replay
# DURATION     TID     FUNCTION
 267.209 us [  4849] | setlocale(LC_ALL, "") = "ko_KR.UTF-8";
   9.825 us [  4849] | init_process_policies();
   7.535 us [  4849] | relinquish_special_privs_perm();
   0.795 us [  4849] | started_with_special_privs();
 558.294 us [  4849] | get_cur_username();
  83.708 us [  4849] | get_cur_groupname();
  26.046 us [  4849] | fprintf(&_IO_2_1_stderr_, "Running as user "%s" and group "%s".") = 40;
   3.335 us [  4849] | running_with_special_privs();
   5.686 us [  4849] | fwrite(&capture.catch_spec, 25, 1, &_IO_2_1_stderr_) = 1;
   5.332 us [  4849] | fputc('\n', &_IO_2_1_stderr_) = 10;
  30.860 us [  4849] | init_progfile_dir();
  12.226 us [  4849] | funnel_set_funnel_ops();
            [  4849] | ws_init_version_info("TShark (Wireshark)", &get_tshark_compiled_version_info, &epan_get_compiled_version_info, &get_tshark_runtime_version_info) {
   8.129 us [  4849] |   g_strdup_printf();
            [  4849] |   get_compiled_version_info(&get_tshark_compiled_version_info, &epan_get_compiled_version_info) {
  10.486 us [  4849] |     g_string_new();
   1.428 us [  4849] |     g_string_append();
            [  4849] |     get_compiled_caplibs_version(0x56096849b600) {
   0.693 us [  4849] |       g_string_append();
   0.293 us [  4849] |       g_string_append();
   0.130 us [  4849] |       g_string_append();
   0.184 us [  4849] |       g_string_append();
   0.484 us [  4849] |       g_string_append();
.....
```


**6. uftrace tui output**

```
$ uftrace tui
 TOTAL TIME : FUNCTION
    1.988  s : (1) tshark
  267.209 us :  ├─(1) setlocale
             :  │
    9.825 us :  ├─(1) init_process_policies
             :  │
    7.535 us :  ├─(1) relinquish_special_privs_perm
             :  │
    0.795 us :  ├─(1) started_with_special_privs
             :  │
  558.294 us :  ├─(1) get_cur_username
             :  │
   83.708 us :  ├─(1) get_cur_groupname
             :  │
   26.046 us :  ├─(1) fprintf
             :  │
    3.335 us :  ├─(1) running_with_special_privs
             :  │
    5.686 us :  ├─(1) fwrite
             :  │
    5.332 us :  ├─(1) fputc
             :  │
   30.860 us :  ├─(1) init_progfile_dir
             :  │
   12.226 us :  ├─(1) funnel_set_funnel_ops
             :  │
  204.836 us :  ├─(1) ws_init_version_info
    8.129 us :  │  ├─(1) g_strdup_printf
             :  │  │
   43.412 us :  │  ├─(1) get_compiled_version_info
   10.486 us :  │  │  ├─(1) g_string_new
             :  │  │  │
    2.069 us :  │  │  ├─(5) g_string_append
             :  │  │  │
    3.817 us :  │  │  ├─(1) get_compiled_caplibs_version
    1.784 us :  │  │  │ (5) g_string_append
             :  │  │  │
    3.056 us :  │  │  ├─(2) g_string_append_printf
...
```

**7. uftrace graph output**

```
$ uftrace graph
# Function Call Graph for 'randpktdump' (session: 6edae12e23cd0a83)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
  459.119 us : (1) randpktdump
    2.039 us :  +-(1) g_malloc0_n
             :  |
    9.537 us :  +-(1) init_process_policies
             :  |
   33.119 us :  +-(1) init_progfile_dir
             :  |
   14.966 us :  +-(1) data_file_url
             :  |
  265.527 us :  +-(1) extcap_base_set_util_info
    1.152 us :  |  +-(1) g_path_get_basename
             :  |  |
    1.552 us :  |  +-(1) g_strdup_printf
             :  |  |
    0.471 us :  |  +-(1) g_strdup
             :  |
    1.096 us :  +-(3) g_free
             :  |
    8.611 us :  +-(1) extcap_base_register_interface
    0.350 us :  |  +-(1) g_malloc0_n
             :  |  |
    0.720 us :  |  +-(4) g_strdup
             :  |  |
    5.054 us :  |  +-(1) g_list_append
             :  |
    2.216 us :  +-(1) g_strdup_printf
...
```
