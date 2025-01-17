---
layout: single
title: "通过 VNC 远程连接 ubuntu 服务器"
date: 2025-1-7
categories: ubuntu
---

> 事情的起因是我想要在服务器上跑 rustdesk 和 vmware，所以需要图形化界面，有需要远程连接。这次做的比较简陋，但是功能基本上有了。

## 服务器配置

### 创建新用户

服务器不要用 root 用户，因为 VNC 连接对 root 用户有限制，一些可能的问题比如方向键不管用、firefox 沙盒环境无法开启等等。   
所以如果你是 root 用户需要创建一个新的用户。   
```bash
adduser yourusername
su - yourusername
```

1. 服务器是 ubuntu，需要安装 tightvncserver。
```bash
sudo apt update
sudo apt install tightvncserver -y
vncserver
```
2. 安装图形化界面

这里我安装的是xfce4，也可以安装 gnome，在后面也有相关的指导。
```bash
sudo apt install xfce4 xfce4-goodies -y
```
编辑 VNC 配置文件： 修改或创建 ~/.vnc/xstartup 文件：  
```
nano ~/.vnc/xstartup
```
内容如下：   
```bash
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

保存后赋予执行权限：
```bash
chmod +x ~/.vnc/xstartup
```

重启 VNC 服务
```bash
vncserver -kill :1
vncserver :1
```
### 安装 gnome(可选)

```bash
sudo apt install ubuntu-desktop 
sudo apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal 
sudo reboot #重启即可看到图形界面
```
1. ubuntu 安装 VNC
```bash
sudo apt-get install x11vnc
sudo apt-get install lightdm
```
安装过程中会出现以下选项,选择lightdm然后回车即可    
设置密码,设置密码后,会问你是否需要将密码保存在:/home/root1/.vnc/passwd,输入y确认即可    
```bash
x11vnc -storepasswd
```

2. 设置 VNC 开机启动

创建一个x11vnc.service文件   
```bash
sudo vim /lib/systemd/system/x11vnc.service
```
按i键进入编辑模式,添加如下信息  
> !!注意: <USERNAME>替换为您ubuntu用户名,添加完成后按Esc键退出编辑,然后输入冒号:wq保存
  
```bash
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/<USERNAME>/.vnc/passwd -rfbport 5900 -shared

[Install]
WantedBy=multi-user.target
```
设置开机启动
```bash
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
```
-------------------------------------------------------
下面是原来的一些实验，事实证明 gnome 完全可以在 VNC 中显示。下面的实验已作废。
```bash
#! /bin/bash
export $(dbus-launch)
export XKL_XMODMAP_DISABLE=1
unset SESSION_MANAGER
gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
gnome-terminal &

xsetroot -solid grep
vncconfig -iconic & 
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
gnome-session &

VNCSERVERS="2:root"
VNCSERVERARGS[1]="-GEOMETRY 800x600"
```
看网上说 VNC 连接灰色是正常的，因为 tightvncserver 并不是和 gnome 用的同一套图形系统，上面的命令可以打开终端和文件系统的图形界面。  

### 把 VNC 设置为开机自启动
1. 创建一个 systemd 服务文件。
```
sudo nano /etc/systemd/system/vncserver@.service
```
将以下内容复制并粘贴到文件中，确保根据你的实际 VNC 配置调整用户名、显示号和路径。
```bash
[Unit]
Description=Start VNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=<your-username>
PAMName=login
PIDFile=/home/<your-username>/.vnc/%H:%i.pid
ExecStart=/usr/bin/vncserver :%i -geometry 1280x1024 -depth 24
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```
- 将 <your-username> 替换为你在服务器上的用户名。    
- :%i 是 VNC 显示号的占位符。例如，如果你启动的 VNC 会话是 :1，那么 ExecStart=/usr/bin/vncserver :1 -geometry 1280x1024 -depth 24 将启动 VNC 会话。    
- 可以根据需要修改分辨率（-geometry 1280x1024）和色深（-depth 24）。    
- 保存并退出编辑器（按 Ctrl+X，然后按 Y 和 Enter）。     
```bash
sudo systemctl daemon-reload
sudo systemctl start vncserver@1
sudo systemctl enable vncserver@1
sudo systemctl status vncserver@1
```

### 安装浏览器
如果你遇到 Snap 的问题，另一个选择是通过 Flatpak 安装 Firefox。
```bash
sudo apt install flatpak
# 添加 Flathub 仓库：
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
# 安装 Firefox：
flatpak install flathub org.mozilla.firefox
# 运行 Firefox：
flatpak run org.mozilla.firefox
```

## 客户机配置
客户机随便装个 VNC 客户端就行，我在 MAC 系统使用的是 realVNC 仅供参考。


# 参考文献
1. https://www.cpolar.com/blog/remote-desktop-ubuntu