**1. Download depot_tools**

Chromium uses a package of scripts called depot_tools to manage checkouts and code reviews.  
The depot_tools package includes `gclient`, `gcl`, `git-cl`, `repo`, and others.  Chromium uses a package of scripts called `depot_tools` to manage checkouts and code reviews.
```
# download depot_tools
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

# environment setup for depot_tools
$ export PATH=`pwd`/depot_tools:$PATH
```

**2. Download chromium**

First, create a directory `chromium`.
```
$ mkdir chromium && cd chromium
```
Then, create `.gclient` file in the current directory as follows:
```py
solutions = [
  {
    "url": "https://chromium.googlesource.com/chromium/src.git",
    "managed": False,
    "name": "src",
    "custom_deps": {},
  },
]
```
Do (shallow) git clone for chromium source code.
```
$ git clone --depth=1 https://chromium.googlesource.com/chromium/src.git -b 73.0.3676.0
$ cd src
```
Then, download third party modules.
```
$ gclient sync --shallow
```

**3. Change build scripts for -pg build support**

The following `use_uftrace.diff` file contains some changes to apply `-pg` build for uftrace.
```diff
diff --git a/build/config/compiler/BUILD.gn b/build/config/compiler/BUILD.gn
index 54ce77b..204e721 100644
--- a/build/config/compiler/BUILD.gn
+++ b/build/config/compiler/BUILD.gn
@@ -762,6 +762,10 @@ config("compiler") {
     asmflags += cflags
     asmflags += cflags_c
   }
+
+  if (use_uftrace) {
+    cflags += [ "-g", "-pg", "-fno-omit-frame-pointer" ]
+  }
 }
 
 # This provides the basic options to select the target CPU and ABI.
diff --git a/build/config/compiler/compiler.gni b/build/config/compiler/compiler.gni
index 18a7f08..ca00a68 100644
--- a/build/config/compiler/compiler.gni
+++ b/build/config/compiler/compiler.gni
@@ -74,6 +74,9 @@ declare_args() {
   # It's currently not possible to collect AFDO profiles on anything but
   # x86{,_64}.
   using_mismatched_sample_profile = current_cpu != "x64" && current_cpu != "x86"
+
+  # Enables -g -pg -fno-omit-frame-pointer for uftrace
+  use_uftrace = false
 }

 assert(!is_cfi || use_thin_lto, "CFI requires ThinLTO")
diff --git a/third_party/ffmpeg/BUILD.gn b/third_party/ffmpeg/BUILD.gn
index fd79bc4..4a3eaae 100755
--- a/third_party/ffmpeg/BUILD.gn
+++ b/third_party/ffmpeg/BUILD.gn
@@ -7,6 +7,7 @@ import("ffmpeg_generated.gni")

 import("//build/buildflag_header.gni")
 import("//build/config/sanitizers/sanitizers.gni")
+import("//build/config/compiler/compiler.gni")

 # Path to platform configuration files.
 platform_config_root =
@@ -265,6 +266,10 @@ target(link_target_type, "ffmpeg_internal") {
       "-Wno-deprecated-declarations",
     ]

+    if (use_uftrace) {
+      cflags -= [ "-fomit-frame-pointer" ]
+    }
+
     if (!is_clang) {
       # gcc doesn't have flags for specific warnings, so disable them
       # all.
diff --git a/third_party/swiftshader/BUILD.gn b/third_party/swiftshader/BUILD.gn
index 56c78ce..54933c0 100644
--- a/third_party/swiftshader/BUILD.gn
+++ b/third_party/swiftshader/BUILD.gn
@@ -76,6 +76,10 @@ config("swiftshader_config") {
         "-Os",
       ]

+      if (use_uftrace) {
+        cflags -= [ "-fomit-frame-pointer" ]
+      }
+
       defines += [
         "ANGLE_DISABLE_TRACE",
         "NDEBUG",
```
Since those changes have to be made across multiple different repositories, it has to be applied as follows:
```
$ patch -p1 < use_uftrace.diff
patching file build/config/compiler/BUILD.gn
patching file build/config/compiler/compiler.gni
patching file third_party/ffmpeg/BUILD.gn
patching file third_party/swiftshader/BUILD.gn
```

**4. Install build dependencies**

The chrome requires many other external packages to be installed. So the following command installs them.
```
$ sudo ./build/install-build-deps.sh
```

**5. Build for chrome**
```
$ gn gen out/Release.pg.g "--args=enable_nacl=false is_component_build=false is_debug=false use_jumbo_build=true symbol_level=2 use_uftrace=true"

$ ninja -C out/Release.pg.g chrome

$ cd out/Release.pg.g
$ nm ./chrome | grep mcount
                 U mcount
```

