**1. Example code for parallel sort**
```cpp
#include <cstdlib>
#include <ctime>
#include <algorithm>
#include <execution>
#include <vector>

void fill_random(std::vector<int> &v, int n)
{
    srand(static_cast<unsigned>(time(nullptr)));
    for (int i = 0; i < n; i++)
        v.push_back(rand() % n);
}

void sequenced_sort(std::vector<int> v)
{
    std::sort(std::execution::seq, v.begin(), v.end());
}

void parallel_sort(std::vector<int> v)
{
    std::sort(std::execution::par, v.begin(), v.end());
}

void parallel_unsequenced_sort(std::vector<int> v)
{
    std::sort(std::execution::par_unseq, v.begin(), v.end());
}

int main(int argc, char *argv[])
{
    std::vector<int> v;
    int n = 5000;
    if (argc == 2)
        n = std::atoi(argv[1]);

    fill_random(v, n);

    sequenced_sort(v);
    parallel_sort(v);
    parallel_unsequenced_sort(v);
}
```

**2. Install Intel Thread Building Blocks (if needed)**
```
$ echo "deb http://cz.archive.ubuntu.com/ubuntu focal main universe" | sudo tee -a  /etc/apt/sources.list
$ sudo apt update
$ sudo apt install libtbb-dev
```

**3. Build the example (g++-9 or higher required)**
```
$ g++ -pg -g -std=c++17 parallel_sort.cc -ltbb
```

