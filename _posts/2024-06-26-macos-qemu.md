---
layout: single
title:  "如何在 macos 配置 qemu 环境"
date:  2024-06-26
categories: docker dora
---

> 在 macos m2 上配置 riscv qemu 环境，并且配置 GDB.

1. 安装 qemu

```bash
brew install qemu
```

2. 安装 gdb

前往清华镜像源下载 GDB 的源码包：[清华镜像](https://mirrors.tuna.tsinghua.edu.cn/gnu/gdb/?C=M&O=D)

```bash
mkdir build
cd build
../configure --prefix=/usr/local --target=riscv64-unknown-elf
```

你会遇到下面的问题：

```shell
configure: error: Building GDB requires GMP 4.2 , and MPFR 3.1.0 . Try the --with-gmp and/or --with-mpfr options to specify their locations.  If you obtained GMP and/or MPFR from a vendor
```

```shell
brew install gmp mpfr
# 找到这两个库的安装路径
brew info gmp
brew info mpfr
# 重新配置
../configure --prefix=/usr/local/ --target=riscv64-unknown-elf --with-gmp=/opt/homebrew/Cellar/gmp/6.3.0/ --with-mpfr=/opt/homebrew/Cellar/mpfr/4.2.1/
# 编译
# macOS
make -j$(sysctl -n hw.ncpu)

sudo make install
```

安装 [gdb-dashboard](https://github.com/cyrus-and/gdb-dashboard/) 插件，优化 debug 体验

```bash
wget -P ~ https://git.io/.gdbinit
```
