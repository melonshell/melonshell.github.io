---
title: TCP_NODELAY和延迟ACK
date: 2019-11-23 11:01:13
categories:
- TCPIP系列
tags:
- TCP/IP
- TCP_NODELAY
- 延迟ACK
---

# 1 MTU和MSS
MTU：Maximum Transmission Unit，最大传输单元，数据链路层的概念，限制的是数据链路层的Payload，即上层协议的大小，如IP，ICMP等。若传输数据包的长度大于MTU，则会对其分片；以太网的MTU为1500B。

MTU对IP协议的影响：
* 较大的IP数据包分为多个小包，并给每个小包打上标签
* 每个小包的IP协议头的16位标识(id)相同
* 每个小包的IP协议头的三位标志字段中，第一位保留，第二位置0，第三位表示结束标志(最后一个小包置1，其余小包置0)
* 达到对端将这些小包按顺序重组，拼装在一起返回给传输层
* 任意小包丢失，Peer重组就会失败，但IP层不负责重传(IP协议无连接，不可靠)

MTU对UDP的影响：
UDP数据报超过1472B(1500 - 20(IP首部) - 8(UDP首部))，则UDP数据报会在网络层分片为多个IP数据包。

MTU对TCP的影响：
* TCP数据报受限于MTU，MSS
* 理想情况下，MSS值恰好是IP不会分片处理的最大长度(受限于MTU)

MSS：Maximum segment size，最大报文段长度，单位B，在连接建立，即发送SYN、SYN/ACK的时候，同时会将MSS发送给peer，告诉peer每个报文段能够发送的最大长度。MSS值通常设置为MTU减去IP和TCP头部的固定大小，IPV4的头部为20B，IPV6的头部为40B，IPV4和IPV6的tcp头部均为20B，因此IPV4的MSS为1460B，IPV6的MSS为1440B。
![MSS](/pic/tcp-nodelay-delayed-ack.png)

# 1 Nagle算法
Nagle算法的目的是减少网络上的小包传输，小包被定义为小于MSS的数据包；Nagle算法规定，如果连接存在已发送但未被ACK的数据包，那么此连接不会再发送小包，直到已发送的数据包收到ACK，这样，小数据包被合并为一个大数据包发送。
 tcp_nagle_check：if packet can be sent now without violation Nagle's rules:
 * 1. It is full sized. (provided by caller in %partial bool)
 * 2. Or it contains FIN. (already checked by caller)
 * 3. Or TCP_CORK is not set, and TCP_NODELAY is set.
 * 4. Or TCP_CORK is not set, and all sent packets are ACKed. With Minshall's modification: all sent small packets are ACKed.

# 2 延迟ACK
通常TCP收到数据后，不会立即响应ACK，而是等待响应数据：
* 如果有响应数据，ACK和响应数据一起发送；
* 如果没有响应数据，延迟等待响应数据，一般情况下40ms，若40ms内有数据发送，则ACK随数据一起发送；若没有数据发送，则ACK延迟定时器触发时，发现ack尚未发送，立即单独发送；
* 如果在等待发送ACK期间，收到了第二个数据，此时立即发送ACK；
ACK延迟定时器在内核启动时，每40ms(200ms/500ms)触发一次。

延迟ACK的好处：
* 避免糊涂窗口综合症；
* 发送数据捎带ACK，减少单独发送ACK；
* 如果延迟时间内有多个数据段到达，协议栈允许一个ACK确认多个报文段；

# 3 问题
假设客户端的请求需要等待服务端应答后才能继续，即串行执行，如在用ab性能测试时只有一个并发做10k的压力测试，测试地址返回的内容只有Hello world，ab发出的请求需要等待服务器response后，才能发起下一个请求；此时ab请求的相关内容包含在header中，而服务器需要返回两个数据：response header和body；

服务器发送端发送的第一个write不会被缓存起来，而是立刻发送（response header）；
这时ab接收端收到对应的数据，但它还需要更多数据（body）才能进行处理，所以不会向服务器发送数据，故没有机会把ACK带回去，根据Delayed ACK机制，服务端发送的response header未被确认，导致body不会被发送；

这样，服务器在等待ab的ACK，ab则在等待服务器的reposonse body，互相等待，死锁了！！！

直到ab的Delayed ACK超时（40ms），此ACK被发送至服务端，服务端继续发送缓存的body，此时ab才收到完整的数据，进行应用层处理，处理完成后继续下一个request；

因此服务器端会在read时出现40ms的阻塞。

# 4 TCP_NODELAY
默认情况下，TCP发送数据采用Nagle算法，虽然提高了网络吞吐量，但是实时性却降低了，这对于交互性很强的应用程序来说是不被允许的，可以用TCP_NODELAY选项禁止Nagle算法。
