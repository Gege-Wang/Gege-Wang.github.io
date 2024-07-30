---
layout: single
title: "cmake compile"
date: 2024-07-26
categories: cmake 
---
## 交叉编译 gmp 库
```bash
wget https://gmplib.org/download/gmp/gmp-6.2.1.tar.xz
tar -xvf gmp-6.2.1.tar.xz
cd gmp-6.2.1
# 在编译 mpfr 库的时候发现 gmp 不可重定位，所以需要加上这个编译参数
export CFLAGS="-fPIC"
export CXXFLAGS="-fPIC"
# 默认会放到 /usr/local/lib 下
./configure --host=aarch64-linux-musl
make
make install
```

重新编译依赖 GMP 的库: MPFR
```bash
# 这两句是为了防止找不到 gmp.h
export CPPFLAGS="-I/usr/local/include"
export LDFLAGS="-L/usr/local/lib"
wget https://www.mpfr.org/mpfr-current/mpfr-4.2.1.tar.gz
cd mpfr-4.2.1
make clean
./configure --host=aarch64-linux-musl --with-gmp=/usr/local
make
make install
```

## 验证 GMP 和 MPFR 库的安装
在编译和安装完成之后，你可以验证 GMP 和 MPFR 库是否正确安装。运行以下命令来检查库文件和头文件是否存在：
```bash
ls /usr/local/lib/libgmp.a
ls /usr/local/lib/libmpfr.a
ls /usr/local/include/gmp.h
ls /usr/local/include/mpfr.h
```
## 设置环境变量
```bash
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```
或者，你可以将 LD_LIBRARY_PATH 添加到你的 .bashrc 或 .profile 文件中以使其永久生效。  
类似于这样：
```bash
export LD_LIBRARY_PATH=/usr/local/lib:/lib:/lib/aarch64-linux-gnu/:/root/aarch64-linux-musl-native/lib:$LD_LIBRARY_PATH
```
## Fortran 编译
在编译 SuiteParse 的时候可能需要用到 fortran 编译器，在交叉编译环境下，我们使用 `aarch64-linux-musl-native/bin/gfortran` 的编译器。
```
Fortran:          /root/aarch64-linux-musl-native/bin/gfortran
```

## docker 安装包空间不足
```bash
root@396af4940636:~# apt install m4
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libsigsegv2
Suggested packages:
  m4-doc
The following NEW packages will be installed:
  libsigsegv2 m4
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 211 kB of archives.
After this operation, 388 kB of additional disk space will be used.
E: You don't have enough free space in /var/cache/apt/archives/.
```

```bash
root@396af4940636:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          59G   56G     0 100% /
tmpfs            64M     0   64M   0% /dev
shm              64M     0   64M   0% /dev/shm
/dev/vda1        59G   56G     0 100% /etc/hosts
tmpfs           2.0G     0  2.0G   0% /sys/firmware
```
查看容器当前 root 下的文件空间并不大
```bash
root@396af4940636:~# du -sh *
465M	OpenBLAS
520M	SuiteSparse
234M	aarch64-linux-musl-native
1.3G	boost_1_85_0
18M	g2o-20200410_git
44M	g2o-20201223_git
20M	gmp-6.2.1
2.0M	gmp-6.2.1.tar.xz
12K	hello
4.0K	hello.c
1.2M	ld-musl-aarch64.so.1
40M	teb_local_planner_no_ros
```
这种情况是 docker 总的存储空间不足，需要把不需要的 docker 容器删掉一些
```bash
root@396af4940636:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          59G   36G   20G  65% /
tmpfs            64M     0   64M   0% /dev
shm              64M     0   64M   0% /dev/shm
/dev/vda1        59G   36G   20G  65% /etc/hosts
tmpfs           2.0G     0  2.0G   0% /sys/firmware
```

## 静态库还是动态库
```bash
gcc -o my_program my_program.o -L/usr/local/lib -lgmp -lmpfr
```
当使用 -L 和 -l 选项链接库时，链接器会根据以下顺序查找库文件：

