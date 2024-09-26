This document is written by [Hyuck Kang](https://github.com/kang-hyuck).

![vvc_logo](https://github.com/namhyung/uftrace/assets/61424701/f895c784-807d-4d31-8fc9-3c3d7da9367a)

**VTM** (VVC Test Model) is the reference software for Rec. ITU-T H.266 | ISO/IEC 23090-3 Versatile Video Coding (VVC). The reference software includes both encoder and decoder functionality. The software has been jointly developed by the ITU-T Video Coding Experts Group (VCEG, Question 6 of ITU-T Study Group 16) and the ISO/IEC Moving Picture Experts Group (MPEG Joint Video Coding Team(s) with ITU-T SG 16, Working Group 5 of Subcommittee 29 of ISO/IEC Joint Technical Committee 1).

For more information, the official site is here:  https://jvet.hhi.fraunhofer.de/


#### 1. Download source code


```
$ git clone https://vcgit.hhi.fraunhofer.de/jvet/VVCSoftware_VTM.git
$ cd VVCSoftware_VTM
$ git checkout tags/VTM-22.0 -b VTM-22.0
```

#### 2. Build

```
$ mkdir out
$ cd out
$ cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS="-pg" -DCMAKE_CXX_FLAGS="-pg"
$ make -j32
```

You can find the build outputs in `VVCSoftware_VTM/bin` directory.

```
$ cd ../bin
$ ls -l
-rwxrwxr-x 1 kang-hyuck kang-hyuck 2356624  9월 27 22:05 BitstreamExtractorAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 3221880  9월 27 22:05 DecoderAnalyserAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 3170512  9월 27 22:05 DecoderAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 5828592  9월 27 22:05 EncoderAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 2300048  9월 27 22:05 SEIFilmGrainAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck  128464  9월 27 22:05 SEIRemovalAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 1919384  9월 27 22:05 StreamMergeAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 2480752  9월 27 22:05 SubpicMergeAppStatic
-rwxrwxr-x 1 kang-hyuck kang-hyuck 1789472  9월 27 22:05 parcatStatic
drwxrwxr-x 3 kang-hyuck kang-hyuck    4096  9월 27 22:05 umake
```


#### 3. Input test sequences
you can get test sequences here: https://media.xiph.org/video/derf/
  
Download test sequences, football (b), QCIF(176x144), 260 frames, 15fps, 4.8MB needed.
```
$ wget https://media.xiph.org/video/derf/y4m/football_qcif_15fps.y4m
```

![image](https://github.com/namhyung/uftrace/assets/61424701/612e8fbc-fb1b-47c2-bd97-df1ea75fd270)

Change y4m format to yuv format.

```
$ ffmpeg -i football_qcif_15fps.y4m football_qcif_15fps.yuv
```


#### 4. Record

Please note that the tracing results, `uftrace.data` was **1.2GB**.

```
$ uftrace record --no-libcall -D 12 -N ^std -N ^_GLOBAL__ ./EncoderAppStatic -i football_qcif_15fps.yuv -wdt 176 -hgt 144 -c ../cfg/encoder_randomaccess_vtm.cfg -f 5 -fr 15
```

![vvc_encoding_log](https://github.com/namhyung/uftrace/assets/61424701/e5fc6ef5-13aa-41bc-90ba-4159da966975)


`str.bin` is encoding output which is VVC bitstream file.  
you can use VVC decoder in VTM.

```
$ ./DecoderAppStatic -b str.bin -o decoded.yuv
```

Run the decoded yuv file using yuv player.

![vvc_decodeded](https://github.com/namhyung/uftrace/assets/61424701/8f085ba9-e4b5-4c26-bf4d-03d6fa9e4d97)


#### 5. Replay

Enjoy VVC video codec encoding algorithm with uftrace :)

```
$ uftrace replay

# DURATION     TID     FUNCTION
            [  7294] | main() {
   0.590 us [  7294] |   ProgramOptionsLite::Options::addOptions();
   5.473 us [  7294] |   ProgramOptionsLite::Options::addOption();
   0.892 us [  7294] |   ProgramOptionsLite::Options::addOption();
            [  7294] |   ProgramOptionsLite::scanArgv::cxx11() {
            [  7294] |     ProgramOptionsLite::ArgvParser::parseSHORT() {
            [  7294] |       ProgramOptionsLite::OptionWriter::storePair() {
   0.060 us [  7294] |         ProgramOptionsLite::ArgvParser::where::cxx11();
   0.070 us [  7294] |         ProgramOptionsLite::SilentReporter::error();
  15.425 us [  7294] |       } /* ProgramOptionsLite::OptionWriter::storePair */
  18.454 us [  7294] |     } /* ProgramOptionsLite::ArgvParser::parseSHORT */
            [  7294] |     ProgramOptionsLite::ArgvParser::parseSHORT() {
            [  7294] |       ProgramOptionsLite::OptionWriter::storePair() {
   0.060 us [  7294] |         ProgramOptionsLite::ArgvParser::where::cxx11();
   0.059 us [  7294] |         ProgramOptionsLite::SilentReporter::error();
   1.212 us [  7294] |       } /* ProgramOptionsLite::OptionWriter::storePair */
   1.634 us [  7294] |     } /* ProgramOptionsLite::ArgvParser::parseSHORT */
            [  7294] |     ProgramOptionsLite::ArgvParser::parseSHORT() {
            [  7294] |       ProgramOptionsLite::OptionWriter::storePair() {
   0.060 us [  7294] |         ProgramOptionsLite::ArgvParser::where::cxx11();
   0.058 us [  7294] |         ProgramOptionsLite::SilentReporter::error();
   0.889 us [  7294] |       } /* ProgramOptionsLite::OptionWriter::storePair */
   1.152 us [  7294] |     } /* ProgramOptionsLite::ArgvParser::parseSHORT */
            [  7294] |     ProgramOptionsLite::ArgvParser::parseSHORT() {
            [  7294] |       ProgramOptionsLite::OptionWriter::storePair() {
            [  7294] |         ProgramOptionsLite::OptionFunc::parse() {
            [  7294] |           ProgramOptionsLite::parseConfigFile() {
            [  7294] |             ProgramOptionsLite::CfgStreamParser::scanStream() {
   0.490 us [  7294] |               ProgramOptionsLite::CfgStreamParser::scanLine();
            [  7294] |               ProgramOptionsLite::CfgStreamParser::scanLine() {
            [  7294] |                 ProgramOptionsLite::OptionWriter::storePair() {
   8.852 us [  7294] |                   ProgramOptionsLite::CfgStreamParser::where::cxx11();
   0.085 us [  7294] |                   ProgramOptionsLite::SilentReporter::error();
  10.454 us [  7294] |                 } /* ProgramOptionsLite::OptionWriter::storePair */
  11.799 us [  7294] |               } /* ProgramOptionsLite::CfgStreamParser::scanLine */

...

            [ 10594] |           CodingStructure::create() {
            [ 10594] |             UnitArea::UnitArea() {
   0.078 us [ 10594] |               CompArea::xRecalcLumaToChroma();
   0.068 us [ 10594] |               CompArea::xRecalcLumaToChroma();
   0.061 us [ 10594] |               CompArea::xRecalcLumaToChroma();
   0.856 us [ 10594] |             } /* UnitArea::UnitArea */
            [ 10594] |             CodingStructure::createInternals() {
   2.213 us [ 10594] |               CodingStructure::createCoeffs();
            [ 10594] |               CodingStructure::initStructData() {
   0.455 us [ 10594] |                 CodingStructure::clearPUs();
   0.465 us [ 10594] |                 CodingStructure::clearTUs();
   0.314 us [ 10594] |                 CodingStructure::clearCUs();
   0.062 us [ 10594] |                 CodingStructure::getMotionBuf();
   2.232 us [ 10594] |               } /* CodingStructure::initStructData */
   5.762 us [ 10594] |             } /* CodingStructure::createInternals */
            [ 10594] |             PelStorage::create() {
   0.673 us [ 10594] |               PelStorage::create();
   0.978 us [ 10594] |             } /* PelStorage::create */
            [ 10594] |             PelStorage::create() {
   0.163 us [ 10594] |               PelStorage::create();
   0.418 us [ 10594] |             } /* PelStorage::create */
            [ 10594] |             PelStorage::create() {
   0.195 us [ 10594] |               PelStorage::create();
   0.460 us [ 10594] |             } /* PelStorage::create */
            [ 10594] |             PelStorage::create() {
   0.165 us [ 10594] |               PelStorage::create();
   0.432 us [ 10594] |             } /* PelStorage::create */
  10.064 us [ 10594] |           } /* CodingStructure::create */
            [ 10594] |           CodingStructure::create() {
            [ 10594] |             UnitArea::UnitArea() {
   0.069 us [ 10594] |               CompArea::xRecalcLumaToChroma();
   0.060 us [ 10594] |               CompArea::xRecalcLumaToChroma();
   0.060 us [ 10594] |               CompArea::xRecalcLumaToChroma();
   0.808 us [ 10594] |             } /* UnitArea::UnitArea */
            [ 10594] |             CodingStructure::createInternals() {
   1.840 us [ 10594] |               CodingStructure::createCoeffs();
            [ 10594] |               CodingStructure::initStructData() {
   0.295 us [ 10594] |                 CodingStructure::clearPUs();
   0.299 us [ 10594] |                 CodingStructure::clearTUs();
   0.263 us [ 10594] |                 CodingStructure::clearCUs();
   0.061 us [ 10594] |                 CodingStructure::getMotionBuf();
   1.745 us [ 10594] |               } /* CodingStructure::initStructData */
   4.211 us [ 10594] |             } /* CodingStructure::createInternals 

```