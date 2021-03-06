### redis基础数据结构
- String
  - 用户缓存
- list
  - linkList,插入删除时间复杂度o1,索引定位慢，on
  - 异步队列
  - ziplist+链表->quicklist
- set
  - 中奖用户信息
- hash
  - 渐进式rehash
  - 存储用户信息，部分获取。但结构的存储高于单个字符串
- zset
  - sortedSet+hashMap:value唯一。每个value有一个score,排序权重
  - 跳跃列表
- list/set/hash/zset 这四种数据结构是容器型数据结构
  - create if not exists
  - drop if no elements
### 应用
#### 分布式锁
- 先setnx。然后delete
  - 如果delete没有调用，死锁
     - 增加过期时间:setnx->expire 5s->delete：如果expire 5s没执行还是死锁，原因setnx,expire不是原子操作
  - Redis 2.8 版本中作者加入了 set 指令的扩展参数，使得 setnx 和 expire 指令可以一起执行
- 超时问题
 - redis分布式锁不能解决超时问题。
   - set时候，设置一个value随机数，先匹配。->使用lua脚本来保证多个指令的原子性
- 可重入性
  - 使用threadLocal包装set
- 分布式环境下redlock算法
  - 大多数机制
  - 场景：高可用性。性能下降
#### 延时队列
- 只有一组消费者
- 移步消息list：lpush 和 rpop
  - 队列空： blpop/brpop -> lpop/rpop
  - 空闲连接自动断开：捕获异常，重试
- 延时队列：zset
   - 消息序列化成一个字符串作 为 zset 的 value，这个消息的到期处理时间作为 score，然后用多个线程轮询 zset 获取到期 的任务进行处理
#### 位图
#### HyperLogLog
- 不精确的去重计数方案
  -  pfadd 和 pfcount pfmerge
