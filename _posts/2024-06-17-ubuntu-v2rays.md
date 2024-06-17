---
layout: single
title:  "ubuntu 下使用订阅节点"
date:  2024-06-17
categories: vmware
---

> 写这篇文章的目的是在没有界面的ubuntu 系统上使用订阅节点。一般的 clash 都是用在桌面版系统中，如果是服务器版怎么办呢？那就可以使用 v2raya了

# 安装v2rayA
1. 添加公钥
```
wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/keyrings/v2raya.asc
```

> root@server-lm4yv6cd:~# wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/keyrings/v2raya.asc
> tee: /etc/apt/keyrings/v2raya.asc: 没有那个文件或目录
> cd /etc/apt
mkdir keyrings
确保密钥卸载这个文件中了，否则下一步apt update中出现以下错误：
错误:10 https://apt.v2raya.org v2raya InRelease
  由于没有公钥，无法验证下列签名： NO_PUBKEY 354E516D494EF95F
正在读取软件包列表... 完成
W: GPG 错误：https://apt.v2raya.org v2raya InRelease: 由于没有公钥，无法验证下列签名： NO_PUBKEY 354E516D494EF95F
E: 仓库 “https://apt.v2raya.org v2raya InRelease” 没有数字签名。
N: 无法安全地用该源进行更新，所以默认禁用该源。
N: 参见 apt-secure(8) 手册以了解仓库创建和用户配置方面的细节。

2. 添加 v2raya 软件源
```
echo "deb [signed-by=/etc/apt/keyrings/v2raya.asc] https://apt.v2raya.org/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
```
3. 安装 v2raya
```
sudo apt install v2raya v2ray ## 也可以使用 xray 包
```
4. 设置 v2raya 并设置开机自启动
```
sudo systemctl start v2raya.service
sudo systemctl enable v2raya.service
```

# 添加订阅节点
1. 登陆云服务器的ubuntu UI界面
在浏览器中打开 localhost:2017进入v2rayA的界面，注册登陆
2. 在订阅节点的网站复制订阅链接
把订阅链接复制到v2rayA的页面
![image](/assets/images/net-default.png)
【是这个其他客户端进行订阅的 “复制默认订阅地址”】 刚开始订阅地址复制错了，一直弄不上，我真的见鬼了，差点窒息了 
3. 启动服务
设置负载均衡转发规则，OK了。
![image](/assets/images/net-set.png)

# reference
关于linux/云服务器科学上网的办法
- [很多很全面，照做系列] <https://ry.huaji.store/2020/08/Linux-magic-network/>
- [关于VPS 和 VPN区别] <https://cn.hostgator.com/news/product/vps/3957.html>
- [节点订阅] <https://www.mojie.cyou/#/dashboard>
- [v2ray网站] <https://v2raya.org/docs/prologue/installation/debian/>
- [视频：v2rayA] <https://www.youtube.com/watch?app=desktop&v=wr67ITwX5Y4>
- [ubuntu下使用 clash for windows] <https://hiif.ong/clash/>