**4. Run the example with uftrace**
```
$ uftrace record -t 10ms ./a.out

$ uftrace report -C sequenced_sort -C parallel_sort -C parallel_unsequenced_sort
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================
  172.376 ms   21.227 ms           1  main
   83.393 ms   83.393 ms           1  sequenced_sort
   36.961 ms   36.500 ms           1  parallel_sort
   30.793 ms   30.793 ms           1  parallel_unsequenced_sort

$ uftrace replay
# DURATION     TID     FUNCTION
            [ 51962] | main() {
  18.818 ms [ 51962] |   fill_random();
            [ 51962] |   sequenced_sort() {
            [ 51962] |     std::sort() {
            [ 51962] |       std::sort() {
            [ 51962] |         __pstl::__internal::__pattern_sort() {
            [ 51962] |           std::sort() {
            [ 51962] |             std::__sort() {
            [ 51962] |               std::__introsort_loop() {
            [ 51962] |                 std::__introsort_loop() {
  24.381 ms [ 51962] |                   std::__introsort_loop();
  31.620 ms [ 51962] |                 } /* std::__introsort_loop */
  40.277 ms [ 51962] |               } /* std::__introsort_loop */
            [ 51962] |               std::__final_insertion_sort() {
  16.075 ms [ 51962] |                 std::__unguarded_insertion_sort();
  16.135 ms [ 51962] |               } /* std::__final_insertion_sort */
  56.414 ms [ 51962] |             } /* std::__sort */
  56.415 ms [ 51962] |           } /* std::sort */
  56.415 ms [ 51962] |         } /* __pstl::__internal::__pattern_sort */
  56.417 ms [ 51962] |       } /* std::sort */
  56.417 ms [ 51962] |     } /* std::sort */
  56.418 ms [ 51962] |   } /* sequenced_sort */
            [ 51962] |   parallel_sort() {
            [ 51962] |     std::sort() {
            [ 51962] |       std::sort() {
            [ 51962] |         __pstl::__internal::__pattern_sort() {
            [ 51962] |           __pstl::__internal::__except_handler() {
            [ 51962] |             __pstl::__internal::__pattern_sort::$_0::operator() {
            [ 51962] |               __pstl::__par_backend::__parallel_stable_sort() {
            [ 51962] |                 tbb::interface7::this_task_arena::isolate() {
            [ 51962] |                   tbb::interface7::internal::isolate_impl() {
            [ 51962] |                     tbb::interface7::internal::isolate_within_arena() {
            [ 51962] |                       tbb::interface7::internal::delegated_function::operator() {
            [ 51962] |                         __pstl::__par_backend::__parallel_stable_sort::$_0::operator() {
            [ 51962] |                           tbb::task::spawn_root_and_wait() {
            [ 51979] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 51979] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 51979] |     std::sort() {
            [ 51979] |       std::__sort() {
            [ 51988] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 51988] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 51988] |     std::sort() {
            [ 51988] |       std::__sort() {
            [ 51984] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 51984] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 51984] |     std::sort() {
            [ 51984] |       std::__sort() {
  10.361 ms [ 51979] |       } /* std::__sort */
  10.364 ms [ 51979] |     } /* std::sort */
  10.365 ms [ 51979] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  10.499 ms [ 51979] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  10.588 ms [ 51988] |       } /* std::__sort */
  10.661 ms [ 51988] |     } /* std::sort */
  10.661 ms [ 51988] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  10.768 ms [ 51988] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  10.571 ms [ 51984] |       } /* std::__sort */
  10.676 ms [ 51984] |     } /* std::sort */
  10.676 ms [ 51984] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  10.780 ms [ 51984] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  25.048 ms [ 51962] |                           } /* tbb::task::spawn_root_and_wait */
  25.058 ms [ 51962] |                         } /* __pstl::__par_backend::__parallel_stable_sort::$_0::operator() */
  25.058 ms [ 51962] |                       } /* tbb::interface7::internal::delegated_function::operator() */
  25.861 ms [ 51962] |                     } /* tbb::interface7::internal::isolate_within_arena */
  25.862 ms [ 51962] |                   } /* tbb::interface7::internal::isolate_impl */
  25.862 ms [ 51962] |                 } /* tbb::interface7::this_task_arena::isolate */
  25.862 ms [ 51962] |               } /* __pstl::__par_backend::__parallel_stable_sort */
  25.863 ms [ 51962] |             } /* __pstl::__internal::__pattern_sort::$_0::operator() */
  25.863 ms [ 51962] |           } /* __pstl::__internal::__except_handler */
  25.863 ms [ 51962] |         } /* __pstl::__internal::__pattern_sort */
  25.864 ms [ 51962] |       } /* std::sort */
  25.864 ms [ 51962] |     } /* std::sort */
  25.865 ms [ 51962] |   } /* parallel_sort */
            [ 51962] |   parallel_unsequenced_sort() {
            [ 51962] |     std::sort() {
            [ 51962] |       std::sort() {
            [ 51962] |         __pstl::__internal::__pattern_sort() {
            [ 51962] |           __pstl::__internal::__except_handler() {
            [ 51962] |             __pstl::__internal::__pattern_sort::$_0::operator() {
            [ 51962] |               __pstl::__par_backend::__parallel_stable_sort() {
            [ 51962] |                 tbb::interface7::this_task_arena::isolate() {
            [ 51962] |                   tbb::interface7::internal::isolate_impl() {
            [ 51962] |                     tbb::interface7::internal::isolate_within_arena() {
            [ 51962] |                       tbb::interface7::internal::delegated_function::operator() {
            [ 51962] |                         __pstl::__par_backend::__parallel_stable_sort::$_0::operator() {
            [ 51962] |                           tbb::task::spawn_root_and_wait() {
            [ 51993] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 51993] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 51993] |     std::sort() {
            [ 51993] |       std::__sort() {
            [ 52000] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 52000] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 52000] |     std::sort() {
            [ 52000] |       std::__sort() {
            [ 52002] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 52020] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 52020] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 52020] |     std::sort() {
            [ 52020] |       std::__sort() {
            [ 52004] | __pstl::__par_backend::__stable_sort_task::execute() {
            [ 52004] |   __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() {
            [ 52004] |     std::sort() {
            [ 52004] |       std::__sort() {
  10.067 ms [ 52002] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  10.933 ms [ 51993] |       } /* std::__sort */
  10.936 ms [ 51993] |     } /* std::sort */
  10.937 ms [ 51993] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  11.073 ms [ 51993] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  10.974 ms [ 52000] |       } /* std::__sort */
  10.977 ms [ 52000] |     } /* std::sort */
  10.978 ms [ 52000] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  11.113 ms [ 52000] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  11.175 ms [ 52020] |       } /* std::__sort */
  11.178 ms [ 52020] |     } /* std::sort */
  11.179 ms [ 52020] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  11.264 ms [ 52020] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  11.653 ms [ 52004] |       } /* std::__sort */
  11.656 ms [ 52004] |     } /* std::sort */
  11.657 ms [ 52004] |   } /* __pstl::__internal::__pattern_sort::$_0::operator()::$_0::operator() */
  11.790 ms [ 52004] | } /* __pstl::__par_backend::__stable_sort_task::execute */
  22.316 ms [ 51962] |                           } /* tbb::task::spawn_root_and_wait */
  22.319 ms [ 51962] |                         } /* __pstl::__par_backend::__parallel_stable_sort::$_0::operator() */
  22.320 ms [ 51962] |                       } /* tbb::interface7::internal::delegated_function::operator() */
  22.320 ms [ 51962] |                     } /* tbb::interface7::internal::isolate_within_arena */
  22.321 ms [ 51962] |                   } /* tbb::interface7::internal::isolate_impl */
  22.321 ms [ 51962] |                 } /* tbb::interface7::this_task_arena::isolate */
  22.321 ms [ 51962] |               } /* __pstl::__par_backend::__parallel_stable_sort */
  22.322 ms [ 51962] |             } /* __pstl::__internal::__pattern_sort::$_0::operator() */
  22.322 ms [ 51962] |           } /* __pstl::__internal::__except_handler */
  22.322 ms [ 51962] |         } /* __pstl::__internal::__pattern_sort */
  22.323 ms [ 51962] |       } /* std::sort */
  22.323 ms [ 51962] |     } /* std::sort */
  22.324 ms [ 51962] |   } /* parallel_unsequenced_sort */
 123.464 ms [ 51962] | } /* main */
```

**5. Visualized trace output of node**
- chrome: https://uftrace.github.io/dump/parallel_sort.html
- flame-graph: https://uftrace.github.io/dump/parallel_sort.svg

[![parallel_sort.svg](https://uftrace.github.io/dump/parallel_sort.svg)](https://uftrace.github.io/dump/parallel_sort.svg)