#### 布隆过滤器
- 不怎么精确的set,存在，可能不存在。不存在。一定不存在
- 场景：给用户推荐新闻，爬虫系统。邮箱系统垃圾邮件过滤
- bf.add,bff.exists/bf.madd,bf.mexists
- 自定义参数：bf.reserve：key, error_rate 和 initial_size
- 大型的位数组和几个不一样的无 偏 hash 函数
![](https://i.loli.net/2020/08/18/l7uPmSnABM18Tde.png)
#### 简单限流
- 限定用户的某个行为在指定的时间里 只能允许发生 N 次
  - 滑动时间窗口
  - zset:key:用户行为 value:时间戳 score:时间戳。
  - 移除当前时间戳之前的行为，获取窗口内的数量，比较阈值
  - 记录窗口所有行为，适用于量不是很大的场景
#### 漏斗限流
- redis-cell:使用了漏斗算法，并 提供了原子的限流指令
#### GeoHash
- 场景：附近的xxx。zset
- Geo 的数据使用单独的 Redis 实例部署，不使用集群环境
#### scan
- keys算法是遍历算法。on
- redis的内存大起大落->大key->定位大key->bigkeys
- scan 指令是一系列指令.zscan:zset.hscan:hash
### 原理
#### 线程io模型
- 非阻塞Io
  - 事件轮询api。称为多路服复用api
  - 事件轮询 API 就是 Java 语言里面的 NIO 技术
- 定时任务
  - 最小堆
#### 通信协议
- RESP(Redis Serialization Protocol)
  - 单行字符串 以 + 符号开头。
  - 多行字符串 以 $ 符号开头，后跟字符串长度。
  - 整数值 以 : 符号开头，后跟整数的字符串形式。
  - 错误消息 以 - 符号开头。
  - 数组 以 * 号开头，后跟数组的长度。
#### 持久化
- 快照
  - 全量备份
  - Redis 使用操作系统的多进程 COW(Copy On Write) 机制来实现快照持久化
  - glibc函数fork子进程。快照由子进程处理。父进程处理客户端请求。当父进程对其中一个页面的数据进行修改时，会将被共享的页面复 制一份分离出来，然后对这个复制的页面进行修改。进程相应的页面是没有变化的， 还是进程产生时那一瞬间的数据。
- aof日志
  - 连续增量备份
  - 存储的是 Redis 服务器的顺序指令序列，AOF 日志只记录对内存进行修改的 指令记录。也就是“重放”
- 混合持久化
  - 将 rdb 文 件的内容和增量的 AOF 日志文件存在一起。这里的 AOF 日志不再是全量的日志，而是自 持久化开始到持久化结束的这段时间发生的增量 AOF 日志
#### 管道
![](https://i.loli.net/2020/08/18/nptkgEyUvLma8WG.png)
- 客户端通过改变了读写的顺序 带来的性能的巨大提升
#### 事务
- Redis 的事务根本不能算「原子性」，而仅仅是满足了事务的「隔 离性」
- multi/exec/discard。multi 指示事务的开始，
exec 指示事务的执行，discard 指示事务的丢弃。
- watch.乐观锁。multi之前使用。盯住关键变量
#### 主从同步
- CAP 原理就是:网络分区发生时，一致性和可用性两难全。
- 修改指令->本地内存buffer（定长环形数组）->异步同步到从节点。从节点反馈偏移量
- 快照同步
 ![](https://i.loli.net/2020/08/18/eT4lYBdEiyqCb1Q.png)
- 无盘复制:指主服务器直接通过套接字 将快照内容发送到从节点 
### 集群
#### Redis Sentinel
- Redis Sentinel 集群看成是一个 ZooKeeper 集群
- 门面+调度器
  - 当主节点挂掉时，自动选择一个最优的从节点切换为 主节点
  - 客户端来连接集群时，会首先连接 sentinel，通过 sentinel 来查询主节点的地址
  - 主节点发生故障时，客户端会重新向 sentinel 要地 址，sentinel 会将最新的主节点地址告诉客户端
  - Sentinel 会持续监控已经挂掉了主节点
  - Sentinel 无法保证消息完全不丢失，但是也尽可能保证消息少丢失
#### codis
- 转发代理中间件
- 默认key分为1024个slot（可配置）.key->crc32->mod 1024  
- 将槽位关系存储在 zk 中
- 自动均衡slot
- 优点
  - 简单
- 缺点
   - 不支持事务
   - 单个value不能过大
   - 非官方，更新慢
#### cluster
- 去中心化
- Redis Cluster 将所有数据划分为 16384 的 slots。每个节点负责一部分slot
- Redis Cluster 的客户端来连接集群时，它也会得到一份集群的槽位配置信息
- 迁移
   - redis-trib
   - 同步。从源节点获取内容 => 存到目标节点 => 从源节点删除内容。
   - 两个特殊的 error 指令，一个是 moved，一个是 asking。
- 容错
  - cluster-require-full-coverage 可以允许部分 节点故障
  - 单主节点故障时，集群会自动将其 中某个从节点提升为主节点
 #### info
- 每秒执行多少指令：info stats->instantaneous_ops_per_sec
- monitor 指令快速观察一下究竟是哪些 key 访问比较频繁
- 连接多少客户端：info clients
- 内存占用多大：info memeory->user_memeory_xxxx
- 复制积压缓冲区：info replication->repl_backlog_size
#### 过期策略
- 定时删除
  - 设置过期时间key放在一个集合
  - 从过期字典中随机 20 个 key;
  - 删除这 20 个 key 中已经过期的 key; 
  - 如果过期的 key 比率超过 1/4，那就重复步骤 1;
- 惰性处理
  - 访问key的时候。如果过期。再删除
- 大批量的 key 过期，要给过期时间设置 一个随机范围，而不能全部在同一时间过期。
- 从库通过aof指令删除
#### 懒惰删除
- Redis 内部实际上并不是只有一个主线程，它还有几个异步线程专门用来 处理一些耗时的操作
- aof sync
- unlink替代flush
#### 保护redis
- 指令安全
 - rename-command
- 端口安全
  - 指定端口，密码，从库密码
- lua脚本安全
   - 禁止 Lua 脚本由用户输入的内容 (UGC) 生成
   - redis以普通用户登陆
#### Redis 安全通信
![](https://i.loli.net/2020/08/19/OYDh72sJymUqRSF.png)