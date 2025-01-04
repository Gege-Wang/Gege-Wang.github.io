---
layout: single
title:  "docker 换源拉取镜像"
date:  2024-06-13
categories: docker 
---

> 写这篇文章的原因是想要拉几个 docker 镜像做实验，国内的 docker 镜像源因为各种各样的原因不能拉取镜像了，另一方面，对于 docker hub, 魔法似乎也失灵了。


## 添加国内镜像源
`vim /etc/docker/daemon.json` 添加以下内容:

```
{ 
  "registry-mirrors" : 
    [ 
      "https://docker.m.daocloud.io", 
      "https://noohub.ru", 
      "https://huecker.io",
      "https://dockerhub.timeweb.cloud",
      "https://hub.uuuadc.top",
      "https://docker.anyhub.us.kg",
      "https://dockerhub.jobcher.com",
      "https://dockerhub.icu",
      "https://docker.ckyl.me",
      "https://docker.awsl9527.cn"
    ] 
}
```
添加完之后保存可能会出现 `"/etc/docker/daemon.json" E212: Can't open file for writing` 的错误， 这个时候 `sudo` 没用的话就需要先自己创建文件夹：    

```
mkdir -p /etc/docker
vim /etc/docker/daemon.json
```

做完之后，保存重启 docker daemon，ok, 愉快的拉取镜像。

> 也不知道这些个镜像源啥时候也没了，那就凉凉了，希望网络环境好起来。

## docker 命令都要添加 sudo 的解决方案
1. 更改通信文件权限
```bash
❯ docker ps
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.46/containers/json": dial unix /var/run/docker.sock: connect: permission denied
❯ sudo chmod 777 /var/run/docker.sock
❯ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
2. 添加用户组

## docker ubuntu22.04 在 MacOS 上的问题
> 起因是想在 docker 的 ubuntu 镜像上装 miniconda

```
root@28e4bbb4da60:~# bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
PREFIX=/root/miniconda3
Unpacking payload ...
qemu-x86_64: Could not open '/lib64/ld-linux-x86-64.so.2': No such file or directory
```
会出现这个问题，待解决。