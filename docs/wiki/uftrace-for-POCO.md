**Getting Started With The POCO C++ Libraries**
- https://pocoproject.org/docs/00200-GettingStarted.html

**1. download prerequisites**
```
$ sudo apt-get install openssl libssl-dev
$ sudo apt-get install libiodbc2 libiodbc2-dev
```
**2. download poco**
```
$ git clone https://github.com/pocoproject/poco.git
$ cd poco
$ git checkout poco-1.7.5-release
```
**3. configure and build poco with `-pg`**
```
$ mkdir cmake_build
$ cd cmake_build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/usr \
           -DCMAKE_C_FLAGS="-g -pg -fno-omit-frame-pointer" \
           -DCMAKE_CXX_FLAGS="-g -pg -fno-omit-frame-pointer"
$ make
$ make install
```
**4. test program**
```cpp
$ cat poco-test.cpp
#include "Poco/BasicEvent.h"
#include "Poco/Delegate.h"
#include <iostream>

using Poco::BasicEvent;
using Poco::Delegate;

class Source
{
public:
    BasicEvent<int> theEvent;

    void fireEvent(int n)
    {
        theEvent(this, n);
    }
};

class Target
{
public:
    void onEvent(const void* pSender, int& arg)
    {
        std::cout << "onEvent: " << arg << std::endl;
    }
};

int main(int argc, char** argv)
{
    Source source;
    Target target;

    source.theEvent += Delegate<Target, int>(
        &target, &Target::onEvent);

    source.fireEvent(42);

    source.theEvent -= Delegate<Target, int>(
        &target, &Target::onEvent);

    return 0;
}
```
**5. compile a poco test program**
```
$ g++ -pg -g -I$HOME/usr/include poco-test.cpp -L$HOME/usr/lib -lPocoFoundation

$ ./a.out
onEvent: 42
```
**6. run uftrace record and replay**
```
$ uftrace -D 2 ./a.out
onEvent: 42
# DURATION     TID     FUNCTION
            [ 22588] | _GLOBAL__sub_I_main() {
   6.700 us [ 22588] |   __static_initialization_and_destruction_0();
   9.453 us [ 22588] | } /* _GLOBAL__sub_I_main */
            [ 22588] | main() {
   5.527 us [ 22588] |   Source::Source();
   2.147 us [ 22588] |   Poco::Delegate::Delegate();
  16.177 us [ 22588] |   Poco::AbstractEvent::operator+=();
   5.197 us [ 22588] |   Poco::Delegate::~Delegate();
  81.363 us [ 22588] |   Source::fireEvent();
   0.906 us [ 22588] |   Poco::Delegate::Delegate();
   8.780 us [ 22588] |   Poco::AbstractEvent::operator-=();
   0.553 us [ 22588] |   Poco::Delegate::~Delegate();
   3.910 us [ 22588] |   Source::~Source();
 128.059 us [ 22588] | } /* main */
   1.800 us [ 22588] | std::ios_base::Init::~Init();
```