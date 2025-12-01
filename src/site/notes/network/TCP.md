---
{"dg-publish":true,"permalink":"/network/tcp/"}
---


原文链接：[万字长文 | 23 个问题 TCP 疑难杂症全解析](https://mp.weixin.qq.com/s/LUtk6u_zv0w8g8GIGWEuCw)

>TCP 即 Transmission Control Protocol，可以看到是一个传输控制协议，重点就在这个控制。
## TCP协议头

![](https://pic.imgdb.cn/item/656ec9ffc458853aef041915.png)

首先可以看到 TCP 包只有端口，没有 IP。

Seq 就是 Sequence Number 即序号，它是用来解决乱序问题的。

ACK 就是 Acknowledgement Numer 即确认号，它是用来解决丢包情况的，告诉发送方这个包我收到啦。

标志位就是 TCP flags 用来标记这个包是什么类型的，用来控制 TPC 的状态。

窗口就是滑动窗口，Sliding Window，用来流控。

## 三次握手

![](https://pic.imgdb.cn/item/656eca50c458853aef050feb.png)


## 四次挥手

![](https://pic.imgdb.cn/item/656ecaa2c458853aef0640b4.png)


### 为什么要TIME_WAIT

断开连接发起方在接受到接受方的 FIN 并回复 ACK 之后并没有直接进入 CLOSED 状态，而是进行了一波等待，等待时间为 2MSL。

MSL 是 Maximum Segment Lifetime，即报文最长生存时间，RFC 793 定义的 MSL 时间是 2 分钟，Linux 实际实现是 30s，那么 2MSL 是一分钟。

那么为什么要等 2MSL 呢？

- 就是怕被动关闭方没有收到最后的 ACK，如果被动方由于网络原因没有到，那么它会再次发送 FIN， 此时如果主动关闭方已经 CLOSED 那就傻了，因此等一会儿。
    
- 假设立马断开连接，但是又重用了这个连接，就是五元组完全一致，并且序号还在合适的范围内，虽然概率很低但理论上也有可能，那么新的连接会被已关闭连接链路上的一些残留数据干扰，因此给予一定的时间来处理一些残留数据。


## TCP重传/滑动窗口
[你还在为 TCP 重传、滑动窗口、流量控制、拥塞控制发愁吗？看完图解就不愁了](https://mp.weixin.qq.com/s/HjOUsKn8eLfDogbBX3hPnA?poc_token=HCHMbmWjwdhoitUl-XlML8M3cSPmzkBFuzq92oWs)

