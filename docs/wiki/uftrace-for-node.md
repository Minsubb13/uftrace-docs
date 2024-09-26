**1. Download node with --enable-pg option**
```
$ git clone --depth=2 https://github.com/honggyukim/node.git -b build/pg
$ cd node
$ git log --oneline
61dc940 build: add --enable-pg flag to configure
8e3c5b5 test: fix flaky test-vm-timeout-escape-queuemicrotask
```

**2. Build node with -pg flag enabled**
```
$ ./configure --enable-pg
$ make -j8
$ nm node | grep mcount
                 U mcount@@GLIBC_2.2.5
```
If cross compilation is needed, more options are needed.
```
$ export CC=aarch64-linux-gnu-gcc
$ export CXX=aarch64-linux-gnu-g++
$ export CC_host=gcc
$ export CXX_host=g++
$ ./configure --enable-pg --dest-cpu=arm64 --cross-compiling
$ make -j8
```

**3. Test the generated node image**
```
# Pure node doesn't work but it's the problem of glibc, so ignore it.
$ ./node
Profiling timer expired

# It works fine with uftrace --nop.
$ uftrace --nop ./node -e 'console.log("Hello, node!")'
Hello, node!
```

**4. Run node execution with uftrace**
```
$ uftrace -F main -D 1 ./node -e 'console.log("Hello, node!")'
Hello, node!
# DURATION     TID     FUNCTION
            [ 95495] | main() {
 211.551 us [ 95495] |   /* linux:schedule */
  41.444 us [ 95495] |   /* linux:schedule */
   6.162 us [ 95495] |   /* linux:schedule */
  50.181 us [ 95495] |   /* linux:schedule */
  10.235 us [ 95495] |   /* linux:schedule */
 327.448 us [ 95495] |   /* linux:schedule */
 179.879 ms [ 95495] | } /* main */
```

`/* linux:schedule */` can be hidden by using `--no-event`.
```
$ uftrace -F main -D 1 --no-event ./node -e 'console.log("Hello, node!")'
Hello, node!
# DURATION     TID     FUNCTION
 433.133 ms [ 95530] | main();
```

Let's try it again with depth 3.
```
$ uftrace -F main -D 3 --no-event ./node -e 'console.log("Hello, node!")'
Hello, node!
# DURATION     TID     FUNCTION
            [ 95597] | main() {
            [ 95597] |   node::Start() {
  14.611 us [ 95597] |     node::PlatformInit();
   0.178 us [ 95597] |     uv_hrtime();
   1.321 us [ 95597] |     uv_setup_args();
 167.037 us [ 95597] |     node::Init();
   0.087 us [ 95597] |     uv_mutex_lock();
   0.076 us [ 95597] |     uv_mutex_unlock();
   8.167 us [ 95597] |     v8::V8::SetEntropySource();
   7.761 us [ 95597] |     node::tracing::Agent::Agent();
   0.081 us [ 95597] |     node::tracing::TraceEventHelper::SetAgent();
   0.872 us [ 95597] |     v8::platform::tracing::TracingController::AddTraceStateObserver();
   0.069 us [ 95597] |     node::tracing::Agent::DefaultHandle();
   2.863 ms [ 95597] |     node::NodePlatform::NodePlatform();
  43.750 us [ 95597] |     v8::V8::InitializePlatform();
 263.825 us [ 95597] |     v8::V8::Initialize();
   0.342 us [ 95597] |     uv_hrtime();
   0.136 us [ 95597] |     uv_default_loop();
   5.976 ms [ 95597] |     node::NewIsolate();
   0.185 us [ 95597] |     uv_mutex_lock();
   0.100 us [ 95597] |     uv_mutex_unlock();
   2.519 us [ 95597] |     v8::Locker::Initialize();
   0.549 us [ 95597] |     v8::Isolate::Enter();
   0.287 us [ 95597] |     v8::HandleScope::HandleScope();
 231.842 us [ 95597] |     node::IsolateData::IsolateData();
 168.294 ms [ 95597] |     node::Start();
   1.212 us [ 95597] |     node::IsolateData::~IsolateData();
   0.294 us [ 95597] |     v8::HandleScope::~HandleScope();
   0.329 us [ 95597] |     v8::Isolate::Exit();
   1.529 us [ 95597] |     v8::Locker::~Locker();
   0.169 us [ 95597] |     uv_mutex_lock();
   0.080 us [ 95597] |     uv_mutex_unlock();
 430.770 us [ 95597] |     v8::Isolate::Dispose();
   4.458 us [ 95597] |     node::NodePlatform::UnregisterIsolate();
   0.127 us [ 95597] |     node::ArrayBufferAllocator::~ArrayBufferAllocator();
   0.109 us [ 95597] |     node::tracing::Agent::Disconnect();
  28.440 us [ 95597] |     v8::V8::Dispose();
 269.844 us [ 95597] |     node::NodePlatform::Shutdown();
   1.718 us [ 95597] |     std::_Sp_counted_deleter::_M_dispose();
   0.132 us [ 95597] |     std::_Sp_counted_deleter::_M_destroy();
   0.121 us [ 95597] |     uv_mutex_destroy();
  20.815 us [ 95597] |     node::tracing::Agent::~Agent();
 178.667 ms [ 95597] |   } /* node::Start */
 178.668 ms [ 95597] | } /* main */
```

