This document is written by [Hyuck Kang](https://github.com/kang-hyuck).

![libjpeg-turbo](https://github.com/namhyung/uftrace/assets/61424701/f44b8f24-72d0-4094-bde1-2fd627ec9591)

**libjpeg-turbo** is a JPEG image codec that uses SIMD instructions (MMX, SSE2, AVX2, Neon, AltiVec) to accelerate baseline JPEG compression and decompression on x86, x86-64, Arm, and PowerPC systems, as well as progressive JPEG compression on x86, x86-64, and Arm systems. 


#### 1. Download source code

```
$ git clone https://github.com/libjpeg-turbo/libjpeg-turbo.git
$ cd libjpeg-turbo
```

#### 2. Build

```
$ mkdir out
$ cd out
$ export CFLAGS=-pg
$ cmake .. -G"Unix Makefiles" -DCMAKE_BUILD_TYPE=Release
$ make -j32
```

#### 3. Input images
you need an ppm image input file to be converted to jpg out file. If you have png image, you can create ppm image with ffmpeg.  
  
(optional) If you need to install ffmpeg, use the following command.
```
$ sudo apt-get install ffmpeg
```

Here is sample images, just right-click and save the image.  
![fresia](https://github.com/namhyung/uftrace/assets/61424701/43bebbb8-9c8f-4433-930d-a8fa080e8d9e)


Convert sample images to ppm files  
```
$ ffmpeg -i fresia.png -vf scale=1280x1280 input.ppm
```


#### 4. Record

```
$ uftrace record --no-libcall ./tjexample input.ppm output.jpg -fastdct
```

#### 5. Replay

You can see the JPEG algorithm process with uftrace :)

```
$ uftrace replay

# DURATION     TID     FUNCTION
            [397196] | main() {
   0.252 us [397196] |   tj3GetScalingFactors();
            [397196] |   tj3Init() {
            [397196] |     _tjInitCompress() {
   0.335 us [397196] |       jpeg_std_error();
            [397196] |       jpeg_CreateCompress() {
            [397196] |         jinit_memory_mgr() {
   0.078 us [397196] |           jpeg_mem_init();
   1.681 us [397196] |           jpeg_get_small();
   9.661 us [397196] |         } /* jinit_memory_mgr */
            [397196] |         alloc_small() {
   0.221 us [397196] |           jpeg_get_small();
   0.686 us [397196] |         } /* alloc_small */
  17.722 us [397196] |       } /* jpeg_CreateCompress */
            [397196] |       jpeg_mem_dest_tj() {
   0.105 us [397196] |         alloc_small();
   0.614 us [397196] |       } /* jpeg_mem_dest_tj */
  37.014 us [397196] |     } /* _tjInitCompress */
  44.502 us [397196] |   } /* tj3Init */
            [397196] |   tj3LoadImage8() {
            [397196] |     tj3Init() {
            [397196] |       _tjInitCompress() {
   0.087 us [397196] |         jpeg_std_error();
            [397196] |         jpeg_CreateCompress() {
            [397196] |           jinit_memory_mgr() {
   0.059 us [397196] |             jpeg_mem_init();
   0.307 us [397196] |             jpeg_get_small();
   1.126 us [397196] |           } /* jinit_memory_mgr */
            [397196] |           alloc_small() {
   0.114 us [397196] |             jpeg_get_small();
   0.408 us [397196] |           } /* alloc_small */
   2.023 us [397196] |         } /* jpeg_CreateCompress */
            [397196] |         jpeg_mem_dest_tj() {
   0.071 us [397196] |           alloc_small();
   0.351 us [397196] |         } /* jpeg_mem_dest_tj */
   3.417 us [397196] |       } /* _tjInitCompress */
   7.580 us [397196] |     } /* tj3Init */
            [397196] |     jinit_read_ppm() {
            [397196] |       alloc_small() {
   7.392 us [397196] |         jpeg_get_small();
   7.939 us [397196] |       } /* alloc_small */
   8.417 us [397196] |     } /* jinit_read_ppm */
            [397196] |     start_input_ppm() {
   0.141 us [397196] |       read_pbm_integer();
   0.096 us [397196] |       read_pbm_integer();
   0.080 us [397196] |       read_pbm_integer();
            [397196] |       my_emit_message() {
   0.070 us [397196] |         emit_message();
   0.555 us [397196] |       } /* my_emit_message */
   0.064 us [397196] |       alloc_small();
   2.583 us [397196] |     } /* start_input_ppm */
   0.174 us [397196] |     realize_virt_arrays();
   5.312 us [397196] |     get_raw_row();
   4.426 us [397196] |     get_raw_row();
   3.375 us [397196] |     get_raw_row();
   3.323 us [397196] |     get_raw_row();
   3.355 us [397196] |     get_raw_row();

...

            [397196] |       process_data_simple_main() {
            [397196] |         pre_process_data() {
   2.158 us [397196] |           jsimd_rgb_ycc_convert();
            [397196] |           sep_downsample() {
            [397196] |             fullsize_downsample() {
   3.553 us [397196] |               jcopy_sample_rows();
   3.991 us [397196] |             } /* fullsize_downsample */
            [397196] |             fullsize_downsample() {
   0.175 us [397196] |               jcopy_sample_rows();
   0.423 us [397196] |             } /* fullsize_downsample */
            [397196] |             fullsize_downsample() {
   2.842 us [397196] |               jcopy_sample_rows();
   3.204 us [397196] |             } /* fullsize_downsample */
   8.563 us [397196] |           } /* sep_downsample */
   1.596 us [397196] |           jsimd_rgb_ycc_convert();
            [397196] |           sep_downsample() {
            [397196] |             fullsize_downsample() {
   0.121 us [397196] |               jcopy_sample_rows();
   0.387 us [397196] |             } /* fullsize_downsample */
            [397196] |             fullsize_downsample() {
   0.127 us [397196] |               jcopy_sample_rows();
   0.370 us [397196] |             } /* fullsize_downsample */
            [397196] |             fullsize_downsample() {
   0.094 us [397196] |               jcopy_sample_rows();
   0.335 us [397196] |             } /* fullsize_downsample */
   1.613 us [397196] |           } /* sep_downsample */

...

            [397196] |           encode_mcu_huff() {
   0.110 us [397196] |             jsimd_huff_encode_one_block();
   0.078 us [397196] |             jsimd_huff_encode_one_block();
   0.101 us [397196] |             jsimd_huff_encode_one_block();
   0.864 us [397196] |           } /* encode_mcu_huff */
            [397196] |           forward_DCT() {
   0.073 us [397196] |             jsimd_convsamp();
   0.086 us [397196] |             jsimd_fdct_ifast();
   0.073 us [397196] |             jsimd_quantize();
   0.780 us [397196] |           } /* forward_DCT */
            [397196] |           forward_DCT() {
   0.072 us [397196] |             jsimd_convsamp();
   0.084 us [397196] |             jsimd_fdct_ifast();
   0.076 us [397196] |             jsimd_quantize();
   0.779 us [397196] |           } /* forward_DCT */
            [397196] |           forward_DCT() {
   0.071 us [397196] |             jsimd_convsamp();
   0.085 us [397196] |             jsimd_fdct_ifast();
   0.074 us [397196] |             jsimd_quantize();
   0.778 us [397196] |           } /* forward_DCT */
            [397196] |           encode_mcu_huff() {
   0.145 us [397196] |             jsimd_huff_encode_one_block();
   0.078 us [397196] |             jsimd_huff_encode_one_block();
   0.078 us [397196] |             jsimd_huff_encode_one_block();
   0.871 us [397196] |           } /* encode_mcu_huff */
            [397196] |           forward_DCT() {
   0.077 us [397196] |             jsimd_convsamp();
   0.086 us [397196] |             jsimd_fdct_ifast();
   0.074 us [397196] |             jsimd_quantize();
   0.783 us [397196] |           } /* forward_DCT */
            [397196] |           forward_DCT() {
   0.071 us [397196] |             jsimd_convsamp();
   0.084 us [397196] |             jsimd_fdct_ifast();
   0.076 us [397196] |             jsimd_quantize();
   0.778 us [397196] |           } /* forward_DCT */
            [397196] |           forward_DCT() {
   0.073 us [397196] |             jsimd_convsamp();
   0.086 us [397196] |             jsimd_fdct_ifast();
   0.073 us [397196] |             jsimd_quantize();
   0.781 us [397196] |           } /* forward_DCT */
```



