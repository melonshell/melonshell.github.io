---
title: Redis集群搭建
date: 2019-12-29 10:37:03
categories:
- Redis
tags:
- docker compose
- 集群搭建
- redis
---

# 1 docker compose
docker compose是一个轻量级的容器编排工具，它使用yaml来编排容器，可以在n个容器中做通信。
显示正在运行的进程：docker-compose top
启动所有服务：docker-compose up -d
列出所有容器：docker-compose ps
停止正在运行的容器，可通过docker-compose start再次启动：docker-compose stop
停止、删除容器、网络、卷、镜像：docker-compose down
查看服务容器的日志：docker-compose logs

# 2 redis集群
redis集群可以把大量的数据分散到n个节点，同时可以对每个节点做备份，来保障redis数据的高可用和稳定性。
集群至少6个节点，分别3主3从，数据被分配到3个主节点，redis的集群是根据内部的槽(slot)来实现，redis有16384个槽，通过CRC16(key)%16384可以计算出key应该存放在哪个槽中。节点之间通过redis的Gossip协议传播信息，检查目标节点是否存活等。如果有主节点宕机，redis会选举出一个新的主节点来替代宕机的主节点，以保证服务的可用性。
  ![redis集群架构](/pic/redisbuildcluster201912291056.png)

# 3 docker compose编写
## 3.1 下载redis.conf
下载地址：https://github.com/antirez/redis/blob/unstable/redis.conf
## 3.2 修改redis.conf
以端口6379为例
开启集群功能：cluster-enabled yes
设置节点端口：port 6379
节点超时时间，单位ms：cluster-node-timeout 15000
集群内部配置文件：cluster-config-file "node-6379.conf"
## 3.3 redis配置
1) redis.conf配置目录如下，需要**注释掉redis.conf第472行(可能是新的配置参数，参见注释说明)，否则docker无法启动**：
```
# repl-diskless-load disabled
否则报错
*** FATAL CONFIG FILE ERROR ***
Reading the configuration file, at line 472
>>> 'repl-diskless-load disabled'
Bad directive or wrong number of arguments
```
2) 注释：# bind 127.0.0.1
3) 关闭保护模式：protected-mode no
**2)和3)的目的是取消redis的连接保护，否则外部容器无法连接redis服务**。

/home/redis/master-6379/redis.conf
/home/redis/master-6380/redis.conf
/home/redis/master-6381/redis.conf
/home/redis/slave-6382/redis.conf
/home/redis/slave-6383/redis.conf
/home/redis/slave-6384/redis.conf

## 3.4 docker-compose.yml配置
1) 挂载文件夹，如果文件夹不存在，则docker会先创建文件夹；
2) 挂载文件，文件必须存在，否则会自动创建文件夹，如redis.conf不存在，则会创建redis.conf文件夹；

例子：
host上文件A不存在，container中文件B已存在，如果将A挂载到B，则会报错：
```
Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type.
```
同时会在host上生成两个空目录A和B, 但是container无法启动。

3) 配置.env文件
```
redis1_port=6379
redis1_cluster=16379
redis2_port=6380
redis2_cluster=16380
redis3_port=6381
redis3_cluster=16381
redis4_port=6382
redis4_cluster=16382
redis5_port=6383
redis5_cluster=16383
redis6_port=6384
redis6_cluster=16384
```

4) 配置docker-compose.yml
redis集群需要容器的内部和外部通信，所以一般设置容器的network为host模式，docker-compose.yml具体配置如下：
```yml
version: '3.3'
services:
  # 节点1
  master-6379:
    # 启动之后的容器名称
    container_name: master-6379
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'redis'
    # 端口映射
    ports:
      - ${redis1_port}
      - ${redis1_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.11
    volumes:
      # 容器的data映射到宿主机
      - /data/redis/master-${redis1_port}:/data
      # 加载配置文件
      - /home/redis/master-${redis1_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点2
  master-6380:
    container_name: master-6380
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'redis'
    # 端口映射
    ports:
      - ${redis2_port}
      - ${redis2_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.12
    volumes:
      # 容器的data映射到宿主机
      - /data/redis/master-${redis2_port}:/data
      # 加载配置文件
      - /home/redis/master-${redis2_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点3
  master-6381:
    container_name: master-6381
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'redis'
    # 端口映射
    ports:
      - ${redis3_port}
      - ${redis3_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.13
    volumes:
      # 容器的data映射到宿主机
      - /data/redis/master-${redis3_port}:/data
      # 加载配置文件
      - /home/redis/master-${redis3_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点4
  slave-6382:
    container_name: slave-6382
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'redis'
    # 端口映射
    ports:
      - ${redis4_port}
      - ${redis4_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.14
    volumes:
      # 容器的data映射到宿主机
      - /data/redis/slave-${redis4_port}:/data
      # 加载配置文件
      - /home/redis/slave-${redis4_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点5
  slave-6383:
    container_name: slave-6383
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'redis'
    # 端口映射
    ports:
      - ${redis5_port}
      - ${redis5_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.15
    volumes:
      # 容器的data映射到宿主机
      - /data/redis/slave-${redis5_port}:/data
      # 加载配置文件
      - /home/redis/slave-${redis5_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点6
  slave-6384:
    container_name: slave-6384
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'redis'
    # 端口映射
    ports:
      - ${redis6_port}
      - ${redis6_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.16
    volumes:
      # 容器的data映射到宿主机
      - /data/redis/slave-${redis6_port}:/data
      # 加载配置文件
      - /home/redis/slave-${redis6_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
networks:
  # 创建集群网络，在容器之间通信
  cluster-net:
    ipam:
      config:
        - subnet: 172.16.238.0/24
```

