---
title: socket之accept
date: 2021-11-07 21:17:13
categories:
- TCPIP系列
tags:
- accept
---

# 1 accept
accept从全连接(accept)队列中取出一个连接，并且创建一个新的socket，返回新socket的描述符。新socket和listen socket绑定的端口相同：
```c
#include <stdlib.h>
#include <stdio.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <string.h>
#include <errno.h>

int main(int argc, char** argv) {
    int listenFD, connFD, ret, len, flag = 1;
    struct sockaddr_in sAddr, cAddr;
    char buf[4096] = {0};

    listenFD = socket(AF_INET, SOCK_STREAM, 0);
    if (listenFD == -1) {
        printf("socket err");
        return 1;
    }
    setsockopt(listenFD, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(int));
    memset(&sAddr, 0, sizeof(struct sockaddr_in));
    sAddr.sin_family = AF_INET;
    sAddr.sin_addr.s_addr = htonl(INADDR_ANY);
    sAddr.sin_port = htons(6666);

    ret = bind(listenFD, (struct sockaddr*)&sAddr, sizeof(sAddr));
    if (ret == -1) {
        printf("bind err:%d, msg:%s", errno, strerror(errno));
        return 1;
    }

    ret = listen(listenFD, 10);
    if (ret == -1) {
        printf("listen err");
        return 1;
    }

    while(1) {
        printf("befor accept\n");
        connFD = accept(listenFD, (struct sockaddr*)NULL, NULL);
        if (connFD == -1) {
            printf("accept err");
            return 1;
        }
        printf("after accept\n");

        ret = recv(connFD, buf, 4096, 0);
        if (ret == -1) {
            printf("recv err");
            return 1;
        }
        printf("recv:%s, ret:%d\n", buf, ret);
        memset(&cAddr, 0, sizeof(struct sockaddr_in));

        getsockname(connFD, (struct sockaddr*)&cAddr, &len);
        printf("port:%d", ntohs(cAddr.sin_port));
        close(connFD);
    }
}
```
结果：
befor accept
after accept
recv:
, ret:2
port:6666
可以发现，新socket和listen socket绑定的端口相同。