**5. Trace node with more advanced options**
```
$ uftrace record -a ./node -e 'console.log("Hello, node!")'

$ uftrace replay -t 300ms --no-event
# DURATION     TID     FUNCTION
            [ 95713] | main() {
            [ 95713] |   node::Start() {
            [ 95724] | node::WorkerThreadsTaskRunner::DelayedTaskScheduler::Start::$_0::_FUN() {
            [ 95724] |   uv_run() {
            [ 95724] |     uv__io_poll() {
            [ 95725] | node::_GLOBAL__N_1::PlatformWorkerThread() {
            [ 95726] | node::_GLOBAL__N_1::PlatformWorkerThread() {
            [ 95727] | node::_GLOBAL__N_1::PlatformWorkerThread() {
            [ 95728] | node::_GLOBAL__N_1::PlatformWorkerThread() {
            [ 95713] |     node::Start() {
            [ 95713] |       node::LoadEnvironment() {
            [ 95713] |         v8::Function::Call() {
            [ 95713] |           v8::internal::Execution::Call() {
            [ 95727] |   uv_cond_wait() {
            [ 95725] |   uv_cond_wait() {
            [ 95726] |   uv_cond_wait() {
 350.930 ms [ 95727] |   } /* uv_cond_wait */
 381.967 ms [ 95725] |   } /* uv_cond_wait */
 651.528 ms [ 95713] |           } /* v8::internal::Execution::Call */
 651.531 ms [ 95713] |         } /* v8::Function::Call */
 681.150 ms [ 95713] |       } /* node::LoadEnvironment */
 693.072 ms [ 95713] |     } /* node::Start */
 305.525 ms [ 95726] |   } /* uv_cond_wait */
 716.258 ms [ 95726] | } /* node::_GLOBAL__N_1::PlatformWorkerThread */
 716.556 ms [ 95724] |     } /* uv__io_poll */
 716.577 ms [ 95724] |   } /* uv_run */
 713.749 ms [ 95728] | } /* node::_GLOBAL__N_1::PlatformWorkerThread */
 716.706 ms [ 95724] | } /* node::WorkerThreadsTaskRunner::DelayedTaskScheduler::Start::$_0::_FUN */
 713.948 ms [ 95727] | } /* node::_GLOBAL__N_1::PlatformWorkerThread */
 716.488 ms [ 95725] | } /* node::_GLOBAL__N_1::PlatformWorkerThread */
 717.792 ms [ 95713] |   } /* node::Start */
 717.793 ms [ 95713] | } /* main */
```

**6. Tracing with Pre-Recorded uftrace.data**
```bash
# Download the trace data
$ wget https://github.com/honggyukim/uftrace.data/raw/master/node/uftrace.data.node.tar.gz
$ tar xfz uftrace.data.node.tar.gz
$ cd uftrace.data.node

# Print tracing information for trace data
$ uftrace info

# Print recorded function trace
$ uftrace replay

# Print statistics and summary for trace data
$ uftrace report

# Show function call graph
$ uftrace graph
```
```bash
# Download the trace data
$ wget https://github.com/honggyukim/uftrace.data/raw/master/node/uftrace.data.node.args.tar.gz
$ tar xfz uftrace.data.node.args.tar.gz
$ cd uftrace.data.node.args

# replay the recorded trace data, also shows function arguments and return values
$ uftrace replay

# replay the recorded trace data hiding functions that spent less than 1 milli-seconds
$ uftrace replay -t 1ms

# replay only the call path that invokes "malloc"
$ uftrace replay -C malloc

# launch Terminal-User-Interface (time-filter also works fine with tui)
$ uftrace tui -t 1ms
```

**7. Console replay output of node**
- https://uftrace.github.io/html/node/replay.html

**8. Visualized trace output of node**
- chrome: https://uftrace.github.io/dump/node-marked.html
- flame-graph: https://uftrace.github.io/dump/node-marked.svg

[![node-marked.svg](https://uftrace.github.io/dump/node-marked.svg)](https://uftrace.github.io/dump/node-marked.svg)