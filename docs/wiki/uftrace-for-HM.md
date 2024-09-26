This document is written by [Hyuck Kang](https://github.com/kang-hyuck).

![hevc_logo](https://github.com/namhyung/uftrace/assets/61424701/24d27733-8d6b-4027-b1d6-8aa7417a537f)


**HM** (HEVC Test Model) is the reference software for Rec. ITU-T H.265 | ISO/IEC 23008-2 High Efficiency Video Coding (HEVC). The software is maintained by the Joint Video Experts Team (JVET) which is a joint collaboration of ITU-T Video Coding Experts Group (VCEG, Question 6 of ITU-T Study Group 16) and the ISO/IEC Moving Picture Experts Group (MPEG, Working Group 5 of Subcommittee 29 of ISO/IEC Joint Technical Committee 1).

For more information, the official site is here:  https://hevc.hhi.fraunhofer.de/


#### 1. Download source code


```
$ git clone https://vcgit.hhi.fraunhofer.de/jvet/HM.git
$ cd HM
$ git checkout tags/HM-18.0 -b HM-18.0
```

#### 2. Build

```
$ mkdir out
$ cd out
$ cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS="-pg" -DCMAKE_CXX_FLAGS="-pg"
$ make -j32
```

You can find the build outputs in `HM/bin` directory.

```
$ cd ../bin
$ ls -l
-rwxrwxr-x 1 kang-hyuck kang-hyuck  939808  9월 24 22:34 MCTSExtractorStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck  450456  9월 24 22:34 SEIFilmGrainAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck  117456  9월 24 22:34 SEIRemovalAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck  999864  9월 24 22:34 TAppDecoderAnalyserStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck  943224  9월 24 22:34 TAppDecoderStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 1804504  9월 24 22:35 TAppEncoderStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck   23480  9월 24 22:33 parcatStatic
drwxrwxr-x 3 kang-hyuck kang-hyuck    4096  9월 24 22:33 umake
```


#### 3. Input test sequences
you can get test sequences here: https://media.xiph.org/video/derf/
  
Download test sequences, ice, 4CIF(704x576), 480 frames, 279MB needed.
```
$ wget https://media.xiph.org/video/derf/y4m/ice_4cif.y4m
```

![ice](https://github.com/namhyung/uftrace/assets/61424701/4952f29b-8914-4a97-ad67-66b347abba24)

Change y4m format to yuv format.

```
$ ffmpeg -i ice_4cif.y4m ice_4cif.yuv
```


#### 4. Record

Please note that the tracing results, `uftrace.data` was **12GB**.

```
$ uftrace record --no-libcall -N ^std ./TAppEncoderStatic -i ice_4cif.yuv -wdt 704 -hgt 576 -fr 24 -f 3 -c ../cfg/encoder_lowdelay_P_main.cfg
```

![hevc_encoding_log](https://github.com/namhyung/uftrace/assets/61424701/788e19fb-7200-4b4c-8cb5-d48138f5156d)

`str.bin` is encoding output and HEVC bitstream file. you can play this with your media player.

![hevc_player](https://github.com/namhyung/uftrace/assets/61424701/e5a1a1dd-103c-491a-a731-44a39ff17ae2)


#### 5. Replay

Enjoy HEVC video codec encoding algorithm with uftrace :)

```
$ uftrace replay

# DURATION     TID     FUNCTION
 115.336 us [424676] | _GLOBAL__sub_I_TAppEncCfg::TAppEncCfg();
   0.334 us [424676] | _GLOBAL__sub_I_TAppEncTop::TAppEncTop();
   0.085 us [424676] | _GLOBAL__sub_I_main();
            [424676] | _GLOBAL__sub_I_Debug.cpp() {
            [424676] |   __static_initialization_and_destruction_0() {
            [424676] |     EnvVar::EnvVar() {
   0.427 us [424676] |       EnvVar::getEnvVarList::cxx11();
   0.085 us [424676] |       splitOnSettings();
   5.280 us [424676] |       lineWrap();
   3.937 us [424676] |       indentNewLines();
  20.426 us [424676] |     } /* EnvVar::EnvVar */
            [424676] |     EnvVar::EnvVar() {
   0.068 us [424676] |       EnvVar::getEnvVarList::cxx11();
   2.418 us [424676] |       splitOnSettings();
   0.182 us [424676] |       lineWrap();
   0.348 us [424676] |       indentNewLines();
   5.289 us [424676] |     } /* EnvVar::EnvVar */
            [424676] |     EnvVar::EnvVar() {
   0.069 us [424676] |       EnvVar::getEnvVarList::cxx11();
   0.202 us [424676] |       splitOnSettings();
   0.100 us [424676] |       lineWrap();
   0.158 us [424676] |       indentNewLines();
   1.599 us [424676] |     } /* EnvVar::EnvVar */
            [424676] |     EnvVar::EnvVar() {
   0.063 us [424676] |       EnvVar::getEnvVarList::cxx11();
   0.124 us [424676] |       splitOnSettings();
   0.126 us [424676] |       lineWrap();
   0.109 us [424676] |       indentNewLines();
   1.444 us [424676] |     } /* EnvVar::EnvVar */
            [424676] |     EnvVar::EnvVar() {
   0.065 us [424676] |       EnvVar::getEnvVarList::cxx11();
   0.159 us [424676] |       splitOnSettings();
   0.072 us [424676] |       lineWrap();
   0.156 us [424676] |       indentNewLines();
   1.351 us [424676] |     } /* EnvVar::EnvVar */
            [424676] |     EnvVar::EnvVar() {
   0.064 us [424676] |       EnvVar::getEnvVarList::cxx11();
   0.158 us [424676] |       splitOnSettings();
   0.457 us [424676] |       lineWrap();
   0.109 us [424676] |       indentNewLines();
   1.839 us [424676] |     } /* EnvVar::EnvVar */
  57.180 us [424676] |   } /* __static_initialization_and_destruction_0 */
  57.451 us [424676] | } /* _GLOBAL__sub_I_Debug.cpp */
   0.114 us [424676] | _GLOBAL__sub_I_ProfileLevelTierFeatures.cpp();
   0.078 us [424676] | _GLOBAL__sub_I_TComChromaFormat.cpp();
   0.070 us [424676] | _GLOBAL__sub_I_TComDataCU.cpp();
   0.072 us [424676] | _GLOBAL__sub_I_TComMotionInfo.cpp();
   0.071 us [424676] | _GLOBAL__sub_I_TComPicYuv.cpp();
   0.071 us [424676] | _GLOBAL__sub_I_TComRom.cpp();
   0.075 us [424676] | _GLOBAL__sub_I_TComSlice.cpp();
   0.072 us [424676] | _GLOBAL__sub_I_TEncGOP.cpp();
   0.076 us [424676] | _GLOBAL__sub_I_TEncRateCtrl.cpp();
   0.072 us [424676] | _GLOBAL__sub_I_TEncSampleAdaptiveOffset.cpp();
   0.076 us [424676] | _GLOBAL__sub_I_TEncSbac.cpp();
   1.036 us [424676] | _GLOBAL__sub_I_TEncSlice.cpp();
   0.079 us [424676] | _GLOBAL__sub_I_TEncTemporalFilter.cpp();
   0.071 us [424676] | _GLOBAL__sub_I_TEncTop.cpp();
   0.073 us [424676] | _GLOBAL__sub_I_WeightPredAnalysis.cpp();
   0.071 us [424676] | _GLOBAL__sub_I_NALwrite.cpp();
   0.076 us [424676] | _GLOBAL__sub_I_SEIEncoder.cpp();
   0.075 us [424676] | _GLOBAL__sub_I_SEIwrite.cpp();
   0.080 us [424676] | _GLOBAL__sub_I_SyntaxElementWriter.cpp();
   0.077 us [424676] | _GLOBAL__sub_I_TEncBinCoderCABAC.cpp();
   0.083 us [424676] | _GLOBAL__sub_I_TEncBinCoderCABACCounter.cpp();
   0.076 us [424676] | _GLOBAL__sub_I_TEncCavlc.cpp();

...

   0.061 us [424676] |           TComYuv::TComYuv();
            [424676] |           TComYuv::create() {
   0.060 us [424676] |             TComYuv::destroy();
   0.347 us [424676] |           } /* TComYuv::create */
            [424676] |           initZscanToRaster() {
            [424676] |             initZscanToRaster() {
            [424676] |               initZscanToRaster() {
            [424676] |                 initZscanToRaster() {
   0.118 us [424676] |                   initZscanToRaster();
   0.112 us [424676] |                   initZscanToRaster();
   0.063 us [424676] |                   initZscanToRaster();
   8.788 us [424676] |                 } /* initZscanToRaster */
            [424676] |                 initZscanToRaster() {
   0.068 us [424676] |                   initZscanToRaster();
   0.060 us [424676] |                   initZscanToRaster();
   0.060 us [424676] |                   initZscanToRaster();
   0.764 us [424676] |                 } /* initZscanToRaster */
            [424676] |                 initZscanToRaster() {
   0.070 us [424676] |                   initZscanToRaster();
   0.060 us [424676] |                   initZscanToRaster();
   0.061 us [424676] |                   initZscanToRaster();
   0.756 us [424676] |                 } /* initZscanToRaster */
   0.061 us [424676] |                 initZscanToRaster();
   0.060 us [424676] |                 initZscanToRaster();
   0.060 us [424676] |                 initZscanToRaster();
  11.519 us [424676] |               } /* initZscanToRaster */

...

            [424676] |         TComSampleAdaptiveOffset::create() {
   0.134 us [424676] |           TComSampleAdaptiveOffset::destroy();
   0.056 us [424676] |           TComPicYuv::TComPicYuv();
            [424676] |           TComPicYuv::create() {
            [424676] |             TComPicYuv::createWithoutCUInfo() {
   0.081 us [424676] |               TComPicYuv::destroy();
  16.886 us [424676] |             } /* TComPicYuv::createWithoutCUInfo */
  20.836 us [424676] |           } /* TComPicYuv::create */
  22.213 us [424676] |         } /* TComSampleAdaptiveOffset::create */
            [424676] |         TEncSampleAdaptiveOffset::createEncData() {
            [424676] |           TEncSbac::TEncSbac() {
   0.065 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.058 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.075 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.056 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   0.057 us [424676] |             ContextModel3DBuffer::ContextModel3DBuffer();
   7.374 us [424676] |           } /* TEncSbac::TEncSbac */
            [424676] |           TEncBinCABACCounter::TEncBinCABACCounter() {
   0.058 us [424676] |             TEncBinCABAC::TEncBinCABAC();
   0.334 us [424676] |           } /* TEncBinCABACCounter::TEncBinCABACCounter */
```