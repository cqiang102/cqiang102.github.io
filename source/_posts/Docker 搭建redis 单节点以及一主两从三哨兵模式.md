---
title: Docker 搭建redis单节点以及一主两从三哨兵模式
date: 2019-10-24 23:46:16
categories: 笔记
tags: [linux,redis,Docker,Spring boot]
type: posts
description: Docker 搭建 redis 集群 ，Spring boot 配置 redis
---

## redis 单节点部署 
#### docker-compose.yml
```yaml
version: '3.1'
services:
  redis:
    image: redis
    container_name: my_redis
    restart: always
    command: redis-server --requirepass your_password
    ports:
      - 6379:6379
    volumes:
      - ./data:/data
```
#### 使用Spring boot 连接配置application.yml
```yaml
spring:
  application:
    name: porject-name
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password: 123456
    timeout: 200
    jedis:
      pool:
        max-idle: 8
        min-idle: 0
        max-wait: -1
        max-active: 8
```
## redis 集群部署 一主两从三哨兵
### 一主两从 docker-compose.yml
```yaml
version: '3.7'
services:
  master:
    image: redis
    container_name: redis-master
    restart: always
    command: redis-server --port 6379 --requirepass your_password  --appendonly yes
    ports:
      - 6379:6379
    volumes:
      - ./data:/data
 
  slave1:
    image: redis
    container_name: redis-slave-1
    restart: always
    command: redis-server --slaveof redis-master 6379  --requirepass your_password --masterauth your_password  --appendonly yes
    ports:
      - 6380:6379
    volumes:
      - ./data:/data
 
 
  slave2:
    image: redis
    container_name: redis-slave-2
    restart: always
    command: redis-server --slaveof redis-master 6379  --requirepass your_password --masterauth your_password  --appendonly yes
    ports:
      - 6381:6379
    volumes:
      - ./data:/data
```
#### 检查 redis 是否部署成功
```
# 使用此命令进入docker 容器
docker exec -it redis-master bash
# 使用redis 客户端连接redis
redis-cli
# 认证
auth your_password
# 添加一条数据
set test1 test
# 再取出来
get test1

# 操作全部没问题就可以退出 master
exit
# 在以同样的方式进入 从服务器 测试是否可以拿到刚添加的数据 且不能使用 set 添加数据，说明 一主两从搭建成功
get test1
```
---
### 三哨兵 docker-compose.yml
```yaml
version: '3.7'
services:
  sentinel1:
    image: redis
    container_name: redis-sentinel-1
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: always
    ports:
      - 26379:26379
    volumes:
      - ./sentinel1.conf:/usr/local/etc/redis/sentinel.conf
 
  sentinel2:
    image: redis
    container_name: redis-sentinel-2
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: always
    ports:
      - 26380:26379
    volumes:
      - ./sentinel2.conf:/usr/local/etc/redis/sentinel.conf
 
  sentinel3:
    image: redis
    container_name: redis-sentinel-3
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: always
    ports:
      - 26381:26379
    volumes:
      - ./sentinel3.conf:/usr/local/etc/redis/sentinel.conf
```
#### 三哨兵 配置文件 sentinel.conf
```
port 26379
dir /tmp
# 自定义集群名，其中 192.168.8.188 为 redis-master 的 ip，6379 为 redis-master 的端口，2 为最小投票数（因为有 3 台 Sentinel 所以可以设置成 2）
sentinel monitor mymaster 192.168.8.188 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster your_password
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes

# 复制三份
cp sentinel.conf sentinel1.conf
cp sentinel.conf sentinel2.conf
cp sentinel.conf sentinel3.conf
```

#### 检查是否可用
```
# 进入sentinel
docker exec -it redis-sentinel-1 bash
# 连接 redis-sentinel
redis-cli -p 26379
# 查看从节点状态
sentinel slaves mymaster

# 查看到每个从节点都连接到主节点，就是正常的
#   31) "master-link-status"
#   32) "ok"

# 可以尝试把主节点停止，可以观察sentinel日志，发现sentinel 会重新选举主节点
docker-compose logs -f
docker stop redis-master

```
#### 使用Spring boot 连接配置application.yml
```yaml
spring:
  application:
    name: porject-name
  redis:
    database: 0
    password: your_password
    timeout: 200
    jedis:
      pool:
        # 连接池中的最大空闲连接
        max-idle: 8
        # 连接池中的最小空闲连接
        min-idle: 0
        # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1
        # 连接池最大连接数,默认8个，（使用负值表示没有限制）
        max-active: 8
    sentinel:
      master: mymaster
      nodes: 127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381
```
