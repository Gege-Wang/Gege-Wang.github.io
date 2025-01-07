<!--
 * @Author: Gege-Wang 2891067867@qq.com
 * @Date: 2024-11-15 00:42:16
 * @LastEditors: Gege-Wang 2891067867@qq.com
 * @LastEditTime: 2024-12-18 13:11:17
 * @FilePath: /Myblog/_posts/2024-11-15-rust-mini-redis.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->

---
layout: single
title: "用 rust 写 mini-redis 实践"
date: 2024-11-15
categories: rust
---

> 最近一时兴起，又想着回去写一下 mini-redis，之前写的时候因为异步编程非常不熟悉，基本上都不是很懂，后来写了 rust os 之后对于一些简单的异步编程框架有了一些粗浅的理解。

## 1. 确立目标
我们这次的目标非常非常清晰，就是写一个 redis 服务端和客户端。具体要求如下：
1. 客户端能够异步的与客户端通信。
2. 。。。

## 2. 拆解任务
[此处是一个任务拆解架构图]

## 3. 实践详解
### 3.1 创建 redis 客户端
```rust
use mini_redis::client;

#[tokio::main]
async fn main() {
    let mut client = client::connect("127.0.0.1:6379").await.unwrap();
    let val = client.get("foo").await.unwrap();
    println!("Got foo value: {:?}", val);
}
```
tokio 异步编程框架适合用在 I/O 操作比较多的情况下，使用的是协作式调度，因此在高并发，网络连接或者数据库上用的比较多。在 CPU 密集型的任务上使用多线程比较合适，能够更加充分的利用多核的资源。
上面的代码简单来说完成了三件事情：
1. 使用 mini-redis 的借口和本地的 6379 端口建立连接。需要在本地启动 mini-redis-server 服务端。
2. 通过连接获取 key: "foo" 的值。.await 代表这段代码完成是异步的。
3. 打印这个值

核心线程跑异步代码，会产生在每一个不同的CPU核心上。