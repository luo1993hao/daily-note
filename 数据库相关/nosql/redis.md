### 数据类型
- String  
```
> set hello world
OK
> get hello
"world"
```
- set
- list
- hash
- zset
### 使用场景
- 计数器
- 缓存
- 缓存session
- 排行榜
- 分布式锁
### 数据淘汰策略
- volatile-lru	从已设置过期时间的数据集中挑选最近最少使用的数据淘汰
- volatile-ttl	从已设置过期时间的数据集中挑选将要过期的数据淘汰
- volatile-random	从已设置过期时间的数据集中任意选择数据淘汰
- allkeys-lru	从所有数据集中挑选最近最少使用的数据淘汰
- allkeys-random	从所有数据集中任意选择数据进行淘汰
- noeviction	禁止驱逐数据
### 持久化
- RBD持久化
- AOF持久化
### 事务
 MULTI 和 EXEC 命令将事务操作包围起来
 