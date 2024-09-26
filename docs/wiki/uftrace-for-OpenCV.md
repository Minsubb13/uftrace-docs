This document is written by [Handong Choi](https://github.com/henrychoi7).

#### 1. Download OpenCV source code

```
$ git clone --depth=1 https://github.com/opencv/opencv.git
$ cd opencv/
```

#### 2. Install build tools and dependencies

(Optional) If you need to install required packages:
```
$ sudo apt-get install build-essential
$ sudo apt-get install cmake libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
```

#### 3. Building OpenCV from source using CMake

Create a temporary directory, which denoted as `<cmake_build_dir>`, where you want to put the generated Makefiles, project files as well the object files and output binaries and enter there.
```
# Make sure you are in the source directory
$ cd ~/opencv

$ mkdir build
$ cd build
```

Run `cmake` with `-g pg`, `Debug` options and assign path to the source directory at the end. You can add some optional parameters (e.g. `DCMAKE_INSTALL_PREFIX`) as well.
```
$ cmake -DCMAKE_INSTALL_PREFIX=/home/<username>/opencv/install -DCMAKE_BUILD_TYPE=Debug  -DCMAKE_C_FLAGS='-g -pg' -DCMAKE_CXX_FLAGS='-g -pg' -DBUILD_SHARED_LIBS=OFF /home/<username>/opencv/
```

Execute `make`, it's recommended to do this in several threads.
```
$ make -j4
```

#### 4. Test and check the generated OpenCV binary

```
$ cd bin
$ ./opencv_test_core
CTEST_FULL_OUTPUT
OpenCV version: 4.1.2-pre
OpenCV VCS version: a74fe2e
Build type: Debug
Compiler: /usr/bin/c++  (ver 9.1.0)
Parallel framework: pthreads
CPU features: SSE SSE2 SSE3 *SSE4.1 *SSE4.2 *FP16? *AVX *AVX2 *AVX512-SKX?
...
$ nm ./opencv_test_core | grep mcount
                 U mcount@@GLIBC_2.2.5
```

#### 5. Run test libraries with uftrace

```
$ uftrace -F main -D 2 --no-event ./opencv_test_core
CTEST_FULL_OUTPUT
...
# DURATION     TID     FUNCTION
            [103913] | main() {
   0.232 us [103913] |   cv::utils::trace::details::Region::Region();
   0.110 us [103913] |   cv::utils::trace::details::Region::Region();
   1.968 us [103913] |   cvtest::TS::ptr();
   0.093 us [103913] |   std::allocator::allocator();
   0.801 us [103913] |   std::__cxx11::basic_string::basic_string();
  10.031 us [103913] |   cvtest::TS::init();
   0.095 us [103913] |   std::allocator::~allocator();
   1.262  s [103913] |   testing::InitGoogleTest();
   0.116 us [103913] |   testing::UnitTest::GetInstance();
   0.149 us [103913] |   testing::UnitTest::listeners();
   0.097 us [103913] |   operator new();
   0.153 us [103913] |   cvtest::SystemInfoCollector::SystemInfoCollector();
   3.304 us [103913] |   testing::TestEventListeners::Append();
  12.102 ms [103913] |   cvtest::parseCustomOptions();
   0.148 us [103913] |   cv::utils::trace::details::Region::~Region();
   1.001  h [103913] |   RUN_ALL_TESTS();
   0.203 us [103913] |   cv::utils::trace::details::Region::~Region();
   1.001  h [103913] | } /* main */
```

#### 6. Generate dump output

```
$ uftrace record -F main -D 2 -- ./opencv_test_core
CTEST_FULL_OUTPUT
...
$ uftrace replay
# DURATION     TID     FUNCTION
            [100975] | main() {
...
```

#### 7. Visualized trace output of OpenCV

TUI:
```
  TOTAL TIME : FUNCTION
   22.014  m : (1) opencv_test_core
   22.014  m : (1) main
    0.344 us :  ├─(2) cv::utils::trace::details::Region::Region
             :  │
    2.178 us :  ├─(1) cvtest::TS::ptr
             :  │
    0.098 us :  ├─(1) std::allocator::allocator
             :  │
    0.816 us :  ├─(1) std::__cxx11::basic_string::basic_string
             :  │
   11.437 us :  ├─(1) cvtest::TS::init
             :  │
    0.080 us :  ├─(1) std::allocator::~allocator
             :  │
  641.863 ms :  ├─(1) testing::InitGoogleTest
    3.445 ms :  │ (38) linux:schedule
             :  │
    0.120 us :  ├─(1) testing::UnitTest::GetInstance
             :  │
    0.139 us :  ├─(1) testing::UnitTest::listeners
             :  │
    0.108 us :  ├─(1) operator new
             :  │
    0.117 us :  ├─(1) cvtest::SystemInfoCollector::SystemInfoCollector
             :  │
    3.542 us :  ├─(1) testing::TestEventListeners::Append
             :  │
    4.582 ms :  ├─(1) cvtest::parseCustomOptions
             :  │
    0.322 us :  ├─(2) cv::utils::trace::details::Region::~Region
             :  │
   22.014  m :  └─(1) RUN_ALL_TESTS
    2.032  m :    (149549) linux:schedule

uftrace graph: session 6a78315f43af6420 (/home/henryman/opencv/build/bin/opencv_test_core)              0%
```

Graph:
```
$ uftrace graph
# Function Call Graph for 'opencv_test_core' (session: 6a78315f43af6420)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
   22.014  m : (1) opencv_test_core
   22.014  m : (1) main
    0.344 us :  +-(2) cv::utils::trace::details::Region::Region
             :  |
    2.178 us :  +-(1) cvtest::TS::ptr
             :  |
    0.098 us :  +-(1) std::allocator::allocator
             :  |
    0.816 us :  +-(1) std::__cxx11::basic_string::basic_string
             :  |
   11.437 us :  +-(1) cvtest::TS::init
             :  |
    0.080 us :  +-(1) std::allocator::~allocator
             :  |
  641.863 ms :  +-(1) testing::InitGoogleTest
    3.445 ms :  | (38) linux:schedule
             :  |
    0.120 us :  +-(1) testing::UnitTest::GetInstance
             :  |
    0.139 us :  +-(1) testing::UnitTest::listeners
             :  |
    0.108 us :  +-(1) operator new
             :  |
    0.117 us :  +-(1) cvtest::SystemInfoCollector::SystemInfoCollector
             :  |
    3.542 us :  +-(1) testing::TestEventListeners::Append
             :  |
    4.582 ms :  +-(1) cvtest::parseCustomOptions
             :  |
    0.322 us :  +-(2) cv::utils::trace::details::Region::~Region
             :  |
   22.014  m :  +-(1) RUN_ALL_TESTS
    2.032  m :    (149549) linux:schedule
```

Chrome:
```
$ uftrace record -t 10ms ./opencv_test_core
$ uftrace dump --chrome > opencv.json
$ trace2html opencv.json
opencv.html
```

Flame graph:
```
$ uftrace record -t 10ms ./opencv_test_core
$ uftrace dump --flame-graph | ./flamegraph.pl >opencv.svg
```
