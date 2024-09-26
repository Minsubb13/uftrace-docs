This document is written by [Minchul Kang](https://github.com/kangtegong)


lcov is a code coverage tool for linux.  


  
**1. Download lcov**
```
$ sudo apt-get install lcov
```
or 
```
$ git clone https://github.com/linux-test-project/lcov.git
```


**2. Build lcov**
```
$ make clean
$ make COVERAGE=1
```
if you want to reset coverage, you can use command below:
```
$ make reset-coverage
```

**3. compile test code with `-pg`option**
```
$ gcc -pg -o t-abc tests/s-abc.c
```

**4. record the object data with uftrace**

record t-abc file with uftrace.

```
$ ./uftrace record --libmcount-path=. t-abc
```
Note that as we want to load necessary internal libraries from this path 
for testing purpose, `-L` option is given.

**5. run script for lcov**

Command below will generate `coverage.html` file.

```
./misc/run-lcov.sh
```

**6. Check the generated html**
```
$ cd coverage.html
```

**7. Run a web server**

In this case, we used SimpleHTTPServer as a simple web server
with port number 18080.

```
$ python -m SimpleHTTPServer 18080
```

--------------------------------

you can get some data sample in link down below:  
open index.html in [coverage.html.zip](https://github.com/kosslab-kr/uftrace/files/3746699/coverage.html.zip)

<img src="https://user-images.githubusercontent.com/47781146/67145686-41040000-f2be-11e9-9d6e-60c83453782f.png" alt = "lcov screenshot" width ="80%">


