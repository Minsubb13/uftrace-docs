**1. Download LLVM Project 12.0.0 and uncompress**
```
$ wget https://github.com/llvm/llvm-project/releases/download/llvmorg-12.0.0/llvm-project-12.0.0.src.tar.xz

$ tar xfx llvm-project-12.0.0.src.tar.xz

$ cd llvm-project-12.0.0
```


**2. Configuration**
```
$ mkdir build.release.pg.g && cd build.release.pg.g

$ cmake -G Ninja ../llvm -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" \
      -DLLVM_TARGETS_TO_BUILD="host;ARM;AArch64" -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-pg -g"

```
if you want see more detail and non-optimize code flow use Debug build but Debug build require many memory.
so if you has not enough memory than use gold instead ld.

```
$ cmake -G Ninja ../llvm -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" \
      -DLLVM_TARGETS_TO_BUILD="host;ARM;AArch64" -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-pg -g" \
      -DLLVM_USE_LINKER=gold
```

Reference: https://llvm.org/docs/GettingStarted.html#getting-the-source-code-and-building-llvm

**3. Build clang**
```
$ ninja clang
```

**4. Test the generated clang image**
```
$ cd ./bin

$ ./clang
clang-12: error: no input files

$ nm clang | grep mcount
                 U mcount@@GLIBC_2.2.5
```

**5. Run clang execution with uftrace**
```
$ uftrace -F main -D 2 --no-libcall -- ./clang
clang-12: error: no input files
# DURATION     TID     FUNCTION
            [ 39782] | main() {
   0.269 us [ 39782] |   clang::noteBottomOfStack();
  27.025 us [ 39782] |   llvm::InitLLVM::InitLLVM();
   3.686 us [ 39782] |   llvm::setBugReportMsg();
   0.082 us [ 39782] |   llvm::SmallVectorImpl::append();
   7.051 us [ 39782] |   llvm::sys::Process::FixupStandardFileDescriptors();
   0.548 us [ 39782] |   LLVMInitializeARMTargetInfo();
   0.763 us [ 39782] |   LLVMInitializeAArch64TargetInfo();
   0.253 us [ 39782] |   LLVMInitializeX86TargetInfo();
 140.562 us [ 39782] |   LLVMInitializeARMTarget();
 120.991 us [ 39782] |   LLVMInitializeAArch64Target();
  85.000 us [ 39782] |   LLVMInitializeX86Target();
   9.391 us [ 39782] |   clang::driver::ToolChain::getTargetAndModeFromProgramName();
   0.276 us [ 39782] |   llvm::vfs::getRealFileSystem();
   0.487 us [ 39782] |   llvm::cl::ExpandResponseFiles();
   0.051 us [ 39782] |   llvm::ThreadSafeRefCountedBase::Release();
  42.694 us [ 39782] |   GetExecutablePath::cxx11();
 535.016 us [ 39782] |   clang::driver::getDriverOptTable();
   0.346 us [ 39782] |   llvm::opt::OptTable::ParseArgs();
 159.267 us [ 39782] |   clang::ParseDiagnosticArgs();
   0.224 us [ 39782] |   llvm::opt::ArgList::hasFlag();
   0.247 us [ 39782] |   llvm::opt::InputArgList::~InputArgList();
   2.254 us [ 39782] |   llvm::errs();
   0.073 us [ 39782] |   clang::TextDiagnosticPrinter::TextDiagnosticPrinter();
   0.707 us [ 39782] |   llvm::sys::path::stem();
   0.114 us [ 39782] |   std::__cxx11::basic_string::_M_construct();
   0.057 us [ 39782] |   clang::DiagnosticIDs::DiagnosticIDs();
   0.284 us [ 39782] |   clang::DiagnosticsEngine::DiagnosticsEngine();
   0.610 us [ 39782] |   clang::ProcessWarningOptions();
   0.172 us [ 39782] |   std::__cxx11::basic_string::_M_construct();
  20.206 us [ 39782] |   llvm::sys::getDefaultTargetTriple::cxx11();
  11.311 us [ 39782] |   clang::driver::Driver::Driver();
   0.106 us [ 39782] |   llvm::SmallVectorImpl::append();
   0.376 us [ 39782] |   llvm::sys::path::filename();
   9.408 us [ 39782] |   llvm::sys::fs::make_absolute();
   0.322 us [ 39782] |   llvm::sys::path::parent_path();
   2.278 us [ 39782] |   llvm::sys::fs::access();
   4.258 us [ 39782] |   llvm::CrashRecoveryContext::Enable();
   1.262 ms [ 39782] |   clang::driver::Driver::BuildCompilation();
   0.235 us [ 39782] |   clang::driver::Driver::ExecuteCompilation();
   0.056 us [ 39782] |   clang::DiagnosticConsumer::finish();
   0.082 us [ 39782] |   llvm::errs();
   0.517 us [ 39782] |   llvm::TimerGroup::printAll();
   0.183 us [ 39782] |   llvm::TimerGroup::clearAll();
   1.274 us [ 39782] |   clang::driver::Compilation::~Compilation();
   9.895 us [ 39782] |   clang::driver::Driver::~Driver();
   1.882 us [ 39782] |   clang::DiagnosticsEngine::~DiagnosticsEngine();
   0.159 us [ 39782] |   llvm::RefCountedBase::Release();
   0.200 us [ 39782] |   llvm::RefCountedBase::Release();
   0.066 us [ 39782] |   std::_Rb_tree::_M_erase();
   0.103 us [ 39782] |   llvm::BumpPtrAllocatorImpl::~BumpPtrAllocatorImpl();
  93.122 us [ 39782] |   llvm::InitLLVM::~InitLLVM();
   2.628 ms [ 39782] | } /* main */
```

**7. Visualized trace output of clang**

[![uftrace-chrome-dump](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-chrome.png)](https://uftrace.github.io/dump/clang.tmp.fib.html)