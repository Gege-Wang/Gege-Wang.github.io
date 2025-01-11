---
layout: single
title: "ubuntu 22.04 安装 vmware"
date: 2025-1-8
categories: ubuntu
---

## 1. 下载 vmware workstation pro
还是在该死的 broadcom 网站，我发誓这辈子和它势不两立，每次都找不到在哪下载。
登录之后，点击右上角的 cloud foundation. 找到我的下载，有的资源点下载的云朵标志会提示 `not entity` 这种就是下载不了，有的提示 `screening required` 或者 `https`，这种点击左上角同意协议就能下载了。

## 2. 通过 VNC 连接 ubuntu GUI 安装 vmware 
需要在 root 用户下才能运行，这一点要特别注意。 具体可以参考这篇 [install](https://forums.opensuse.org/t/vmware-workstation-15-1-invalid-mit-magic-cookie-1/136383)。   

开启 VNC 的用户必须和开启 vmware 的用户是一个，否则会出现闪退的问题。经过检查已经开启系统虚拟化，所以就是权限问题。

```bash
sudo -i
cp /home/jsberrios/.Xauthority /root/; export DISPLAY=:0.0
sh ./VMware-Workstation-Full-15.1.0-13591040.x86_64.bundle
/usr/bin/vmware
```
## 配置外网SSH连接内网虚拟机
具体可以看参考文献。最终是用的 tailscale，免费实现异地组网，并且能够完成 VNC 连接到功能。

# 参考文献
[【网络基础】分享几款免费实用的国产内网穿透工具很全哦](https://www.cnblogs.com/kukuxjx/p/17595471.html)
[VNC远程桌面怎么使用？简单实用的外网访问内网主机教程](http://218.244.147.32/Pages_74_1147.jsp)
[ubuntu GUI 下 vmware网络配置打不开？vmware-cfg权限问题](https://bbs.archlinux.org/viewtopic.php?id=283551)
[外网SSH访问内网LINUX-非网站应用映射方法](http://www.nat123.com/Pages_23_539.jsp)
[外网SSH连接内网Windows上linux虚拟机](https://xumingmingming.github.io/2019/06/28/linux/wai-wang-ssh-lian-jie-nei-wang-windows-shang-linux-xu-ni-ji/)
[自己用 docker 搭建 frp 服务器](https://github.com/frank-lam/lanproxy-nat)
[利用Tailscale+RustDesk实现内网穿透以及远程访问](http://bbs.keinsci.com/thread-43175-1-1.html)