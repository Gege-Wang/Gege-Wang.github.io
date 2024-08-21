---
layout: single
title: "ohos-alpine-cartographer"
date: 2024-08-02
categories: ohos, alpine, cartographer
---

## 编译动态库
```bash
# 使用 cmake 时
cmake -DBUILD_SHARED_LIBS=ON ..
# 运行 configure 脚本时，需要传递 --enable-shared 选项来指示编译动态库。使用以下命令配置编译选项
./configure --enable-shared

../configure --enable-shared --disable-static
```

## 需要的依赖库
```bash
apk add build-base
apk add gcc g++ musl-dev linux-headers
apk add glog
apk add boost boost-dev 
apk add suitesparse-dev  7.7.0
apk add lua5.2 lua5.2-dev   必须是 >=5.2 我用的5.2
apk add cairo cairo-dev

```

## 需要自己编的东西
```bash
ceres-solver 2.0.0(必须找到 suitesparse 的 组件才能编译 否则找不到 suitesparse 组件)
abseil-cpp  官方有脚本 版本已固定
protobuf3 官方有脚本 版本已固定
suitesparse 5.8.1 这里是因为有一个 suitesparseQR 的函数在不同版本里面不一样
```

## 找不到 AllFiles.cmake
```bash
  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Build type: Release
CMake Error at cmake/functions.cmake:140 (include):
  include could not find requested file:

    /root/cartographer/build/AllFiles.cmake
Call Stack (most recent call first):
  CMakeLists.txt:29 (google_initialize_cartographer_project)


-- Found AMD headers in: /usr/include/suitesparse
```
因为 alpine 里面没有 find 命令。这个错误表明 find 命令遇到了一个不被识别的选项 -iwholename。这是因为我们使用的是 BusyBox 的 find 命令，而 BusyBox 的 find 命令实现较为简化，不支持所有 GNU find 的选项。
```bash
 apk add bash
 apk add findutils
```

## 需要进行的修改
```bash
In file included from /root/cartographer/cartographer/common/thread_pool.h:28,
                 from /root/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:26,
                 from /root/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.cc:17:
/root/cartographer/cartographer/common/task.h:41:18: error: expected ';' at end of member declaration
   41 |   State GetState() LOCKS_EXCLUDED(mutex_);
      |                  ^
      |                   ;
/root/cartographer/cartographer/common/task.h:41:35: error: 'mutex_' has not been declared
   41 |   State GetState() LOCKS_EXCLUDED(mutex_);
      |                                   ^~~~~~
/root/cartographer/cartographer/common/task.h:44:45: error: expected ';' at end of member declaration
   44 |   void SetWorkItem(const WorkItem& work_item) LOCKS_EXCLUDED(mutex_);
      |                                             ^
      |                                              ;
/root/cartographer/cartographer/common/task.h:44:62: error: 'mutex_' has not been declared
   44 |   void SetWorkItem(const WorkItem& work_item) LOCKS_EXCLUDED(mutex_);
      |                                                              ^~~~~~
/root/cartographer/cartographer/common/task.h:44:47: error: 'int cartographer::common::Task::LOCKS_EXCLUDED(int)' cannot be overloaded with 'int cartographer::common::Task::LOCKS_EXCLUDED(int)'
   44 |   void SetWorkItem(const WorkItem& work_item) LOCKS_EXCLUDED(mutex_);
      |                                               ^~~~~~~~~~~~~~
/root/cartographer/cartographer/common/task.h:41:20: note: previous declaration 'int cartographer::common::Task::LOCKS_EXCLUDED(int)'
   41 |   State GetState() LOCKS_EXCLUDED(mutex_);
      |                    ^~~~~~~~~~~~~~
/root/cartographer/cartographer/common/task.h:48:52: error: expected ';' at end of member declaration
   48 |   void AddDependency(std::weak_ptr<Task> dependency) LOCKS_EXCLUDED(mutex_);
      |                                                    ^
      |                                                     ;
/root/cartographer/cartographer/common/task.h:48:69: error: 'mutex_' has not been declared
   48 |   void AddDependency(std::weak_ptr<Task> dependency) LOCKS_EXCLUDED(mutex_);
      |                                                                     ^~~~~~
/root/cartographer/cartographer/common/task.h:48:54: error: 'int cartographer::common::Task::LOCKS_EXCLUDED(int)' cannot be overloaded with 'int cartographer::common::Task::LOCKS_EXCLUDED(int)'
   48 |   void AddDependency(std::weak_ptr<Task> dependency) LOCKS_EXCLUDED(mutex_);
      |                                                      ^~~~~~~~~~~~~~
/root/cartographer/cartographer/common/task.h:41:20: note: previous declaration 'int cartographer::common::Task::LOCKS_EXCLUDED(int)'
   41 |   State GetState() LOCKS_EXCLUDED(mutex_);
      |                    ^~~~~~~~~~~~~~
```
这是版本不匹配的问题
这是因为没有用官方的下载脚本安装 abseil-cpp 导致版本不匹配。
切换到 相应分支 abseil-cpp 编译出现错误
```bash
[  7%] Building CXX object absl/base/CMakeFiles/malloc_internal.dir/internal/low_level_alloc.cc.o
In file included from /root/abseil-cpp/absl/base/internal/low_level_alloc.cc:26:
/root/abseil-cpp/absl/base/internal/direct_mmap.h:75:25: error: 'off64_t' has not been declared
   75 |                         off64_t offset) noexcept {
      |                         ^~~~~~~
make[2]: *** [absl/base/CMakeFiles/malloc_internal.dir/build.make:76: absl/base/CMakeFiles/malloc_internal.dir/internal/low_level_alloc.cc.o] Error 1
make[1]: *** [CMakeFiles/Makefile2:624: absl/base/CMakeFiles/malloc_internal.dir/all] Error 2
make: *** [Makefile:136: all] Error 2
```
解决办法是修改 `abseil-cpp/absl/base/internal/direct_mmap.h` 将off64_t 改成 off_t

