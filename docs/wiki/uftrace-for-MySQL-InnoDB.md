This document is written by [Kyungmin Kim](https://github.com/lufovic77).

**1. Download the MySQL server source code and extract (5.7.24 stable for my case)**
```
$ wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24.tar.gz
$ tar -xvf mysql-5.7.24.tar.gz
$ cd mysql-5.7.24
```
**2. Solve Dependencies & Download Boost C++ libraries**
```
$ sudo apt-get install gcc g++ libncurses5-dev libxml2-dev openssl libssl-dev curl libcurl4-openssl-dev libjpeg-dev libpng-dev libfreetype6-dev libsasl2-dev autoconf libncurses5-dev
$ cd ../
$ wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
$ tar -xvf boost_1_59_0.tar.gz
```
You don't have to other stuffs with boost. Just extract it.  <br/> 
**3. Install cmake**
```
$ sudo apt-get install cmake
```
**4. Cmake**
```
$ cd mysql-5.7.24
$ sudo cmake \
-DCMAKE_INSTALL_PREFIX=/your/workspace/ \
-DMYSQL_DATADIR=/your/data/directory/ \
-DENABLED_LOCAL_INFILE=1 \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DSYSCONFDIR=/your/configuration/directory/ \
-DWITH_EXTRA_CHARSETS=all \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/where/boost/is/downloaded/ \ #till here, common options when compiling MySQL
-DCFLAGS="-pg -g"\
-DCXXFLAGS="-pg -g"
```
-pg option for uftrace and -g for debugging (arguments, return types)  <br/> 
**5. Make**
```
$ make
$ make test
$ sudo make install
```
**6. Enter directory specified in DCMAKE_INSTALL_PREFIX in step 4**
```
$ cd /your/workspace/bin
```
**7.Initialize MySQL**
```
$ ./mysqld --initialize
```
_Before going to step 8, please back up your data. Database crash sometimes happenes when using uftrace._ <br/>
**8. Run MySQL server**
```
$ uftrace record ./mysqld --defaults-file=/path/to/your/my.cnf --skip-grant-tables
```
Give any options here you want.   <br/>  
**9. Enter MySQL and do something with MySQL. UDATE, INSERT ...etc**
```
$ ./mysql -u root
```
If you have any error with running as root, add 'user = root' to your configuration file under [mysqld].  <br/>
**10. Shutdown MySQL server**
```
$ ./mysqladmin shutdown
```
Shutdown before going further.  <br/>
**11. Replay with uftrace**
```
$ uftrace replay uftrace.data/ > record.txt 
```
You can also give options here. 