执行命令时，需要去掉中文注释：
docker-compose up -d
docker-compose ps

# 4 配置集群
1) 查看容器IP
* docker ps -a获取container ID;
* docker inspect container_ID;

2) 初始化集群
进入**任意**一个redis容器：
docker exec -it ba83800c75f3 /bin/bash
执行命令然后输入yes：
```
# -a redis表示连接密码为redis
#--cluster-replicas表示创建一个从节点
redis-cli --cluster create 172.16.238.11:6379 172.16.238.12:6380 172.16.238.13:6381 172.16.238.14:6382 172.16.238.15:6383 172.16.238.16:6384 --cluster-replicas 1

root@ba83800c75f3:/data# redis-cli --cluster create 172.16.238.11:6379 172.16.238.12:6380 172.16.238.13:6381 172.16.238.14:6382 172.16.238.15:6383 172.16.238.16:6384 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.16.238.15:6383 to 172.16.238.11:6379
Adding replica 172.16.238.16:6384 to 172.16.238.12:6380
Adding replica 172.16.238.14:6382 to 172.16.238.13:6381
M: 8069f31dabaf00445c974c6ed64104d9613f5d36 172.16.238.11:6379
   slots:[0-5460] (5461 slots) master
M: df9cff52073cc8f506b069610a703f898b3fe0f0 172.16.238.12:6380
   slots:[5461-10922] (5462 slots) master
M: eb98e131c0fa78d23015c7665877f109924a8c30 172.16.238.13:6381
   slots:[10923-16383] (5461 slots) master
S: f5174164ad32584d9a667bdb9e0ffe7e90be7ff2 172.16.238.14:6382
   replicates eb98e131c0fa78d23015c7665877f109924a8c30
S: b621cff395d2852250b80f7f2cdb35a85ee050cb 172.16.238.15:6383
   replicates 8069f31dabaf00445c974c6ed64104d9613f5d36
S: 67cf3535e28f5bc3e43fe1219a99271ce2bd0f41 172.16.238.16:6384
   replicates df9cff52073cc8f506b069610a703f898b3fe0f0
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.16.238.11:6379)
M: 8069f31dabaf00445c974c6ed64104d9613f5d36 172.16.238.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: df9cff52073cc8f506b069610a703f898b3fe0f0 172.16.238.12:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: b621cff395d2852250b80f7f2cdb35a85ee050cb 172.16.238.15:6383
   slots: (0 slots) slave
   replicates 8069f31dabaf00445c974c6ed64104d9613f5d36
M: eb98e131c0fa78d23015c7665877f109924a8c30 172.16.238.13:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 67cf3535e28f5bc3e43fe1219a99271ce2bd0f41 172.16.238.16:6384
   slots: (0 slots) slave
   replicates df9cff52073cc8f506b069610a703f898b3fe0f0
S: f5174164ad32584d9a667bdb9e0ffe7e90be7ff2 172.16.238.14:6382
   slots: (0 slots) slave
   replicates eb98e131c0fa78d23015c7665877f109924a8c30
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
# 5 验证集群
普通模式连接：test根据哈希槽计算，是分布在6392服务上，所以这里会转到6392。
```
[root@VM_113_88_centos /home/redis]# redis-cli -h 172.16.238.11 -p 6379
172.16.238.11:6379> set test 1
(error) MOVED 6918 172.16.238.12:6380
```

集群模式：例子显示操作正常。
```
[root@VM_113_88_centos /home/redis]# redis-cli -c -h 172.16.238.11 -p 6379 set test 1
OK
[root@VM_113_88_centos /home/redis]# redis-cli -c -h 172.16.238.11 -p 6379 get test
"1"
```
