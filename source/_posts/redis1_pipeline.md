---
title: pipeline
date: 2020-06-07 22:46:04
categories:
- Redis
tags:
- pipeline
---

# pipeline介绍
Redis协议是基于Request/Response的，客户端发送一个命令，等待Redis应答，Redis接收到命令，处理后应答。这种情况下，如果同时需要执行大量的命令，那就需要等待上一条命令应答后再执行，这中间不仅仅多了RTT（Round Time Trip），而且还频繁的调用系统IO。Pipeline允许客户端一次发送多条命令，而不等待上一条命令执行的结果。

* pipeline"独占"connection，直到pipeline结束，期间将不能进行非"管道"类型的其他操作；如果你的pipeline指令集很庞大，为了不干扰链接中的其他操作，你可以为pipeline操作新建Client连接，让pipeline和其他正常操作分离在2个client连接中；
* 如果使用pipeline发送的命令很多，建议对返回的结果加标签，当然这也会增加使用的内存；

# 实现原理
要实现pipeline，既要服务端支持，也要客户端支持。对服务端来说，所需要的是能够处理客户端通过同一个TCP连接发来的多个命令，可以理解为，将多个命令切分，和处理单个命令一样（老生常谈的黏包现象），Redis就是这样处理的；而客户端则要将多个命令缓存起来，缓冲区满了就发送，然后再写缓冲，最后才处理Redis的应答。

# 注意点
Redis的pipeline和transaction不同，transaction会存储客户端的命令，最后一次性执行；而pipeline则是处理一条，响应一条，但是这里却有一点，就是客户端会并不会调用read去读取socket里面的缓冲数据，这也造就了，如果Redis应答的数据填满了该接收缓冲（SO_RECVBUF），那么客户端会通过ACK，WIN=0（接收窗口）来控制服务端不能再发送数据，如此一来，数据就会缓冲在Redis的客户端应答列表里面，所以需要注意控制pipeline的大小。

TCP传输层来说，pipeline的多个命令很大概率是在同一个TCP segment中发送，当然也有可能由于分段，命令处于不同的segment；
正常请求，需要等待上一条命令应答后，再执行下一条，Redis(createClient)设置了TCP_NODELAY；

# 性能提升
* 减少了RTT(round-trip time, which is the time it takes for a small packet to travel from client to server and back to the client)
* 减少socket IO调用次数(IO调用涉及到用户态到内核态之间的切换)
* 减少connection连接，redis对QPS敏感，如果客户端使用不合理而造成qps放大，则redis可能更早触及性能瓶颈而导致系统响应严重下降

# 原生批量命令和pipeline
* 原生批量命令是原子的，pipeline非原子
* 原生批量命令是一个命令对应多个key，pipeline支持多个命令
* 原生批量命令是redis服务端实现，pipeline需要服务端与客户端共同实现

# 事务和pipeline
同一个redis client发送命令MULTI表示准备执行事务，同时发送的命令会在server端排队，期间不影响其他客户端发送命令，服务器不会阻塞，该客户端发送EXEC，服务器端把执行完的结果一起发送给客户端。如果同一个Transaction中的命令太多，服务器会阻塞(也就是在事务执行期间，服务器不会执行其他命令)。

* pipeline关注的是RTT时间，transaction关注的是一致性；
* pipeline一次请求，服务端顺序执行，一次返回；
* transaction多次请求（MULTI＋其他n个命令＋EXEC，所以至少是2次请求），服务端顺序执行，一次返回；
* 集群模式下使用pipeline时，slot必须是对的，不然服务端会返回redirecred to slot xxx的错误；不建议使用Transaction，因为假设一个Transaction中的命令即在Master A上执行，也在Master B执行，A成功了，B因为某种原因失败了，这样数据就不一致了，这个有点类似于分布式事务，无法保证绝对一致性。
