This document is written by [Hyuck Kang](https://github.com/kang-hyuck).

![h264_logo](https://github.com/namhyung/uftrace/assets/61424701/8ebd78e9-45fb-436f-b236-aba50de9d4e0)

**JM** (Joint Test Model) is The reference software for H.264/MPEG-4 AVC. Advanced Video Coding (H.264/MPEG-4 AVC) is a joint video coding standardization project of the ITU-T Video Coding Experts Group (ITU-T Q.6/SG 16) and ISO/IEC Moving Picture Experts Group (ISO/IEC JTC 1/SC 29/WG 11).

For more information, the official site is here:  https://avc.hhi.fraunhofer.de/


#### 1. Download source code


```
$ git clone https://vcgit.hhi.fraunhofer.de/jvet/JM.git
$ cd JM
$ git checkout tags/JM-19.1 -b JM-19.1
```

#### 2. Build

```
$ mkdir out
$ cd out
$ export CFLAGS=-pg
$ cmake .. -DCMAKE_BUILD_TYPE=Release
$ make -j32
```

You can find the build outputs in `JM/bin` directory.

```
$ cd ../bin
$ ls -l
-rwxrwxr-x 1 kang-hyuck kang-hyuck  853840  9월 24 12:46 ldecod_static
-rwxrwxr-x 1 kang-hyuck kang-hyuck 1699928  9월 24 12:46 lencod_static
-rwxrwxr-x 1 kang-hyuck kang-hyuck   17600  9월 24 12:46 rtpdump_static
-rwxrwxr-x 1 kang-hyuck kang-hyuck   17840  9월 24 12:46 rtploss_static
drwxrwxr-x 3 kang-hyuck kang-hyuck    4096  9월 24 12:46 umake

```


#### 3. Input test sequences
JM officially provides a sample test sequence, foreman_part_qcif.yuv, 176x144 size, yuv420 foramt, 3 frames.  
you can find sample test sequence in `JM/cfg` directory.

![forman](https://github.com/namhyung/uftrace/assets/61424701/7f4e9153-3b7e-4821-b37c-8a50401f9c1e)


Copy sample test sequences and cfg file to `JM/bin` directory.

```
$ cp ../cfg/encoder.cfg .
$ cp ../cfg/foreman_part_qcif.yuv .
```

#### 4. Record

```
$ uftrace record --no-libcall ./lencod_static
```

![JM_results](https://github.com/namhyung/uftrace/assets/61424701/6a4b0ad9-a11a-44be-8c1d-e3dfad8ffedf)




#### 5. Replay

Enjoy H.264/AVC video codec encoding algorithm with uftrace :)

```
$ uftrace replay

 DURATION     TID     FUNCTION
            [412425] | main() {
   0.194 us [412425] |   init_time();
            [412425] |   Configure() {
   2.259 us [412425] |     InitParams();
  85.252 us [412425] |     GetConfigFileContent();
   1.397 ms [412425] |     ParseContent();
   0.976 us [412425] |     ParseVideoType();
   0.207 us [412425] |     ParseFrameNoFormatFromString();
   0.261 us [412425] |     profile_check();
   1.695 ms [412425] |   } /* Configure */
            [412425] |   level_check() {
   0.246 us [412425] |     get_level_index();
   0.078 us [412425] |     get_level_index();
   0.065 us [412425] |     get_level_index();
   1.662 us [412425] |   } /* level_check */
   6.757 us [412425] |   OpenFiles();
   0.180 us [412425] |   set_storage_format();
            [412425] |   init_qmatrix() {
            [412425] |     get_mem5Dquant() {
  20.489 us [412425] |       get_mem2Dquant();
  22.467 us [412425] |     } /* get_mem5Dquant */
            [412425] |     get_mem5Dquant() {
 219.463 us [412425] |       get_mem2Dquant();
 226.333 us [412425] |     } /* get_mem5Dquant */
 257.652 us [412425] |   } /* init_qmatrix */
            [412425] |   init_qoffset() {
            [412425] |     get_mem3Dshort() {
  49.827 us [412425] |       get_mem2Dshort();
  50.597 us [412425] |     } /* get_mem3Dshort */
            [412425] |     get_mem3Dshort() {
  92.180 us [412425] |       get_mem2Dshort();
  93.664 us [412425] |     } /* get_mem3Dshort */
   4.831 us [412425] |     get_mem2Dshort();
   0.360 us [412425] |     get_mem2Dshort();
  11.178 us [412425] |     InitOffsetParam();
 162.413 us [412425] |   } /* init_qoffset */
   0.098 us [412425] |   init_poc();
            [412425] |   generate_parameter_sets() {
   5.013 us [412425] |     AllocSPS();
   0.262 us [412425] |     GenerateSequenceParameterSet();
   4.472 us [412425] |     AllocPPS();
   0.212 us [412425] |     AllocPPS();
   4.127 us [412425] |     AllocPPS();
   0.170 us [412425] |     GeneratePictureParameterSet();
   0.083 us [412425] |     GeneratePictureParameterSet();
   0.073 us [412425] |     GeneratePictureParameterSet();
  16.902 us [412425] |   } /* generate_parameter_sets */
   0.176 us [412425] |   get_level_index();
            [412425] |   get_mem4Dint() {
  50.673 us [412425] |     get_mem2Dint();
  51.784 us [412425] |   } /* get_mem4Dint */
            [412425] |   get_mem4Dint() {
  14.591 us [412425] |     get_mem2Dint();
  15.630 us [412425] |   } /* get_mem4Dint */
            [412425] |   alloc_mbs() {
   0.448 us [412425] |     get_mem2Dpel();
   0.159 us [412425] |     get_mem2Dpel();
   0.115 us [412425] |     get_mem2Dpel();
   0.114 us [412425] |     get_mem2Dpel();
   0.105 us [412425] |     get_mem2Dpel();

...

  24.525 us [412425] |     WriteAnnexbNALU();
   0.408 us [412425] |     FreeNALU();
            [412425] |     GeneratePic_parameter_set_NALU() {
  47.241 us [412425] |       AllocNALU();
            [412425] |       GeneratePic_parameter_set_rbsp() {
            [412425] |         write_ue_v() {
   0.068 us [412425] |           writeUVLC2buffer();
   0.305 us [412425] |         } /* write_ue_v */
            [412425] |         write_ue_v() {
   0.066 us [412425] |           writeUVLC2buffer();
   0.246 us [412425] |         } /* write_ue_v */
   0.066 us [412425] |         write_u_1();
   0.073 us [412425] |         write_u_1();
            [412425] |         write_ue_v() {
   0.066 us [412425] |           writeUVLC2buffer();
   0.259 us [412425] |         } /* write_ue_v */
            [412425] |         write_ue_v() {
   0.110 us [412425] |           writeUVLC2buffer();
   0.317 us [412425] |         } /* write_ue_v */
            [412425] |         write_ue_v() {
   0.087 us [412425] |           writeUVLC2buffer();
   0.294 us [412425] |         } /* write_ue_v */
   0.074 us [412425] |         write_u_1();
            [412425] |         write_u_v() {
   0.075 us [412425] |           writeUVLC2buffer();
   0.264 us [412425] |         } /* write_u_v */
            [412425] |         write_se_v() {
   0.075 us [412425] |           writeUVLC2buffer();
   0.266 us [412425] |         } /* write_se_v */
            [412425] |         write_se_v() {
   0.075 us [412425] |           writeUVLC2buffer();
   0.255 us [412425] |         } /* write_se_v */
            [412425] |         write_se_v() {
   0.065 us [412425] |           writeUVLC2buffer();
   0.244 us [412425] |         } /* write_se_v */
   0.066 us [412425] |         write_u_1();
   0.065 us [412425] |         write_u_1();
   0.115 us [412425] |         write_u_1();
   0.074 us [412425] |         write_u_1();
   0.066 us [412425] |         write_u_1();
            [412425] |         write_se_v() {
   0.065 us [412425] |           writeUVLC2buffer();
   0.258 us [412425] |         } /* write_se_v */
   0.067 us [412425] |         SODBtoRBSP();
  12.948 us [412425] |       } /* GeneratePic_parameter_set_rbsp */
            [412425] |       RBSPtoNALU() {
   0.109 us [412425] |         RBSPtoEBSP();
   0.303 us [412425] |       } /* RBSPtoNALU */
  61.116 us [412425] |     } /* GeneratePic_parameter_set_NALU */



```