**6. Tracing chrome with uftrace**

The following command records a simple chrome execution in headless mode.
```
$ uftrace record -t 5ms --no-libcall ./chrome --no-sandbox --headless https://www.google.com
```

**7. TUI output**

**7.1. Session List**

Chrome launches many processes and their memory maps are different. To distinguish those, uftrace provides session window when it has multiple sessions as follows:
```
$ uftrace tui -t 30ms
Key uftrace command
 G  call Graph for session #1: chrome
    call Graph for session #2: chrome
    call Graph for session #3: chrome
 R  Report functions
 I  uftrace Info
 h  Help message
 q  quit
```
**7.2. Browser Process**
```
  TOTAL TIME : FUNCTION
   13.048  m : (1) chrome
    1.032  m :  ├─(1) main
    1.032  m :  │ (1) ChromeMain
    1.032  m :  │ (1) headless::HeadlessShellMain
    1.032  m :  │ (1) headless::HeadlessShellMain
    1.032  m :  │ (1) headless::HeadlessBrowserMain
    1.032  m :  │ (1) headless::_GLOBAL__N_1::RunContentMain
    1.032  m :  │ (1) content::ContentMain
    1.032  m :  │ (1) service_manager::Main
   45.596  s :  │  ├▶(1) content::ContentServiceManagerMainDelegate::Initialize
             :  │  │
   46.730  s :  │  └─(1) content::ContentServiceManagerMainDelegate::RunEmbedderProcess
   46.730  s :  │    (1) content::ContentMainRunnerImpl::Run
   46.730  s :  │    (1) content::ContentMainRunnerImpl::RunServiceManager
   46.691  s :  │    (1) headless::HeadlessContentMainDelegate::RunProcess
  244.923 ms :  │     ├▶(1) content::BrowserMainRunnerImpl::Initialize
             :  │     │
   46.411  s :  │     ├─(1) content::BrowserMainRunnerImpl::Run
   46.411  s :  │     │ (1) content::BrowserMainLoop::RunMainMessageLoopParts
   46.411  s :  │     │ (1) content::BrowserMainLoop::MainMessageLoopRun
   46.411  s :  │     │ (1) base::RunLoop::Run
   46.411  s :  │     │ (1) base::MessageLoopImpl::Run
   46.411  s :  │     │ (1) base::MessagePumpGlib::Run
   71.332 ms :  │     │  ├▶(1) base::MessageLoopImpl::DoWork
             :  │     │  │
   45.829  s :  │     │  └─(34) g_main_context_iteration
    5.630  s :  │     │     ├─(32) linux:schedule
             :  │     │     │
   40.151  s :  │     │     └─(1) base::_GLOBAL__N_1::WorkSourceDispatch
   40.151  s :  │     │       (1) base::MessageLoopImpl::DoWork
   40.151  s :  │     │       (1) base::MessageLoopImpl::DoWork
   40.151  s :  │     │       (1) base::MessageLoopImpl::RunTask
   40.151  s :  │     │       (1) base::debug::TaskAnnotator::RunTask
   40.151  s :  │     │       (1) base::internal::Invoker::RunOnce
   40.151  s :  │     │       (1) headless::HeadlessDevToolsClientImpl::DispatchEventTask
   40.151  s :  │     │       (1) base::internal::Invoker::Run
   40.151  s :  │     │       (1) headless::page::Domain::DispatchLoadEventFiredEvent
   40.151  s :  │     │       (1) headless::HeadlessShell::OnLoadEventFired
   40.151  s :  │     │       (1) headless::HeadlessShell::OnPageReady
   40.151  s :  │     │      ▶(1) headless::HeadlessShell::Shutdown
```
**7.3. Renderer Process**
```
  TOTAL TIME : FUNCTION
    1.030  m : (1) chrome
   47.364  s :  ├▶(1) main
             :  │
    6.234  s :  ├─(1) content::RendererMain
   98.462 ms :  │  ├▶(1) content::RenderThreadImpl::RenderThreadImpl
             :  │  │
    6.124  s :  │  └─(1) base::RunLoop::Run
    6.124  s :  │    (1) base::MessageLoopImpl::Run
    6.124  s :  │    (1) base::MessagePumpDefault::Run
    4.459  s :  │     ├─(13) base::MessageLoopImpl::DoWork
    4.459  s :  │     │ (13) base::MessageLoopImpl::DoWork
    4.459  s :  │     │ (13) base::MessageLoopImpl::RunTask
    4.459  s :  │     │ (13) base::debug::TaskAnnotator::RunTask
    4.459  s :  │     │ (13) base::internal::Invoker::Run
    4.459  s :  │     │ (13) base::sequence_manager::internal::ThreadControllerImpl::DoWork
    4.344  s :  │     │ (12) base::debug::TaskAnnotator::RunTask
    1.953  s :  │     │  ├─(6) base::internal::Invoker::Run
   65.726 ms :  │     │  │  ├▶(1) IPC::_GLOBAL__N_1::ChannelAssociatedGroupController::AcceptOnProxyThread
             :  │     │  │  │
    1.888  s :  │     │  │  └─(5) blink::TaskHandle::Runner::Run
    1.888  s :  │     │  │    (5) base::internal::Invoker::RunOnce
    1.888  s :  │     │  │    (5) blink::HTMLParserScheduler::ContinueParsing
    1.888  s :  │     │  │    (5) blink::HTMLDocumentParser::PumpPendingSpeculations
    1.803  s :  │     │  │    (6) blink::HTMLDocumentParser::ProcessTokenizedChunkFromBackgroundParser
    1.667  s :  │     │  │     ├─(4) blink::HTMLParserScriptRunner::ProcessScriptElement
    1.667  s :  │     │  │     │ (4) blink::HTMLParserScriptRunner::ProcessScriptElementInternal
    1.667  s :  │     │  │     │ (4) blink::ScriptLoader::PrepareScript
    1.666  s :  │     │  │     │ (4) blink::PendingScript::ExecuteScriptBlock
    1.665  s :  │     │  │     │ (4) blink::PendingScript::ExecuteScriptBlockInternal
    1.665  s :  │     │  │     │ (4) blink::ClassicScript::RunScript
    1.665  s :  │     │  │     │ (4) blink::ScriptController::ExecuteScriptInMainWorld
    1.665  s :  │     │  │     │ (4) blink::ScriptController::EvaluateScriptInMainWorld
    1.665  s :  │     │  │     │ (4) blink::ScriptController::ExecuteScriptAndReturnValue
  135.282 ms :  │     │  │     │  ├▶(2) blink::V8ScriptRunner::CompileScript
             :  │     │  │     │  │
    1.524  s :  │     │  │     │  └─(4) blink::V8ScriptRunner::RunCompiledScript
    1.524  s :  │     │  │     │    (4) v8::Script::Run
    1.524  s :  │     │  │     │    (4) v8::internal::Execution::Call
    1.524  s :  │     │  │     │    (4) v8::internal::_GLOBAL__N_1::Invoke
  721.268 ms :  │     │  │     │     ├─(1) v8::internal::Runtime_GetProperty
  721.267 ms :  │     │  │     │     │ (1) v8::internal::Runtime::GetObjectProperty
  721.264 ms :  │     │  │     │     │ (1) v8::internal::Object::GetProperty
  721.263 ms :  │     │  │     │     │ (1) v8::internal::Object::GetPropertyWithAccessor
  721.260 ms :  │     │  │     │     │ (1) v8::internal::Builtins::InvokeApiFunction
  721.260 ms :  │     │  │     │     │ (1) v8::internal::_GLOBAL__N_1::HandleApiCallHelper
  721.259 ms :  │     │  │     │     │ (1) v8::internal::FunctionCallbackArguments::Call
  721.258 ms :  │     │  │     │     │ (1) blink::V8HTMLElement::OffsetWidthAttributeGetterCallback
  721.257 ms :  │     │  │     │     │ (1) blink::HTMLElement::offsetWidthForBinding
  721.236 ms :  │     │  │     │     │ (1) blink::Document::EnsurePaintLocationDataValidForNode
  719.933 ms :  │     │  │     │     │ (1) blink::Document::UpdateStyleAndLayout
   31.066 ms :  │     │  │     │     │  ├─(1) blink::Document::UpdateStyleAndLayoutTree
             :  │     │  │     │     │  │
  688.864 ms :  │     │  │     │     │  └─(1) blink::LocalFrameView::UpdateLayout
  688.438 ms :  │     │  │     │     │    (1) blink::LocalFrameView::PerformLayout
  688.345 ms :  │     │  │     │     │    (1) blink::LayoutView::UpdateLayout
  688.341 ms :  │     │  │     │     │    (1) blink::LayoutBlock::UpdateLayout
  688.336 ms :  │     │  │     │     │    (1) blink::LayoutView::UpdateBlockLayout
  688.329 ms :  │     │  │     │     │    (1) blink::LayoutBlockFlow::UpdateBlockLayout
  148.730 ms :  │     │  │     │     │     ├▶(1) blink::LayoutBlockFlow::LayoutChildren
             :  │     │  │     │     │     │
  538.604 ms :  │     │  │     │     │     └─(1) blink::LayoutBlock::LayoutPositionedObjects
  538.603 ms :  │     │  │     │     │       (1) blink::LayoutBlock::LayoutPositionedObject
  538.600 ms :  │     │  │     │     │      ▶(1) blink::LayoutBlock::UpdateLayout
             :  │     │  │     │     │
   46.254 ms :  │     │  │     │     ├▶(1) v8::internal::Runtime_LoadIC_Miss
             :  │     │  │     │     │
  596.228 ms :  │     │  │     │     └▶(1) v8::internal::Builtin_HandleApiCall
             :  │     │  │     │
   69.122 ms :  │     │  │     ├▶(1) blink::HTMLTreeBuilder::ConstructTree
             :  │     │  │     │
   66.743 ms :  │     │  │     └▶(1) blink::HTMLDocumentParser::PrepareToStopParsing
             :  │     │  │
    2.390  s :  │     │  └▶(6) base::internal::Invoker::RunOnce
             :  │     │
    1.323  s :  │     └▶(9) base::WaitableEvent::TimedWaitUntil
             :  │
   37.208  s :  └▶(6) base::_GLOBAL__N_1::ThreadFunc
```
**7.4. GPU Process**
```
  TOTAL TIME : FUNCTION
  439.557 ms : (1) chrome
  283.375 ms :  ├─(1) main
  283.375 ms :  │ (1) ChromeMain
  282.174 ms :  │ (1) headless::HeadlessShellMain
  282.174 ms :  │ (1) headless::HeadlessShellMain
  282.169 ms :  │ (1) headless::RunChildProcessIfNeeded
  282.122 ms :  │ (1) headless::_GLOBAL__N_1::RunContentMain
  282.103 ms :  │ (1) content::ContentMain
  282.098 ms :  │ (1) service_manager::Main
   20.642 ms :  │  ├▶(1) content::ContentServiceManagerMainDelegate::Initialize
             :  │  │
  261.199 ms :  │  └─(1) content::ContentServiceManagerMainDelegate::RunEmbedderProcess
  261.199 ms :  │    (1) content::ContentMainRunnerImpl::Run
  261.064 ms :  │    (1) content::RunOtherNamedProcessTypeMain
  261.059 ms :  │    (1) content::GpuMain
   54.474 ms :  │     ├─(1) gpu::GpuInit::InitializeAndStartSandbox
   28.751 ms :  │     │  ├─(1) gl::init::InitializeGLNoExtensionsOneOff
   28.700 ms :  │     │  │ (1) gl::init::_GLOBAL__N_1::InitializeGLOneOffHelper
   28.691 ms :  │     │  │ (1) gl::init::InitializeGLOneOffImplementation
   13.796 ms :  │     │  │  ├▶(1) gl::init::InitializeStaticGLBindings
             :  │     │  │  │
   14.894 ms :  │     │  │  └─(1) gl::init::InitializeGLOneOffPlatform
   14.875 ms :  │     │  │    (1) gl::GLSurfaceEGL::InitializeOneOff
   14.209 ms :  │     │  │    (1) gl::GLSurfaceEGL::InitializeOneOffCommon
   10.358 ms :  │     │  │    (1) gl::InitializeGLContext
   10.357 ms :  │     │  │   ▶(1) gl::GLContextEGL::Initialize
             :  │     │  │
   24.909 ms :  │     │  └▶(2) gpu::_GLOBAL__N_1::CollectGraphicsInfo
             :  │     │
  200.349 ms :  │     └─(1) base::RunLoop::Run
  200.346 ms :  │       (1) base::MessageLoopImpl::Run
  200.346 ms :  │       (1) base::MessagePumpDefault::Run
    5.398 ms :  │        ├▶(1) base::WaitableEvent::TimedWaitUntil
             :  │        │
   28.873 ms :  │        └─(2) base::WaitableEvent::Wait
   28.872 ms :  │          (2) base::WaitableEvent::TimedWaitUntil
   28.835 ms :  │          (2) pthread_cond_wait
   28.812 ms :  │          (2) linux:schedule
             :  │
  156.181 ms :  └▶(3) base::_GLOBAL__N_1::ThreadFunc
```

**8. Visualized trace output of chrome**
- chrome: https://uftrace.github.io/dump/chrome.html
- flame-graph: https://uftrace.github.io/dump/chrome.svg

[![chrome.svg](https://uftrace.github.io/dump/chrome.svg)](https://uftrace.github.io/dump/chrome.svg)