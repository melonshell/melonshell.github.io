---
title: hmac算法
date: 2020-07-19 17:03:20
categories:
- 安全
- 加密
tags:
- 安全
- hmac
---

# 1 HMAC概念
In cryptography, an HMAC (sometimes expanded as either keyed-hash message authentication code or hash-based message authentication code) is a specific type of message authentication code (MAC) involving **a cryptographic hash function and a secret cryptographic key**. As with any MAC, it may be used to simultaneously verify **both the data integrity and the authenticity of a message**.

通过同一个渠道发送数据和散列值的话(如消息认证码)，就要考虑数据和MD5同时被篡改的问题，如果第三方修改了数据，然后进行MD5散列，并一起发给接收方，则接收方无法觉察到数据被篡改。HMAC-MD5可以用发送方和接收方都有的key进行计算，而没有此key的第三方是无法计算出正确的散列值，因此可以防止数据和MD5同时被篡改(伪造)。

通信双方共享了一个密钥key，现在要发消息给对方，既要保证消息没有篡改，又要能证明信息确实是你本人所发，那么就要把原信息和使用key计算的HMAC值一起发送。对方接收后，使用key把消息重新计算HMAC，与接收的HMAC比较，如果一致，则消息没有被篡改也没有被伪造。

# 2 典型应用
HMAC加密算法的一个典型应用是登录身份认证中，认证流程如下：
* 客户端向服务器发起验证请求;
* 服务器收到此请求后生成一个随机数返回给客户端，并在会话中记录随机值;
* 客户端将收到的随机数作为密钥，和用户密码进行HMAC-MD5运算，把结果提交给服务器;
* 服务器也使用该随机数与存储在服务器数据库中的用户密钥进行HMAC-MD5运算，如果运算结果与客户端传回的响应结果相同，则验证用户合法;

# 3 安全性
* 密钥是双方事先约定的，第三方不可能知道。作为非法截获信息的第三方，能够得到的信息只有随机数和客户端回传的HMAC结果，无法根据这两个数据推算出用户密钥；
* HMAC加密算法与一般加密的重要区别在于，它具有瞬时性，即认证只在当时有效，而加密算法被破解后，之前的加密结果都可能被解密；
