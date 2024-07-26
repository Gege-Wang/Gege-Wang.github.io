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

## 使用 docker 配置 musl 跨编译环境
1. 拉取 docker 镜像 
```bash
docker run -dt --name ohosmusl ubuntu:22.04 bash
docker exec -it ohosmusl /bin/bash
```
2. 获取 `ld-musl-aarch64.so.1`
```bash
hdc file recv /lib/ld-musl-aarch64.so.1 .
docker cp  ohosmusl:/root
```
3. 配置 `docker  aarch64-linux-musl-native` 环境
> 做这件事情主要是因为 `aarch64-linux-musl-native` 的里面的 `ld-musl-aarch64.so.1` 会软链接到 `/lib/libc.so` ，但是在 `docker ubuntu` 环境中并没有这个 `/lib/libc.so` 文件。
```bash
wget http://musl.cc/aarch64-linux-musl-native.tgz
tar zxf aarch64-linux-musl-native.tgz
```
这时候我们的 docker 环境中并没有 `gnu-gcc` 所以我们可以直接安装 `musl` 的 `gcc`，这样我们的系统就很干净
我们需要在 `~/.bashrc` 中添加环境变量，以至于我们能够找到 musl 的工具链
```bash 
export PATH=/root/aarch64-linux-musl-native/bin:$PATH
export LD_LIBRARY_PATH=/root/aarch64-linux-musl-native/lib:/lib:/usr/lib:$LD_LIBRARY_PATH
```
但是我们有一个非常棘手的问题，/root/aarch64-linux-musl-native/lib 里有一个 ld-musl-aarch64.so.1 是软链接到 /lib/libc.so ，但是 docker ubuntu 环境中并没有这个 /lib/libc.so 文件。
所以我们直接从外面把这个文件拷贝过来
```bash
chmod +x ld-musl-aarch64.so.1
cp ld-musl-aarch64.so.1 /usr/lib/
cp ld-musl-aarch64.so.1 /root/aarch64-linux-musl-native/lib
```
然后我们就可以写一个小的 c 程序验证一下。
这里有一些好用的命令：
```bash
file hello
ldd hello
ls -al /lib
```
NOTE: 做这些之后 `apt install` 会出现
```bash
root@396af4940636:~# apt
Segmentation fault
```