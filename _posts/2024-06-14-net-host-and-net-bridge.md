---
layout: single
title:  "虚拟机和 docker 的 网络模式"
date:  2024-06-13
categories: docker 
---
> 写这篇文章是因为我想要把有两个在局域网中的设备相互通信，并且其中一台设备有摄像头。一种方式使用 docker, 就是 miniconda 装的时候缺少库。另一种方式是使用虚拟机。

## 虚拟机

1. 桥接模式
使用桥接方式相当于虚拟机直接作为一台独立的设备，可以从网络中的 DHCP 获取动态 IP， 也可以配置静态 IP， 总之虚拟机可以正常像局域网中的其他设备一样。物理网络中的设备也可以直接访问虚拟机。
2. NAT 模式
虚拟机通过主机的网络访问外部网络，虚拟机的网络是一个虚拟的私有网络。也就是说如果我的电脑在局域网当中，它拥有一个私有 IP，其实我的电脑是要通过 NAT 才能访问外网的，所有局域网中的设备共同使用一个公网地址。现在我的电脑里有两台虚拟机都是 NAT 的方式，那么这两台虚拟机有自己的私有 IP， 但是对外都通过我的电脑的私有 IP 作为公有 IP。这相当于虚拟机套娃加上网络也套娃的方式。

> 比较形象的说明就是，桥接模式下，主机和虚拟机在网络上是兄弟关系，NAT模式下，主机和虚拟机网络上是父子关系。

## 容器的四种网络模式

TODO
