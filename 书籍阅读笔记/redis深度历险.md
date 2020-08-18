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