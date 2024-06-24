---
layout: single
title:  "127.0.0.1 和 localhost 以及 peer-ip"
date:  2024-06-18
categories: net
---

> 在进行 dora 分布式节点部署的时候，发现 inter-daemon 的地址被记录成了 127.0.0.1, 而不是它的私有地址，所以这篇文章总结一下使用 127.0.0.1 和 使用私有地址的区别，以及什么时候 peer-ip 会将对方识别为 127.0.0.1，什么时候会将对方识别为私有地址。

# 1. 本地回环地址是不是会走网络栈

可以看到虚拟回环借口通常会被命名为 lo 或者 lo0， 在 macOS 中使用 ifconfig 进行查看。

```
╰─$ ifconfig
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
	options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
	inet 127.0.0.1 netmask 0xff000000
	inet6 ::1 prefixlen 128
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1
	nd6 options=201<PERFORMNUD,DAD>
```
127 开头的都属于回环地址。源码里是这样定义的。
```
 /* Address to loopback in software to local host.  */
 #define INADDR_LOOPBACK   0x7f000001  /* 127.0.0.1   */
```

# 2. 127.0.0.1 和 localhost 是什么关系

localhost 不是一个地址，是一个域名，就类似于 www.baidu.com，所以需要对 localhost 进行 DNS 解析。在 /etc/hosts 中有对应关系
```
╰─$ cat /etc/hosts                                                          2 ↵
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
```
localhost 的 DNS 解析会得到回环地址 127.0.0.1，对于 ipv6 的地址类型，本地回环地址是 ::1， 所以 默认情况下 localhost 对应的就是本机的回环地址，但是如果修改 /etc/hosts 就可以改变默认设置，只不过没必要改。

# 3. 监听 0.0.0.0 有什么用？

ping 0.0.0.0 是失败的，因为它的 ipv4 中表示的是无效的目的地址。
```
╰─$ ping 0.0.0.0
PING 0.0.0.0 (0.0.0.0): 56 data bytes
ping: sendto: Socket is not connected
ping: sendto: Socket is not connected
Request timeout for icmp_seq 0
ping: sendto: Socket is not connected
Request timeout for icmp_seq 1
^C
```
但是我们会在启动服务器的时候监听一个 ip 和 端口，那么如果这个 ip 是 0.0.0.0， 那么它表示本机的所有 ipv4 地址。

```
 /* Address to accept any incoming messages. */
 #define INADDR_ANY    ((unsigned long int) 0x00000000) /* 0.0.0.0   */
```
刚刚提到的 127.0.0.1 和 192.168.31.6 ，都是本机的IPV4地址，如果监听 0.0.0.0 ，那么用上面两个地址，都能访问到这个服务器。



# reference
- [回环地址] <https://shansan.top/2021/11/27/loopback-addr/>