---
title: 安全概念
date: 2020-07-19 18:34:17
categories:
- 安全
- 加密
tags:
- 安全
---

# 1 基本概念
数据传输过程中，
为防止第三方看到明文消息，需要加密；
为防止消息被篡改，需要签名；
为防止伪造(消息和签名同时篡改)，需要身份认证；

# 2 加密算法不保证数据完整性(data interity)
If you change any bit of the output, a bit of the clear text will change, and the recipient has no way to detect this.

Both DES and AES are examples of block ciphers, and block ciphers do not have any inherent integrity protection.

以AES-CBC为例，参见下文[点击链接](https://alicegg.tech/2019/06/23/aes-cbc.html)
**Introducing Bit Flipping attacks**
It’s the process of XORing plaintext blocks with the previous ciphertext block during decryption that will introduce a data integrity vulnerability. If we take a look at the XOR truth table, we can see that switching one bit of one of the ciphertext will change the output from 0 to 1, or 1 to 0 :

Ciphertext	Plaintext	Output
0	          0	          0
1	          0	          1
0	          1	          1
1	          1	          0

What this table is telling us, is that if we switch one bit from the previous block of ciphertext, it will be switched in the next deciphered block too.

Of course, that could cause the previous block of plaintext to be replaced by unpredictable garbage data. **Some people could assume that if the ciphertext gets modified, the plaintext will not be anything readable and the application will simply return an error. This is a very dangerous assumption because there are many cases in which some parts of the data might get scrambled yet not trigger any error which could warn the user of an issue and the modified message will affect the system in the way the attacker wants**.