---
title: Docker搭建Redis集群
date: 2020-11-30 11:01:50
tags:
- Docker
- Redis集群
---

# Docker搭建Redis集群

本篇博客主要参考了[这篇文章](https://segmentfault.com/a/1190000021882802)，也有一些小变动，记录一下方便自己之后需要。

## 编辑配置

### 配置模板

 `redis-cluster.tmpl`

```properties
# redis端口
port ${PORT}
# 关闭保护模式
protected-mode no
# 开启集群
cluster-enabled yes
# 集群节点配置
cluster-config-file nodes.conf
# 超时
cluster-node-timeout 5000
# 集群节点IP host模式为宿主机IP
cluster-announce-ip 192.xxx
# 集群节点端口 7001 - 7006
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
# 开启 appendonly 备份模式
appendonly yes
# 每秒钟备份
appendfsync everysec
# 对aof文件进行压缩时，是否执行同步操作
no-appendfsync-on-rewrite no
# 当目前aof文件大小超过上一次重写时的aof文件大小的100%时会再次进行重写
auto-aof-rewrite-percentage 100
# 重写前AOF文件的大小最小值 默认 64mb
auto-aof-rewrite-min-size 64mb
```

### 生成配置脚本 `redis-cluster-config.sh` 

```sh
for port in `seq 7001 7006`; do \
  mkdir -p ./redis-cluster/${port}/conf \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./redis-cluster/${port}/conf/redis.conf \
  && mkdir -p ./redis-cluster/${port}/data; \
done
```

### 利用脚本批量生成配置

```bash
[hzh@localhost redis]$ chmod +x redis-cluster-config.sh
[hzh@localhost redis]$ ./redis-cluster-config.sh
```

```bash
[hzh@localhost redis]$ tree
.
├── redis-cluster
│   ├── 7001
│   │   ├── conf
│   │   │   └── redis.conf
│   │   └── data
│   ├── 7002
│   │   ├── conf
│   │   │   └── redis.conf
│   │   └── data
│   ├── 7003
│   │   ├── conf
│   │   │   └── redis.conf
│   │   └── data
│   ├── 7004
│   │   ├── conf
│   │   │   └── redis.conf
│   │   └── data
│   ├── 7005
│   │   ├── conf
│   │   │   └── redis.conf
│   │   └── data
│   └── 7006
│       ├── conf
│       │   └── redis.conf
│       └── data
├── redis-cluster-config.sh
```

## 生成容器

docker-compose 配置批量

`docker-compose-redis-cluster.yml`

```yml
version: '3.7'

services:
  redis7001:
    image: 'redis'
    container_name: redis7001
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7001/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7001/data:/data
    ports:
      - "7001:7001"
      - "17001:17001"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai

  redis7002:
    image: 'redis'
    container_name: redis7002
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7002/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7002/data:/data
    ports:
      - "7002:7002"
      - "17002:17002"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai

  redis7003:
    image: 'redis'
    container_name: redis7003
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7003/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7003/data:/data
    ports:
      - "7003:7003"
      - "17003:17003"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai

  redis7004:
    image: 'redis'
    container_name: redis7004
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7004/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7004/data:/data
    ports:
      - "7004:7004"
      - "17004:17004"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai

  redis7005:
    image: 'redis'
    container_name: redis7005
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7005/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7005/data:/data
    ports:
      - "7005:7005"
      - "17005:17005"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai

  redis7006:
    image: 'redis'
    container_name: redis7006
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7006/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7006/data:/data
    ports:
      - "7006:7006"
      - "17006:17006"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
```

### 执行生成命令

```bash
[hzh@localhost redis]$ docker-compose -f docker-compose-redis-cluster.yml up -d
bash: docker-compose: command not found...
[hzh@localhost redis]$ pip3 install --user docker-compose  # 安装docker-compose
```

再次执行

```bash
[hzh@localhost redis]$ docker-compose -f docker-compose-redis-cluster.yml up -d
Creating network "redis_default" with the default driver
Creating redis7003 ... done
Creating redis7001 ... done
Creating redis7004 ... done
Creating redis7002 ... done
Creating redis7006 ... done
Creating redis7005 ... done
[hzh@localhost redis]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
                                              NAMES
8cab1789f565        redis               "docker-entrypoint.s…"   24 seconds ago      Up 23 seconds       0.0.0.0:7005->7005/tcp, 6379/tcp, 0.0.0.0:17005->17005/tcp   redis7005
1743591a742f        redis               "docker-entrypoint.s…"   24 seconds ago      Up 23 seconds       0.0.0.0:7006->7006/tcp, 6379/tcp, 0.0.0.0:17006->17006/tcp   redis7006
dd5c4405833f        redis               "docker-entrypoint.s…"   24 seconds ago      Up 23 seconds       0.0.0.0:7003->7003/tcp, 6379/tcp, 0.0.0.0:17003->17003/tcp   redis7003
f3d813dfbfdd        redis               "docker-entrypoint.s…"   24 seconds ago      Up 23 seconds       0.0.0.0:7001->7001/tcp, 6379/tcp, 0.0.0.0:17001->17001/tcp   redis7001
269f81bfcd6b        redis               "docker-entrypoint.s…"   24 seconds ago      Up 23 seconds       0.0.0.0:7002->7002/tcp, 6379/tcp, 0.0.0.0:17002->17002/tcp   redis7002
9b978437186d        redis               "docker-entrypoint.s…"   24 seconds ago      Up 23 seconds       0.0.0.0:7004->7004/tcp, 6379/tcp, 0.0.0.0:17004->17004/tcp   redis7004
```

## 开启集群

```bash
docker exec -it redis7001 redis-cli -p 7001 --cluster create 59.78.194.153:7001 59.78.194.153:7002 59.78.194.153:7003 59.78.194.153:7004 59.78.194.153:7005 59.78.194.153:7006 --cluster-replicas 1
```

可能会报错，这里是因为端口没有开放。记得不仅要开放7001-7006端口，还需要开放17001-17006端口，不然开启集群时会卡在Waiting for the cluster to join 。因为1700X端口是集群通信端口。

```bash
[hzh@localhost redis]$ su
Password:
(base) [root@localhost redis]# firewall-cmd --zone=public --add-port=7001/tcp --permanent
success
......
(base) [root@localhost redis]# firewall-cmd --zone=public --add-port=7006/tcp --permanent
success
......
(base) [root@localhost redis]# firewall-cmd --zone=public --add-port=17006/tcp --permanent
success
(base) [root@localhost redis]# firewall-cmd --reload
success
```

再次执行

```bash
[hzh@localhost redis]$ docker exec -it redis7001 redis-cli -p 7001 --cluster create 59.78.194.153:7001 59.78.194.153:7002 59.78.194.153:7003 59.78.194.153:7004 59.78.194.153:7005 59.78.194.153:7006 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 59.78.194.153:7005 to 59.78.194.153:7001
Adding replica 59.78.194.153:7006 to 59.78.194.153:7002
Adding replica 59.78.194.153:7004 to 59.78.194.153:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: e2cd40d097ea688af18cd022883fa13a8afd5ef3 59.78.194.153:7001
   slots:[0-5460] (5461 slots) master
M: a698698fbe9ce85e7448bcb27ba5f19f0d175207 59.78.194.153:7002
   slots:[5461-10922] (5462 slots) master
M: 51f55fa1094c0eed9a2ff8121a4923e9a86ff0d4 59.78.194.153:7003
   slots:[10923-16383] (5461 slots) master
S: 14f8fbba8b2e9623865dd9453f22b63bd914ef13 59.78.194.153:7004
   replicates a698698fbe9ce85e7448bcb27ba5f19f0d175207
S: 003c2b6cad73550648f93ba3257abcf085b347e0 59.78.194.153:7005
   replicates 51f55fa1094c0eed9a2ff8121a4923e9a86ff0d4
S: 112ce7ce41c2930f6ce8507ffe3880e36e319cac 59.78.194.153:7006
   replicates e2cd40d097ea688af18cd022883fa13a8afd5ef3
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 59.78.194.153:7001)
M: e2cd40d097ea688af18cd022883fa13a8afd5ef3 59.78.194.153:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: a698698fbe9ce85e7448bcb27ba5f19f0d175207 59.78.194.153:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 14f8fbba8b2e9623865dd9453f22b63bd914ef13 59.78.194.153:7004
   slots: (0 slots) slave
   replicates a698698fbe9ce85e7448bcb27ba5f19f0d175207
S: 112ce7ce41c2930f6ce8507ffe3880e36e319cac 59.78.194.153:7006
   slots: (0 slots) slave
   replicates e2cd40d097ea688af18cd022883fa13a8afd5ef3
S: 003c2b6cad73550648f93ba3257abcf085b347e0 59.78.194.153:7005
   slots: (0 slots) slave
   replicates 51f55fa1094c0eed9a2ff8121a4923e9a86ff0d4
M: 51f55fa1094c0eed9a2ff8121a4923e9a86ff0d4 59.78.194.153:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

## 简单测试

```bash
> 59.78.194.153@7001 connected!
> set name admin
OK
> get name
admin
```

```bash
> 59.78.194.153@7002 connected!
> get name
admin
```

查看集群状态

```bash
> 59.78.194.153@7001 connected!
> cluster nodes
e2cd40d097ea688af18cd022883fa13a8afd5ef3 59.78.194.153:7001@17001 master - 0 1606736718359 1 connected 0-5460
14f8fbba8b2e9623865dd9453f22b63bd914ef13 59.78.194.153:7004@17004 slave a698698fbe9ce85e7448bcb27ba5f19f0d175207 0 1606736718000 2 connected
003c2b6cad73550648f93ba3257abcf085b347e0 59.78.194.153:7005@17005 slave 51f55fa1094c0eed9a2ff8121a4923e9a86ff0d4 0 1606736718560 3 connected
112ce7ce41c2930f6ce8507ffe3880e36e319cac 59.78.194.153:7006@17006 slave e2cd40d097ea688af18cd022883fa13a8afd5ef3 0 1606736716854 1 connected
a698698fbe9ce85e7448bcb27ba5f19f0d175207 59.78.194.153:7002@17002 myself,master - 0 1606736717000 2 connected 5461-10922
51f55fa1094c0eed9a2ff8121a4923e9a86ff0d4 59.78.194.153:7003@17003 master - 0 1606736718560 3 connected 10923-16383
```



## 参考

https://segmentfault.com/a/1190000021882802

https://blog.csdn.net/XIANZHIXIANZHIXIAN/article/details/82392172