```bash
 45 | ABSL_INTERNAL_CONVERSION_CHARS_EXPAND_(ABSL_INTERNAL_CHAR_SET_CASE, )
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/root/cartographer/scripts/abseil-cpp/absl/strings/internal/str_format/extension.cc:44:13: error: 'FormatConversionCharSet' does not name a type; did you mean 'FormatConversionCharInternal'?
   44 |   constexpr FormatConversionCharSet FormatConversionCharSetInternal::c;
      |             ^~~~~~~~~~~~~~~~~~~~~~~
/root/cartographer/scripts/abseil-cpp/absl/strings/internal/str_format/extension.h:167:33: note: in expansion of macro 'ABSL_INTERNAL_CHAR_SET_CASE'
  167 |   X_VAL(g) X_SEP X_VAL(G) X_SEP X_VAL(a) X_SEP X_VAL(A) X_SEP \
      |                                 ^~~~~
/root/cartographer/scripts/abseil-cpp/absl/strings/internal/str_format/extension.cc:45:1: note: in expansion of macro 'ABSL_INTERNAL_CONVERSION_CHARS_EXPAND_'
   45 | ABSL_INTERNAL_CONVERSION_CHARS_EXPAND_(ABSL_INTERNAL_CHAR_SET_CASE, )
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~. 
```
参考这条进行修改[commit](https://github.com/abseil/abseil-cpp/commit/b957f0ccd00481cd4fd663d8320aa02ae0564f18#diff-bfcda9be15511d0da830b4d46d2de4221d662d1a1dace2bab2ea15600e31d728)

## GCC 版本以及编译选项
在编译 abseil-cpp 时，我们使用的是 c++17 标准，但是在编译 cartographer 时，固定的是 c++11 标准，所以需要修改 cartographer 的编译选项，将 CXX_STANDARD 改为 17。

```bash
root/cartographer/cartographer/mapping/internal/range_data_collator_test.cc:96:62:
/usr/include/eigen3/Eigen/src/Core/PlainObjectBase.h:512:17: error: 'empty_data.cartographer::sensor::TimedPointCloudData::origin.Eigen::Matrix<float, 3, 1, 0, 3, 1>::<unnamed>.Eigen::PlainObjectBase<Eigen::Matrix<float, 3, 1> >::m_storage' may be used uninitialized [-Werror=maybe-uninitialized]
  512 |       : Base(), m_storage(other.m_storage) { }
      |                 ^~~~~~~~~~~~~~~~~~~~~~~~~~
/root/cartographer/cartographer/mapping/internal/range_data_collator_test.cc: In member function 'virtual void cartographer::mapping::{anonymous}::RangeDataCollatorTest_SingleSensorEmptyData_Test::TestBody()':
/root/cartographer/cartographer/mapping/internal/range_data_collator_test.cc:94:31: note: 'empty_data' declared here
   94 |   sensor::TimedPointCloudData empty_data{
      |                               ^~~~~~~~~~
cc1plus: some warnings being treated as errors
```
由于c++17标准的检查更加严格，不允许有未初始化的变量，所以我们需要修改编译选项，将未初始化视为 warning。
```bash
# CMakeLists.txt
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-maybe-uninitialized")
```