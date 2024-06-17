---
layout: single
title:  "MacOS M2 安装 vmware fusion"
date:  2024-06-13
---

> 为什么要写这个玩意儿，纯粹是因为 vmware 的 broadcom 有点难以琢磨，一度让我以为这玩意儿是个假的，想下载不知道在哪里下载，所以记录下来。

# 快速获取
网盘链接里有 vmware fusion 13.5.2 的安装包， 需要的自行下载

链接: <https://pan.baidu.com/s/121md5NWJLzkeRflc8V1ORg> 提取码: vtvp 

如果想从官网获取最新版，也可以按照下面的步骤。

# 下载 vmware fusion pro 个人版

1. 你要去官网 大概率会看到这个玩意

![这是图片](/assets/images/fusion_01.png)

所以你要通过 boardcom 进行注册和登录，然后进去之后会看到下面的内容

![这是图片](/assets/images/fusion_02.png)

现在点击 MyDownload 那里是什么都没有的， 你可能不知道在哪里下载，点击下面的 all products, 你会看到很多可以获得的产品。

![这是图片](/assets/images/fusion_03.png)

然后在右上方的搜索框中搜索 fusion ，就会出现很多可以下载的版本，最新的几个版本是可以下载的，旧的版本可能资源没法获取了。

![这是图片](/assets/images/fusion_04.png)

# 安装虚拟机

1. 获取虚拟机镜像
需要注意的是， 如果你的 mac 是 M 系列的芯片，虚拟机镜像一定是 arm 架构的才行。

[镜像获取地址] <https://cdimage.ubuntu.com/jammy/daily-live/current/>

2. 自定义安装
选择自定义安装之后，自己不能选择磁盘大小(默认 20G)，内存(默认 2G), 网络设置(默认使用主机 NAT)。
在启动虚拟机之后可以选择关机，然后自己把上面的默认设置改掉。

3. 无脑安装
进入 ubuntu 的界面之后一定要选择 install 工具进行安装，不要以为秒开，否则每次关机之后重开，以前的工作白做！！！

4. 设置网络桥接模式并连接外网
在上面第二步关机的时候可以将网络设置成桥接模式，这样虚拟机在网络层面上就相当于局域网中一台独立的设备。
开机之后进入虚拟机的网络设置，设置静态 ip， 这里的 ip 设置成和你的主机在同一个子网中就行，子网掩码和网关也是和你的主机相同。

![image](/assets/images/network.png)

ubuntu arm clash 下载地址：<https://app.chongjin.icu/Linux/Clash_Verge_REV/clash-verge_arm64.deb>

```
sudo dpkg -i _arm.deb
```
可以进行安装， 然后打开导入自己的订阅节点即可。
关闭 ipv6, 勾选 系统代理，完美上网～

# 安装 miniconda
> arm 架构下的 miniconda 也是有的， 官网的教程是 x86 的，所以会出错，上 stackoverflow 有人说没有arm 的 miniconda， 其实是有的。

```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
```
ok, 又完美安装。

