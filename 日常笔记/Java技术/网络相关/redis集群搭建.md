# redis集群搭建

## 搭建过程
### 1. 下载redis
### 2. 拷贝redis.conf文件
实例

```
mkdir -p /usr/local/cluster
mkdir -p /usr/local/cluster/7000
mkdir -p /usr/local/cluster/7001
mkdir -p /usr/local/cluster/7002
mkdir -p /usr/local/cluster/7003
mkdir -p /usr/local/cluster/7004
mkdir -p /usr/local/cluster/7005
cp -rf /usr/local/redis/redis.conf  /usr/local/cluster/7000/
cp -rf /usr/local/redis/ redis.conf  /usr/local/cluster/7001/
cp -rf /usr/local/redis/ redis.conf  /usr/local/cluster/7002/
cp -rf /usr/local/redis/ redis.conf  /usr/local/cluster/7003/
cp -rf /usr/local/redis/ redis.conf  /usr/local/cluster/7004/
cp -rf /usr/local/redis/ redis.conf  /usr/local/cluster/7005/
```
### 修改redis.conf文件
blind 127.0.0.1 修改为blind 本机Ip

port 端口号

daemonize yes

cluster-enabled yes

 cluster-config-file nodes.conf
 
cluster-node-timeout 5000

appendonly yes

**有些修改部分用#注释了，记得去掉#**

**不同redis.conf只有port与blind不同**
### 启动redis
进入src目录。./redis-server /(conf文件目录)/redis.conf
六个都启动

**启动之后使用命令查看redis的启动情况ps –ef | grep redis**

###  安装集群所需配置

#### 1.需要安装ruby的环境，这里推荐使用yum install ruby
命令如下：
yum install ruby
#### 2. 安装rubygems组件
命令如下：
yum install rubygems
#### 3. 下载安装gem服务器
```
wget https://rubygems.global.ssl.fastly.net/gems/redis-3.2.1.gem
```
```
gem install -l ./redis-3.2.1.gem
```
#### 4.创建redis集群
进入redis目录的src文件夹下

```bash
./redis-trib.rb  create --replicas 1 172.31.1.60:7000 172.31.1.60:7001 172.31.1.60:7002 172.31.1.60:7003 172.31.1.60:7004 172.31.1.60:7005
（所有ip与端口号）
```
出现以下信息

```
>>> Creating cluster
Connecting to node 172.31.1.60:7000: OK
Connecting to node 172.31.1.60:7001: OK
Connecting to node 172.31.1.60:7002: OK
Connecting to node 172.31.1.60:7003: OK
Connecting to node 172.31.1.60:7004: OK
Connecting to node 172.31.1.60:7005: OK
>>> Performing hash slots allocation on 6 nodes...

Using 3 masters:
172.31.1.60:7000
172.31.1.60:7001
172.31.1.60:7002
Adding replica 172.31.1.60:7003 to 172.31.1.60:7000
Adding replica 172.31.1.60:7004 to 172.31.1.60:7001
Adding replica 172.31.1.60:7005 to 172.31.1.60:7002
M: 2022f24d581b4a7c3342e3245c32927cbd5ec16d 172.31.1.60:7000
   slots:0-5460 (5461 slots) master
M: 37b7008f80f8c21a698da8cb1f1b32db8c0c415c 172.31.1.60:7001
   slots:5461-10922 (5462 slots) master
M: ac6dc5fa96e856b34c1ba4c3814394e4ebb698dd 172.31.1.60:7002
   slots:10923-16383 (5461 slots) master
S: b5b76d70bbb0dbf3e7df8a38f1259e95e2054721 172.31.1.60:7003
   replicates 2022f24d581b4a7c3342e3245c32927cbd5ec16d
S: 6881f8fef9c25da486f320ebf2ead39c1502db4c 172.31.1.60:7004
   replicates 37b7008f80f8c21a698da8cb1f1b32db8c0c415c
S: f090526d32cced97731eef2a2e1722a7bac7d9ea 172.31.1.60:7005
   replicates ac6dc5fa96e856b34c1ba4c3814394e4ebb698dd
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 172.31.1.60:7000)
M: 2022f24d581b4a7c3342e3245c32927cbd5ec16d 172.31.1.60:7000
   slots:0-5460 (5461 slots) master
M: 37b7008f80f8c21a698da8cb1f1b32db8c0c415c 172.31.1.60:7001
   slots:5461-10922 (5462 slots) master
M: ac6dc5fa96e856b34c1ba4c3814394e4ebb698dd 172.31.1.60:7002
   slots:10923-16383 (5461 slots) master
M: b5b76d70bbb0dbf3e7df8a38f1259e95e2054721 172.31.1.60:7003
   slots: (0 slots) master
   replicates 2022f24d581b4a7c3342e3245c32927cbd5ec16d
M: 6881f8fef9c25da486f320ebf2ead39c1502db4c 172.31.1.60:7004
   slots: (0 slots) master
   replicates 37b7008f80f8c21a698da8cb1f1b32db8c0c415c
M: f090526d32cced97731eef2a2e1722a7bac7d9ea 172.31.1.60:7005
   slots: (0 slots) master
   replicates ac6dc5fa96e856b34c1ba4c3814394e4ebb698dd
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

## 注意事项

### 1.主从与集群区别
集群：
1 将数据自动切分(split)到多个节点

1 当集群中的某一个节点故障时，redis还可以继续处理客户端的请求。

 主从：
redis的复制功能是支持多个数据库之间的数据同步。一类是主数据库（master）一类是从数据库（slave），主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据，一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。

通过redis的复制功能可以很好的实现数据库的读写分离，提高服务器的负载能力。主数据库主要进行写操作，而从数据库负责读操作。

### 2.搭建redis集群最少6个节点