查看指定路径（例如 -L/usr/local/lib）下是否有动态库文件（libgmp.so 和 libmpfr.so）。如果动态库不存在，查看指定路径下是否有静态库文件（libgmp.a 和 libmpfr.a）。如果两者都存在，通常情况下链接器会优先选择动态库。

如果你明确希望链接静态库，可以在编译时使用以下方式指定：
显式指定静态库路径：
```bash
gcc -o my_program my_program.o /usr/local/lib/libgmp.a /usr/local/lib/libmpfr.a
```
使用链接器选项强制链接静态库：
```bash
gcc -o my_program my_program.o -L/usr/local/lib -Wl,-Bstatic -lgmp -lmpfr -Wl,-Bdynamic
```
这里的 `-Wl,-Bstatic` 强制链接器使用静态库，而 `-Wl,-Bdynamic` 之后的库（如果有）则恢复到默认的动态链接行为。


## cmake 设置
```bash
CMake Error at /usr/share/cmake-3.22/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find BLAS (missing: BLAS_LIBRARIES)
Call Stack (most recent call first):
  /usr/share/cmake-3.22/Modules/FindPackageHandleStandardArgs.cmake:594 (_FPHSA_FAILURE_MESSAGE)
  /usr/share/cmake-3.22/Modules/FindBLAS.cmake:1337 (find_package_handle_standard_args)
  SuiteSparse_config/cmake_modules/SuiteSparseBLAS.cmake:244 (find_package)
  SuiteSparse_config/CMakeLists.txt:123 (include)
```
```bash
which cc
/usr/bin/cc 
```
把 `cc` 软链接到 `/root/aarch64-linux-musl-native/bin/gcc`

## LAPACK 动态库安装
[参考](https://blog.csdn.net/qq_32115939/article/details/109183998)
之前试了很多 SHARED = 1、 NO_STATIC 之类的都不管用，下面这个是对的
```bash
cmake .. -DCMAKE_BUILD_TYPE=RELEASE -DBUILD_SHARED_LIBS=ON
```

## 一些库
```bash
## wget 
wget https://ftp.gnu.org/gnu/wget/wget-1.21.2.tar.gz

./configure --host=aarch64-linux-musl --with-included-libtasn1 --with-included-unistring --without-p11-kit
## apt
git clone https://salsa.debian.org/apt-team/apt.git
wget https://ftp.gnu.org/gnu/libiconv/libiconv-1.17.tar.gz
```

```bash
configure: WARNING: unrecognized options: --with-gmp
configure: summary of build options:

  Version:           nettle 3.6
  Host type:         aarch64-unknown-linux-musl
  ABI:               standard
  Assembly files:    none
  Install prefix:    /usr/local
  Library directory: ${exec_prefix}/lib
  Compiler:          /root/aarch64-linux-musl-native/bin/gcc
  Static libraries:  yes
  Shared libraries:  yes
  Public key crypto: no
  Using mini-gmp:    no
  Documentation:     no

root@396af4940636:~/nettle-3.6# ./configure --host=aarch64-linux-musl --with-gmp=/usr/local/lib
```
在 nettle 的 configure 脚本中，--with-gmp 选项被忽略了。nettle 的配置选项可能不支持 --with-gmp，而是通过 PKG_CONFIG_PATH 和 CPPFLAGS 等环境变量来指定 GMP 库的位置。
```bash
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
export CPPFLAGS="-I/usr/local/include"
export LDFLAGS="-L/usr/local/lib"
```
这确保 pkg-config、编译器和链接器能够找到 GMP 的头文件和库文件。

运行 configure 脚本

现在运行 configure 脚本时，不需要 --with-gmp 选项，使用下面的命令：
```bash
./configure --host=aarch64-linux-musl
```
验证
```bash
pkg-config --cflags --libs nettle
```

为什么 wget 原来的还是可以运行，但是 apt 原来的就不能运行了。我不理解为什么？？？