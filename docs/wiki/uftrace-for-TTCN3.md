# uftrace for TTCN3

[TTCN-3](https://en.wikipedia.org/wiki/TTCN-3) (Testing and Test Control Notation version 3) is a testing language used in [conformance testing](https://en.wikipedia.org/wiki/Conformance_testing) of communicating systems.

TTCN-3 has been used to define conformance test suites.

- The [Open Mobile Alliance](https://en.wikipedia.org/wiki/Open_Mobile_Alliance) adopted in 2008 a strategy of using TTCN-3 for translating some of the test cases in an enabler test specification into an executable representation.
- The [AUTOSAR](https://en.wikipedia.org/wiki/AUTOSAR) project promoted (2008) the use of TTCN-3 within the automotive industry.
- The [3GPP](https://en.wikipedia.org/wiki/3GPP) project promoted the use of TTCN-3 within the mobile industry.

We can install `TTCN3` Toolset `eclipse-titan`.
```
$  sudo apt install eclipse-titan
```

Take the source code for `intel`'s `net-test-suites` for `TTCN-3`, and do uftrace tracing.

```
$ git clone https://github.com/intel/net-test-suites.git
```
Building TTCN-3 sequence is `.ttcn` -> `.cc` -> Executable binary.

We need to add `-pg` options.

```diff
diff --git a/src/make.sh b/src/make.sh
index 4710bdb..3e1486a 100755
--- a/src/make.sh
+++ b/src/make.sh
@@ -17,7 +17,7 @@ ttcn3_makefilegen -e test_suite \

     sed -i 's#^CPPFLAGS =.*#CPPFLAGS += -D$(PLATFORM) -I. -I$(TTCN3_DIR)/include -I$(TTCN3_DIR)/include/titan#' Makefile

-    sed -i 's/^CXXFLAGS =.*/CXXFLAGS += -O3 -Wno-misleading-indentation -Wno-pointer-arith/' Makefile
+    sed -i 's/^CXXFLAGS =.*/CXXFLAGS += -O3 -pg -Wno-misleading-indentation -Wno-pointer-arith/' Makefile

     sed -i 's#^LDFLAGS =.*#LDFLAGS += -L$(TTCN3_DIR)/lib/titan#' Makefile
```

We will use 2 terminal. First `mctr_cli` for the test response.

```
$ mctr_cli tcp2_check_3_runs.cfg

*************************************************************************
* TTCN-3 Test Executor - Main Controller 2                              *
* Version: CRL 113 200/6 R6B                                            *
* Copyright (c) 2000-2019 Ericsson Telecom AB                           *
* All rights reserved. This program and the accompanying materials      *
* are made available under the terms of the Eclipse Public License v2.0 *
* which accompanies this distribution, and is available at              *
* https://www.eclipse.org/org/documents/epl-2.0/EPL-2.0.html            *
*************************************************************************

Using configuration file: tcp2_check_3_runs.cfg
MC@: Unix server socket created successfully.
MC@: Listening on TCP port 41571.
```

Second terminal for the `test_suite` program tracing.

Port number follow `mctr_cli` message `MC@: Listening on TCP port 41571.`

```
$ uftrace record ./test_suite DESKTOP-NK4TH6S. 41571
```

```
MC@: New HC connected from localhost [127.0.0.1]. DESKTOP-NK4TH6S: Linux 5.15.133.1-microsoft-standard-WSL2 on x86_64.

MC2> cmtc

MC@: Downloading configuration file to all HCs.
MC@: Configuration file was processed on all HCs.
MC@: Creating MTC on host localhost.
MC@: MTC is created.

MC2>  smtc tcp2_check.test_tcp_connect_data_close

MTC@: Dynamic test case error: Test case test_tcp_connect_data_close does not exist in module tcp2_check.
MC@: Test execution finished.

MC2>  exit

MC@: Terminating MTC.
MTC@: Verdict statistics: 0 none, 0 pass, 0 inconc, 0 fail, 0 error.
MTC@: Number of errors outside test cases: 1
MTC@: Test execution summary: 0 test case was executed. Overall verdict: error
MC@: MTC terminated.
MC@: Shutting down session.
MC@: Shutdown complete.
```


# uftrace dump to `Perfetto`

We can use also `Perfetto` with dumped json file.

```
# alias uftrace_json="$HOME/uftrace/uftrace dump --libmcount-path=$HOME/uftrace --chrome > dump.json"
$ uftrace_json
```
Upload dumped json file from `Open trace file` button.


<img src="https://github.com/namhyung/uftrace/assets/43713967/d39f3c33-20cb-485a-abf5-6725a455fcfa" height="104" width="180" >

- https://ui.perfetto.dev/

- [ttcn3.json](https://github.com/namhyung/uftrace/files/14156394/ttcn3.json)
![image](https://github.com/namhyung/uftrace/assets/43713967/b8f12ab2-e9c9-46dc-b80b-001e88c25816)

# uftrace dump to `trace2html`

We can use shortcut for trace2html.
```
# alias uftrace_dump="( $HOME/uftrace/uftrace dump --libmcount-path=$HOME/uftrace --chrome > dump.json ) && ( $HOME/catapult/tracing/bin/trace2html dump.json )"
$ uftrace_dump
```
- [ttcn3.html](https://github.com/namhyung/uftrace/files/14149341/ttcn3.zip)
![image](https://github.com/namhyung/uftrace/assets/43713967/6a299947-98e7-4de6-935f-be1b925769cd)
