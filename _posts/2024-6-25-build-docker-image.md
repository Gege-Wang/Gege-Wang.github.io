---
layout: single
title:  "如何将当前容器打包成镜像并提交到 dockerhub"
date:  2024-06-25
categories: docker dora
---

> 这篇文章的原因是，在测试 dora 的分布式工作目录的时候需要使用 docker，所以我在自己的本地 docker 中把环境搭建好，直接传给我的 reviewer, 方便他快速测试。

创建 docker 镜像有两种方法：   
- 将本地容器打包， 使用 `docker commit`, 直接构建镜像
- 使用 Dockerfile 脚本构建镜像。

我这里使用的是**第一种**，因为我使用第二种的时候拉取镜像还有执行命令的时候总是遇到很多问题。

### 1. 拉取一个基础操作系统镜像

我使用的是 ubuntu:22.04。在 Mac 系统上会自动拉取 Arm 架构的镜像。
```shell
docker pull ubuntu:22.04
```

### 2. 创建容器

```shell
# 启动 02_ubuntu 容器
docker run -dt --name 02_ubuntu ubuntu:22.04
# 进入交互式环境
docker exec -it 02_ubuntu /bin/bash
# 查看 docker ip 地址
hostname -i
# 查看两个容器是否能 ping 通
apt-get update
apt-get install iputils-ping
apt-get install curl
apt-get install build-essential
apt-get install git
## 安装 rust 
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
## dora binary
git clone https://github.com/dora-rs/dora.git
cd dora
cargo build --release -p dora-cli
PATH=$PATH:$(pwd)/target/release
## Python API
maturin develop -m apis/python/node/Cargo.toml
dora --help
```

### 3. 提交镜像

```shell
## 退出容器
exit
## 将容器打包成镜像
docker login ## 登录到你自己的 dockerhub 账号
docker ps -a ## 查看自己的 docker container, 找到想要打包的容器的 id
## docker commit -a author_name -m message container_id image_name:tag
docker commit -a "miyamo505" -m "dora distributed" 28e4bbb4da60 dora-distributed:v1.0
## docker images 可以查看自己的刚才打包出来的镜像
## docker tag image_id image_name:tag 给刚才的镜像打上 tag
docker tag 0245fbfafabc miyamo505/dora-distributed:v1.0
## 把做好的镜像 push 到远程仓库
docker push miyamo505/dora-ditributed:v1.0
```

 Ok，镜像提交成功了。可以从远程仓库拉取镜像并且测试。
