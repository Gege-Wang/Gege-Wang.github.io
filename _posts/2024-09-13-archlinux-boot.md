---
layout: single
title: "archlinux boot failed"
date: 2024-09-13
categories: archlinux
---

> archlinux firefox 打不开了，于是重启，启动失败，无法进入系统。

## 报错日志
启动过程中出现下面的错误 
![boot-failed](/assets/images/archlinux-boot.jpeg)
看起来是 gnupg 服务出现了问题，并且文件系统是只读挂载的。

## 解决方案
**首先，我们需要一个能够输入命令的界面，否则什么事情都做不了。**  

### 1. 进入紧急模式或救援模式

- 在启动菜单（GRUB）中选择你的 Arch Linux 内核条目。
- 按 e 键来编辑引导参数。
- 找到以 linux 开头的行，在行尾添加 systemd.unit=emergency.target，然后按 Ctrl + X 或 F10 来启动到紧急模式。
- 紧急模式会启动最小限度的系统环境，你将拥有 root 权限，可以尝试进行一些修复。

### 2. 修复文件系统

输入密码进入 root 模式后，发现突然报错
```bash
BTRFS error incorrect extent count for 25800212480;count 565 expect 566
```
这是文件系统元数据的错误，需要修复。

- 为了防止进一步的损坏，首先尝试将文件系统挂载为只读。
```bash
# 你可以通过以下命令挂载 Btrfs 文件系统为只读模式
sudo mount -o ro /dev/nvme0n1p7 /mnt
# 如果文件系统已经挂载为读写模式，先卸载它，再挂载为只读模式
sudo umount /mnt
sudo mount -o ro /dev/nvme0n1p7 /mnt
```
- 使用 btrfs check 进行检查
```bash
sudo btrfs check /dev/nvme0n1p7
# 这个命令将检查文件系统是否有错误，但不会进行任何修复操作。如果检查结果显示有问题，可以尝试修复。
# 如果权限不够，尝试下面的命令。
sudo btrfs check --force /dev/nvme0n1p7
```

- 使用 btrfs check --repair 进行修复
```bash
sudo btrfs check --repair /dev/nvme0n1p7
```

### 3. 重启系统

```bash
systemctl reboot
```
成功解决 :)


## reference
- [linux 进入 recovery mode](https://worktile.com/kb/ask/324982.html)
- [archlinux 解决 gnupg 服务启动失败](https://chatgpt.com/share/66e3c0a0-1a8c-8004-8798-82dd26e8fb99)