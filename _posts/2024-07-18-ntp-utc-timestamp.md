---
layout: single
title: NTP Time
date: 2024-07-18
tags:
---
> 此 blog 转载自一个大神 [starlitxiling](https://starlitxiling.github.io/2024/07/17/time/)
NTP(Network Time Protocol)是一种广泛使用的时间同步协议，旨在通过网络将计算机时钟同步到UTC(Coordinated Universal Time),目的是为了解决不同系统之间时间一致性的问题，可以看一下这个典型的[时钟同步算法](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E6%99%82%E9%96%93%E5%8D%94%E5%AE%9A#%E6%97%B6%E9%92%9F%E5%90%8C%E6%AD%A5%E7%AE%97%E6%B3%95)
**工作原理**：
- 时间交换：NTP客户端向一个或多个NTP服务器发送时间请求，服务器回应当前的时间。客户端接收到多个服务器的时间后，会计算出最准确的时间。
- 延迟补偿：NTP使用复杂的算法来计算并补偿网络延迟，确保时间的同步性。

**场景**：在分布式系统中，往往使用NTP来实现时间同步，这样可以让依赖时间戳的处理变得正确。

NTP的时间戳是从1900年1月1日开始计算的，NTP时间戳是64位值，其中高32位用于秒，低32位用于秒的小数部分，所以如果从NTP服务器获取时间戳，会得到一个1900年以来的总秒数，你可以用以下程序将一个NTP时间戳转换为ns级，然后和获取到的UTC时间进行比较，计算延迟
~~~c
unsigned long long send_timestamp = read_dora_input_timestamp(event);  # 这里获得的是封装后的从1970年１月１日开始的NTP时间
unsigned long long seconds = send_timestamp >> 32;
unsigned long long fraction = send_timestamp & 0xFFFFFFFF;
unsigned long long seconds_ns = seconds * 1000000000LL;
unsigned long long fraction_ns = (fraction * 1000000000LL) / 4294967296LL;
unsigned long long result = seconds_ns + fraction_ns;
~~~

然后和UTC时间进行计算，如下：
~~~c
struct timespec ts;
clock_gettime(CLOCK_REALTIME, &ts);
unsigned long long receive_timestamp = (unsigned long long)ts.tv_sec * 1000000000LL + ts.tv_nsec;
unsigned long long latency = receive_timestamp - result;
printf("Latency: %llu ns\n", latency);
~~~
在这里的`clock_gettime`因为传入的是`CLOCK_REALTIME`,所以获取的是1970年1月1日到现在的时间，具体的参数信息可以查阅[man page](https://man7.org/linux/man-pages/man3/clock_gettime.3.html)


时间格式|起始时间|位数含义(单位是bit)
--:|--:|--:
NTP|1900年1月１日|高32位是秒，低32位是纳秒
UNIX|1970年１月１日|高64位是秒，低64位是纳秒


在Linux系统中，你可以使用
```bash
sntp time.pool.aliyun.com #查看当前时间和服务器事件的事件偏差
sntp -s time.pool.aliyun.com # 突然调整服务器时间
```


这里挂一些其他的时间协议:
- [精确时间协议PTP](https://en.wikipedia.org/wiki/Precision_Time_Protocol)
- [简单网络时间协议SNTP](https://www.ibm.com/docs/zh/i/7.5?topic=services-simple-network-time-protocol)
- [时间触发协议TTP](https://en.wikipedia.org/wiki/Time-Triggered_Protocol)