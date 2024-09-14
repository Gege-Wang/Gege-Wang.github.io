---
layout: single
title: "archlinux wps cn"
date: 2024-09-14
categories: archlinux
---
> archlinux 安装 wps 竟然都不是一步到位，挺好挺好。
## 安装 wps
```bash
yay -S wps-office
```
好家伙，安装好了之后不能输入中文，没有中文输入法，脑神经炸裂了。看来 archlinux 的中文输入法问题要一直伴随着我了。

## 导入环境变量
编辑 `/usr/bin/wps` 在文件中添加两行代码
```bash
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
```
输入之后重启 wps，重启电脑还是有问题。好吧，神经。

## 安装 wps-office-cn
```bash
yay -R wps-office
yay -S wps-office-cn wps-office-mui-zh-cn ttf-wps-fonts
```
全部安装完之后打开全部中文化。之前不让我输中文，现在把你全换成中文。 :)
