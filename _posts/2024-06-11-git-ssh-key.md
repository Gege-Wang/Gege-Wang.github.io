---
layout: single
title:  "git ssh key"
date:  2024-06-11
categories: git
---
> 写这篇文章的目的是为了搞清楚 git 的认证机制， 虽然 https token 和 ssh 我都没有搞的非常明白，但是呢，已经能够解决一些常见的问题。

### 遇到的问题

在 git 增加仓库的时候，出现了这个问题。猜测我之前只是设置了 https 的 token ，并没有添加 ssh 的密钥，所以在使用 ssh url 的时候就会出现权限错误。

```bash
╰─$ sudo kbuild patch add axstarry                                        100 ↵
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.34s
Patching https://github.com/Starry-OS/axstarry.git:ed3c90b8e10d9bdd92583833dd5ay
ssh url: git@github.com:Starry-OS/axstarry.git
Cloning into 'crates/axstarry'...
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
Error: No such file or directory (os error 2)
```

### 解决方案

> 方案概述：生成 ssh 私钥和 公钥。将公钥复制到  GitHub 上，将私钥添加在本地的 ssh-client 上。根据方案概述可以看出，这个 ssh 私钥是只针对这一台机器适用的，用其他的机器就要重新设置 ssh 密钥。

2.1 验证确实是没有 ssh 权限

```shell
╰─$  ssh -T git@github.com                                                100 ↵
git@github.com: Permission denied (publickey).
```

2.2 查看本机是否有 ssh 密钥

```shell
ls ~/.ssh/
```

如果存在 id_rsa 和 id_rsa.pub 就说明存在。

2.3 生成 ssh 密钥

```shell
cd ~/.ssh
ssh-keygen -t rsa -C "git@github.com" 
```

2.4 将公钥拷贝到 github

```shell
cat id_rsa.pub
```

github settings -> SSH and GPG keys -> new SSH key
将公钥拷贝到框里就行了。把这一步做完就去验证也能获得相同的结果，但是拉取仓库还是不成功。

2.5 添加私钥到本地的 ssh-client

```shell
$eval "$(ssh-agent -s)" # 确保 ssh-agent 是工作的
ssh-add ~/.ssh/id_rsa
```

2.6 验证

```shell
╰─$ ssh -T git@github.com
Hi <your-name>! You've successfully authenticated, but GitHub does not provide shell access.
```

ok，完美解决。

### 记录配置代理

```shell
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
git config --global --unset http.proxy
git config --global --unset https.proxy
```
