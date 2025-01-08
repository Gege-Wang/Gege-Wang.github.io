<!--
 * @Author: Gege-Wang 2891067867@qq.com
 * @Date: 2025-01-08 22:37:56
 * @LastEditors: Gege-Wang 2891067867@qq.com
 * @LastEditTime: 2025-01-08 22:45:59
 * @FilePath: /Myblog/_posts/2025-01-08-ubuntu-vmware.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
---
layout: single
title: "ubuntu 22.04 安装 vmware"
date: 2025-1-8
categories: ubuntu
---

## 1. 下载 vmware workstation pro
还是在该死的 broadcom 网站，我发誓这辈子和它势不两立，每次都找不到在哪下载。
登录之后，点击右上角的 cloud foundation. 找到我的下载，有的资源点下载的云朵标志会提示 `not entity` 这种就是下载不了，有的提示 `screening required`，这种点击左上角同意协议就能下载了。

## 2. 通过 VNC 连接 ubuntu GUI 安装 vmware 
需要在 root 用户下才能运行，这一点要特别注意。 具体可以参考这篇 [install](https://forums.opensuse.org/t/vmware-workstation-15-1-invalid-mit-magic-cookie-1/136383)。   

如果不在 root 用户执行，会碰到闪退的问题，经过检查已经开启系统虚拟化，所以就是权限问题。

```bash
sudo -i
cp /home/jsberrios/.Xauthority /root/; export DISPLAY=:0.0
sh ./VMware-Workstation-Full-15.1.0-13591040.x86_64.bundle
/usr/bin/vmware
```