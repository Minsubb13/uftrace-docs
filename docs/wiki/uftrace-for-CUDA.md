This document is written by [Kang Minchul](https://github.com/kangtegong)


**Prerequisites**

nvcc compiler

   ```
   $ nvcc --version
   nvcc: NVIDIA (R) Cuda compiler driver
   Copyright (c) 2005-2022 NVIDIA Corporation
   Built on Wed_Sep_21_10:33:58_PDT_2022
   Cuda compilation tools, release 11.8, V11.8.89
   Build cuda_11.8.r11.8/compiler.31833905_0
   ```

GPU hardware

   ```
   $ nvidia-smi
   Mon Oct 23 14:31:49 2023
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 520.61.05    Driver Version: 520.61.05    CUDA Version: 11.8     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |                               |                      |               MIG M. |
   |===============================+======================+======================|
   |   0  NVIDIA GeForce ...  On   | 00000000:01:00.0 Off |                  N/A |
   |  0%   41C    P8    29W / 340W |     67MiB / 10240MiB |      0%      Default |
   |                               |                      |                  N/A |
   +-------------------------------+----------------------+----------------------+
   |   1  NVIDIA GeForce ...  On   | 00000000:81:00.0 Off |                  N/A |
   |  0%   45C    P8    27W / 340W |      5MiB / 10240MiB |      0%      Default |
   |                               |                      |                  N/A |
   +-------------------------------+----------------------+----------------------+
   
   +-----------------------------------------------------------------------------+
   | Processes:                                                                  |
   |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
   |        ID   ID                                                   Usage      |
   |=============================================================================|
   |    0   N/A  N/A      1233      G   /usr/lib/xorg/Xorg                 56MiB |
   |    0   N/A  N/A      1681      G   /usr/bin/gnome-shell                9MiB |
   |    1   N/A  N/A      1233      G   /usr/lib/xorg/Xorg                  4MiB |
   +-----------------------------------------------------------------------------+
   ```

   

**1. Compile Cuda Code with `-pg` option**


```c
$ cat hellocu.cu
#include "cuda_runtime.h"
#include "device_launch_parameters.h"
#include <stdio.h>

__global__ void helloCUDA(void)
{
        printf("Hello CUDA from GPU!\n");
}

int main(void)
{
        printf("Hello GPU from CPU!\n");
        helloCUDA << <1, 10 >> > ();
        cudaDeviceSynchronize();
        return 0;
}

```



```
$ nvcc -pg hellocu.cu -o hellocu
```


**2. Record the compilied binary with uftrace**


```
$ uftrace record hellocu
Hello GPU from CPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!
Hello CUDA from GPU!

```

**3. replay the recorded data**

```
$ uftrace replay
# DURATION     TID     FUNCTION
            [  6293] | pthread_once() {
   0.520 us [  6293] |   malloc();
   0.280 us [  6293] |   pthread_mutexattr_init();
   0.130 us [  6293] |   pthread_mutexattr_settype();
   0.140 us [  6293] |   pthread_mutexattr_setpshared();
   0.450 us [  6293] |   pthread_mutex_init();
   0.150 us [  6293] |   pthread_mutexattr_destroy();
   0.260 us [  6293] |   __cxa_atexit();
   9.651 us [  6293] | } /* pthread_once */
   0.170 us [  6293] | __cxa_atexit();
            [  6293] | __sti____cudaRegisterAll() {
   0.150 us [  6293] |   pthread_once();
   0.220 us [  6293] |   malloc();
            [  6293] |   __nv_cudaEntityRegisterCallback() {
   0.140 us [  6293] |     __nv_save_fatbinhandle_for_managed_rt();
   0.100 us [  6293] |     pthread_once();
   0.170 us [  6293] |     malloc();
   1.520 us [  6293] |   } /* __nv_cudaEntityRegisterCallback */
   0.100 us [  6293] |   pthread_once();
   0.150 us [  6293] |   pthread_mutex_lock();
   0.300 us [  6293] |   calloc();
   0.160 us [  6293] |   free();
   0.120 us [  6293] |   malloc();
   0.210 us [  6293] |   pthread_mutex_unlock();
   0.140 us [  6293] |   __cxa_atexit();
  14.750 us [  6293] | } /* __sti____cudaRegisterAll */
            [  6293] | main() {
   9.801 us [  6293] |   puts();
   0.090 us [  6293] |   dim3::dim3();
   0.130 us [  6293] |   dim3::dim3();
   0.110 us [  6293] |   pthread_once();
            [  6293] |   helloCUDA() {
            [  6293] |     __device_stub__Z9helloCUDAv() {
   0.120 us [  6293] |       pthread_once();
            [  6293] |       cudaLaunchKernel() {
            ...

```


**Dynamic Tracing the CUDA-Samples**

**1. Get cuda-samples**

```
$ git clone https://github.com/NVIDIA/cuda-samples.git
$ cd cuda-samples
$ git checkout v11.8 	# checkout to installed CUDA version
```


**2. make cuda-samples**

```
$ sudo make
```


**3. check the binary in cuda-samples/bin/[arch]/linux/release**

```
cuda-samples/bin/x86_64/linux/release$ ls
alignedTypes                               conjugateGradientCudaGraphs     dwtHaar1D               matrixMulDrv                radixSortThrust               simpleCUFFT_MGPU               sortingNetworks
asyncAPI                                   conjugateGradientMultiBlockCG   dxtc                    matrixMulDynlinkJIT         reduction                     simpleDrvRuntime               stereoDisparity
bandwidthTest                              conjugateGradientMultiDeviceCG  eigenvalues             matrixMul_kernel64.fatbin   reductionMultiBlockCG         simpleHyperQ                   streamOrderedAllocation
batchCUBLAS                                conjugateGradientPrecond        fastWalshTransform      matrixMul_nvrtc             scalarProd                    simpleIPC                      streamOrderedAllocationIPC
batchedLabelMarkersAndLabelCompressionNPP  conjugateGradientUM             FDTD3d                  MC_EstimatePiInlineP        scan                          simpleLayeredTexture           streamOrderedAllocationP2P
bf16TensorCoreGemm                         convolutionFFT2D                FilterBorderControlNPP  MC_EstimatePiInlineQ        segmentationTreeThrust        simpleMultiCopy                StreamPriorities
binaryPartitionCG                          convolutionSeparable            fp16ScalarProduct       MC_EstimatePiP              shfl_scan                     simpleMultiGPU                 systemWideAtomics
binomialOptions                            convolutionTexture              freeImageInteropNPP     MC_EstimatePiQ              simpleAssert                  simpleOccupancy                template
binomialOptions_nvrtc                      cppIntegration                  globalToShmemAsyncCopy  MC_SingleAsianOptionP       simpleAssert_nvrtc            simpleP2P                      tf32TensorCoreGemm
BlackScholes                               cppOverload                     graphMemoryFootprint    memMapIPCDrv                simpleAtomicIntrinsics        simplePitchLinearTexture       threadFenceReduction
BlackScholes_nvrtc                         cudaCompressibleMemory          graphMemoryNodes        memMapIpc_kernel64.ptx      simpleAtomicIntrinsics_nvrtc  simplePrintf                   threadMigration
boxFilterNPP                               cudaOpenMP                      histEqualizationNPP     mergeSort                   simpleAttributes              simpleSeparateCompilation      threadMigration_kernel64.fatbin
c++11_cuda                                 cudaTensorCoreGemm              histogram               MersenneTwisterGP11213      simpleAWBarrier               simpleStreams                  topologyQuery
cannyEdgeDetectorNPP                       cuHook                          HSOpticalFlow           MonteCarloMultiGPU          simpleCallback                simpleSurfaceWrite             transpose
cdpAdvancedQuicksort                       cuSolverDn_LinearSolver         immaTensorCoreGemm      newdelete                   simpleCooperativeGroups       simpleTemplates                UnifiedMemoryPerf
cdpBezierTessellation                      cuSolverRf                      inlinePTX               NV12toBGRandResize          simpleCubemapTexture          simpleTemplates_nvrtc          UnifiedMemoryStreams
cdpQuadtree                                cuSolverSp_LinearSolver         inlinePTX_nvrtc         nvJPEG                      simpleCUBLAS                  simpleTexture                  vectorAdd
cdpSimplePrint                             cuSolverSp_LowlevelCholesky     interval                nvJPEG_encoder              simpleCUBLAS_LU               simpleTextureDrv               vectorAddDrv
cdpSimpleQuicksort                         cuSolverSp_LowlevelQR           jacobiCudaGraphs        p2pBandwidthLatencyTest     simpleCUBLASXT                simpleTexture_kernel64.fatbin  vectorAdd_kernel64.fatbin
clock                                      dct8x8                          libcuhook.so.1          ptxjit                      simpleCudaGraphs              simpleVoteIntrinsics           vectorAddMMAP
clock_nvrtc                                deviceQuery                     lineOfSight             ptxjit_kernel64.ptx         simpleCUFFT                   simpleVoteIntrinsics_nvrtc     vectorAdd_nvrtc
concurrentKernels                          deviceQueryDrv                  matrixMul               quasirandomGenerator        simpleCUFFT_2d_MGPU           simpleZeroCopy                 warpAggregatedAtomicsCG
conjugateGradient                          dmmaTensorCoreGemm              matrixMulCUBLAS         quasirandomGenerator_nvrtc  simpleCUFFT_callback          SobolQRNG                      watershedSegmentationNPP

```


**4. replay one of them with full dynamic tracing**

```
$ uftrace -P . ./simpleCallback
Starting simpleCallback
Found 2 CUDA capable GPUs
GPU[0] NVIDIA GeForce RTX 3080 supports SM 8.6, capable GPU Callback Functions
GPU[1] NVIDIA GeForce RTX 3080 supports SM 8.6, capable GPU Callback Functions
2 GPUs available to run Callback Functions
Starting 8 heterogeneous computing workloads
Total of 8 workloads finished:
Success
# DURATION     TID     FUNCTION
            [ 31969] | __cudart426() {
            [ 31969] |   __cudart1635() {
            [ 31969] |     pthread_once() {
            [ 31969] |       __cudart1670() {
   0.140 us [ 31969] |         malloc();
            [ 31969] |         __cudart1332() {
   0.160 us [ 31969] |           pthread_mutexattr_init();
   0.060 us [ 31969] |           pthread_mutexattr_settype();
   0.060 us [ 31969] |           pthread_mutexattr_setpshared();
   0.100 us [ 31969] |           pthread_mutex_init();
   0.060 us [ 31969] |           pthread_mutexattr_destroy();
   1.380 us [ 31969] |         } /* __cudart1332 */
            [ 31969] |         atexit() {
   0.110 us [ 31969] |           __cxa_atexit();
   0.380 us [ 31969] |         } /* atexit */
   2.890 us [ 31969] |       } /* __cudart1670 */
   3.530 us [ 31969] |     } /* pthread_once */
   4.530 us [ 31969] |   } /* __cudart1635 */
   0.060 us [ 31969] |   __cxa_atexit();
   5.320 us [ 31969] | } /* __cudart426 */
            [ 31969] | __sti____cudaRegisterAll() {
            [ 31969] |   __cudaRegisterFatBinary() {
            [ 31969] |     __cudart667() {
            [ 31969] |       __cudart1635() {
   0.040 us [ 31969] |         pthread_once();
   0.320 us [ 31969] |       } /* __cudart1635 */
   0.570 us [ 31969] |     } /* __cudart667 */
            [ 31969] |     __cudart524() {
   0.090 us [ 31969] |       malloc();
   0.340 us [ 31969] |     } /* __cudart524 */
   1.150 us [ 31969] |   } /* __cudaRegisterFatBinary */
            [ 31969] |   __nv_cudaEntityRegisterCallback() {
   0.120 us [ 31969] |     __nv_save_fatbinhandle_for_managed_rt();
            [ 31969] |     __cudaRegisterFunction() {
            [ 31969] |       __cudart667() {
            [ 31969] |         __cudart1635() {
   0.050 us [ 31969] |           pthread_once();
   0.250 us [ 31969] |         } /* __cudart1635 */
   0.380 us [ 31969] |       } /* __cudart667 */
            [ 31969] |       __cudart530() {
   0.060 us [ 31969] |         malloc();
   0.340 us [ 31969] |       } /* __cudart530 */
   0.940 us [ 31969] |     } /* __cudaRegisterFunction */
   1.330 us [ 31969] |   } /* __nv_cudaEntityRegisterCallback */
            [ 31969] |   __cudaRegisterFatBinaryEnd() {
            [ 31969] |     __cudart667() {
            [ 31969] |       __cudart1635() {
...
```