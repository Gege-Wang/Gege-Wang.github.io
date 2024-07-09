---
layout: single
title:  "dora node communication"
date:  2024-07-02
categories: dora
---

## 有界通道 flume

```rust
let (rx, tx) = flume::bounded(0);
```
flume 是高性能、多生产者、多消费者的异步通道库。`flume::bounded` 创建一个有固定容量的通道，到达容量的话，发送操作被阻塞，直到有空间可用。`flume::bounded(0)` 意味着发送者和接受者必须同时准备好发送和接收消息。