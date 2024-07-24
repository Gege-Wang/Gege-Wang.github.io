---
layout: single
title: "cmake compile"
date: 2024-07-24
categories: cmake 
---
> 今天要把现在的程序交叉编译到 aarch64-linux-musl- 的环境上所以，要对很多东西进行交叉编译

1. 配置交叉编译环境
如果你是 `x86`  的架构，那么你就可以使用 musl 官方预编译的交叉编译工具链。

```
# download
wget https://musl.cc/aarch64-linux-musl-cross.tgz
wget https://musl.cc/riscv64-linux-musl-cross.tgz
wget https://musl.cc/x86_64-linux-musl-cross.tgz
# install
tar zxf aarch64-linux-musl-cross.tgz
tar zxf riscv64-linux-musl-cross.tgz
tar zxf x86_64-linux-musl-cross.tgz
# exec below command in bash OR add below info in ~/.bashrc
export PATH=`pwd`/x86_64-linux-musl-cross/bin:`pwd`/aarch64-linux-musl-cross/bin:`pwd`/riscv64-linux-musl-cross/bin:$PATH

```
验证安装是否成功
```bash
/desired/path/aarch64-linux-musl-cross/bin/aarch64-linux-musl-gcc --version
```
2. 获取程序的源码
```bash
git clone https://github.com/ysuga/navigation_amcl
```
对于这种具有 cmakeList.txt 的项目，我们可以直接加一个文件

```bash
# aarch64-musl-toolchain.cmake

SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_PROCESSOR aarch64)

SET(CMAKE_C_COMPILER /desired/path/aarch64-linux-musl-cross/bin/aarch64-linux-musl-gcc)
SET(CMAKE_CXX_COMPILER /desired/path/aarch64-linux-musl-cross/bin/aarch64-linux-musl-g++)

SET(CMAKE_FIND_ROOT_PATH /desired/path/aarch64-linux-musl-cross/aarch64-linux-musl)

SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```
然后开始编译项目

```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=../aarch64-musl-toolchain.cmake ..
make
```