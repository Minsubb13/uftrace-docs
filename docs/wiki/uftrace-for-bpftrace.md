**1. install bpftrace dependency**
```bash
# On ubuntu 19.04 & newer
sudo apt-get install libbpfcc-dev

# if not, compile bcc first
# https://github.com/iovisor/bcc/blob/master/INSTALL.md#install-and-compile-bcc
# For other distros, or detailed info can be found at:
# https://github.com/iovisor/bpftrace/blob/master/INSTALL.md#building-bpftrace 

# install dependency
sudo apt-get update
sudo apt-get install -y bison cmake flex g++ git libelf-dev zlib1g-dev libfl-dev systemtap-sdt-dev
sudo apt-get install -y llvm-7-dev llvm-7-runtime libclang-7-dev clang-7
```

**2. Clone bpftrace source**
```bash
git clone https://github.com/iovisor/bpftrace
```

**3. Compilation**
```bash
mkdir bpftrace/build
cd bpftrace/build
cmake -DCMAKE_CXX_FLAGS=-pg -DCMAKE_BUILD_TYPE=Release ..
make -j8
make install
```
**Option `-DCMAKE_CXX_FLAGS=-pg` must be added!**

**4. Check the generated binary**
```
$ nm src/bpftrace | grep mcount
                 U mcount@@GLIBC_2.2.5
```

**5. Run `bpftrace` with uftrace**
```
$ sudo uftrace record --no-libcall -K 10 bpftrace --include linux/blk_types.h -e 'kprobe:submit_bio { printf("[%s]BIO struct, cnt: %p, %u\n", comm, arg1, ((bio *)arg1)->bi_vcnt); }'
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 30336
[vim]BIO struct, cnt: 0xffff9640b246c950, 47808
...
```

**6. Check traced data with filters**
```
$ uftrace tui -t 20us -N __do_page_fault@k
   43.806 ms :        ├─(1) bpftrace::BPFtrace::attach_probe
   43.802 ms :        │ (1) bpftrace::AttachedProbe::AttachedProbe
    1.149 ms :        │  ├─(1) bpftrace::AttachedProbe::load_prog
   55.358 us :        │  │  ├─(1) __x64_sys_openat
   54.943 us :        │  │  │ (1) do_sys_open
   46.538 us :        │  │  │ (1) do_filp_open
   45.811 us :        │  │  │ (1) path_openat
             :        │  │  │
  935.903 us :        │  │  ├─(1) __x64_sys_bpf
  932.678 us :        │  │  │ (1) bpf_prog_load
   85.879 us :        │  │  │  ├─(1) bpf_prog_alloc
   81.422 us :        │  │  │  │ (1) __vmalloc
   80.750 us :        │  │  │  │ (1) __vmalloc_node_range
   69.368 us :        │  │  │  │ (1) __get_vm_area_node
   62.103 us :        │  │  │  │ (1) alloc_vmap_area
             :        │  │  │  │
  499.094 us :        │  │  │  ├─(1) bpf_check
   22.980 us :        │  │  │  │  ├─(1) vzalloc
   22.613 us :        │  │  │  │  │ (1) __vmalloc_node_range
             :        │  │  │  │  │
   39.709 us :        │  │  │  │  ├─(1) bpf_prog_calc_tag
   26.569 us :        │  │  │  │  │ (1) vmalloc
   25.963 us :        │  │  │  │  │ (1) __vmalloc_node_range
             :        │  │  │  │  │
  313.240 us :        │  │  │  │  └─(1) do_check
             :        │  │  │  │
  304.040 us :        │  │  │  ├─(1) bpf_prog_select_runtime
  208.101 us :        │  │  │  │  ├─(1) bpf_int_jit_compile
   36.605 us :        │  │  │  │  │  ├─(1) bpf_jit_binary_alloc
   35.174 us :        │  │  │  │  │  │ (1) module_alloc
   32.433 us :        │  │  │  │  │  │ (1) __vmalloc_node_range
   24.674 us :        │  │  │  │  │  │ (1) __get_vm_area_node
   20.615 us :        │  │  │  │  │  │ (1) alloc_vmap_area
... Truncated
```

Upper snippet traced result shows execution flow of `BPFtrace::attach_probe`.