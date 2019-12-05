---
title: 常用命令
date: 2019-11-26 16:13:27
categories:
- docker
---

1 查看容器的挂载目录
docker inspect container_name | grep Mounts -A 20
docker inspect container_id | grep Mounts -A 20
