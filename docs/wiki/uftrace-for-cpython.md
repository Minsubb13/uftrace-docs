**1. Download cpython**
```
$ git clone --depth=1 https://github.com/python/cpython.git
```

**2. Configuration**
```
$ cd cpython/
$ ./configure CFLAGS='-pg -g'
```

**3. Compilation**
```
$ make -j4
```

**4. Check the generated binary**
```
$ ./python -c 'print("Hello")'
Hello

$ nm python | grep mcount
                 U mcount@@GLIBC_2.2.5
```

**5. Run python interpreter with uftrace**
```
$ uftrace -D 4 ./python -c 'print("Hello")'
Hello
# DURATION     TID     FUNCTION
            [  1015] | main() {
            [  1015] |   Py_BytesMain() {
            [  1015] |     pymain_init() {
  76.561 us [  1015] |       _PyRuntime_Initialize();
   0.186 us [  1015] |       PyPreConfig_InitPythonConfig();
 183.022 us [  1015] |       _Py_PreInitializeFromPyArgv();
   0.250 us [  1015] |       PyConfig_InitPythonConfig();
   6.287 us [  1015] |       PyConfig_SetBytesArgv();
            [  1015] |       Py_InitializeFromConfig() {
  23.847 us [  1015] |         /* linux:schedule */
  70.295 ms [  1015] |       } /* Py_InitializeFromConfig */
   9.540 us [  1015] |       PyConfig_Clear();
  70.586 ms [  1015] |     } /* pymain_init */
            [  1015] |     Py_RunMain() {
   1.533 us [  1015] |       _PyPathConfig_ComputeSysPath0();
   4.037 us [  1015] |       _PyDict_GetItemIdWithError();
   0.990 us [  1015] |       PyList_Insert();
   1.744 us [  1015] |       PyUnicode_FromWideChar();
   0.220 us [  1015] |       PySys_Audit();
   1.153 us [  1015] |       PyUnicode_AsUTF8String();
   0.324 us [  1015] |       _Py_Dealloc();
   0.160 us [  1015] |       PyBytes_AsString();
   1.276 ms [  1015] |       PyRun_SimpleStringFlags();
   0.550 us [  1015] |       _Py_Dealloc();
   1.240 us [  1015] |       _Py_GetEnv();
  12.330 ms [  1015] |       Py_FinalizeEx();
   0.807 us [  1015] |       _PyImport_Fini2();
   3.453 us [  1015] |       _PyPathConfig_ClearGlobal();
   0.344 us [  1015] |       _Py_ClearStandardStreamEncoding();
   1.844 us [  1015] |       _Py_ClearArgcArgv();
   0.380 us [  1015] |       _PyRuntime_Finalize();
  13.634 ms [  1015] |     } /* Py_RunMain */
  84.221 ms [  1015] |   } /* Py_BytesMain */
  84.223 ms [  1015] | } /* main */
```

**6. Trace python with arguments and return values**
```
$ uftrace record -D 4 -a ./python -c 'print("Hello")'
Hello

$ uftrace replay
# DURATION     TID     FUNCTION
            [  1082] | main(3, 0x7ffe8fceb688) {
            [  1082] |   Py_BytesMain(3, 0x7ffe8fceb688) {
            [  1082] |     pymain_init(0x7ffe8fceb520) {
  84.278 us [  1082] |       _PyRuntime_Initialize();
   0.360 us [  1082] |       PyPreConfig_InitPythonConfig(0x7ffe8fceb340);
 211.232 us [  1082] |       _Py_PreInitializeFromPyArgv(0x7ffe8fceb340, 0x7ffe8fceb520);
   1.047 us [  1082] |       PyConfig_InitPythonConfig(0x7ffe8fceb370) = 0x7ffe8fceb320;
   9.510 us [  1082] |       PyConfig_SetBytesArgv(0x7ffe8fceb370, 3, 0x7ffe8fceb688) = 0x7ffe8fceb320;
 132.287 ms [  1082] |       Py_InitializeFromConfig(0x7ffe8fceb370);
  14.447 us [  1082] |       PyConfig_Clear(0x7ffe8fceb370);
 132.626 ms [  1082] |     } = 0x7ffe8fceb540; /* pymain_init */
            [  1082] |     Py_RunMain() {
   2.504 us [  1082] |       _PyPathConfig_ComputeSysPath0(0x26d55e0, 0x7ffe8fceb438) = 1;
   9.017 us [  1082] |       _PyDict_GetItemIdWithError(0x7fc0e2f64e00, &PyId_path.13766) = 0x7fc0e2f73ec0;
   1.893 us [  1082] |       PyList_Insert(0x7fc0e2f73ec0, 0, 0x7fc0e2fd42f0) = 0;
   2.787 us [  1082] |       PyUnicode_FromWideChar(0x26f1b40, -1) = 0x7fc0e2f1cbb0;
 400.904 us [  1082] |       PySys_Audit("cpython.run_command", "O") = 0;
   3.457 us [  1082] |       PyUnicode_AsUTF8String(0x7fc0e2f1cbb0) = 0x7fc0e2f48690;
   0.934 us [  1082] |       _Py_Dealloc(0x7fc0e2f1cbb0);
   0.273 us [  1082] |       PyBytes_AsString(0x7fc0e2f48690) = "print('Hello')\n";
   1.723 ms [  1082] |       PyRun_SimpleStringFlags("print('Hello')\n", 0x7ffe8fceb438);
   1.410 us [  1082] |       _Py_Dealloc(0x7fc0e2f48690);
   2.040 us [  1082] |       _Py_GetEnv(1, "PYTHONINSPECT");
  23.279 ms [  1082] |       Py_FinalizeEx();
   1.440 us [  1082] |       _PyImport_Fini2();
   6.104 us [  1082] |       _PyPathConfig_ClearGlobal();
   0.496 us [  1082] |       _Py_ClearStandardStreamEncoding();
   3.417 us [  1082] |       _Py_ClearArgcArgv();
   0.820 us [  1082] |       _PyRuntime_Finalize();
  25.459 ms [  1082] |     } = 0; /* Py_RunMain */
 158.089 ms [  1082] |   } = 0; /* Py_BytesMain */
 158.093 ms [  1082] | } = 0; /* main */
```

**6. Generate chrome dump output**
```
# record
$ uftrace record -t 4ms ./python -c 'print("Hello")'
Hello

# dump in json format
$ uftrace dump --chrome > cpython.json

# convert json to html
$ trace2html cpython.json
cpython.html
```

**7. Visualized trace output of cpython**
- chrome: https://uftrace.github.io/dump/cpython.html
- flame-graph: https://uftrace.github.io/dump/cpython.svg

[![cpython.svg](https://uftrace.github.io/dump/cpython.svg)](https://uftrace.github.io/dump/cpython.svg)