##### 剖析mysql查询
1. 捕获mysql的查询到日志文件中
2. 日志分析工具url
```
https://www.cnblogs.com/luyucheng/p/6265873.html
```
##### 剖析单条数据
1. show profile
2. show status
##### explain详解
```
mysql> explain select * from servers;
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | servers | ALL  | NULL          | NULL | NULL    | NULL |    1 | NULL  |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
row in set (0.03 sec)
```
expain出来的信息有10列，分别是id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra,下面对这些字段出现的可能进行解释：
- id 执行顺序，从大到小执行，子查询会递增
- select_type 每个select子句的类型
  - simple
  - primary
  - dependent primary
- table 查询的哪张表，可能是别名
- type 访问类型 常用的类型有： **ALL, index,  range, ref, eq_ref, const, system, NULL（从左到右，性能从差到好，这也是优化的依据）**
- possible_keys 可能的索引，但是并不一定使用
- key 实际使用的索引
- key_len 索引使用的字节数，索引字段的最大长度，并非实际使用长度，越短越好
- ref 连接条件
- rows 估算的找到所需记录所需要读取的行数
- Extra 解决查询的详细信息
```
Using where:列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤

Using temporary：表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询

Using filesort：MySQL中无法利用索引完成的排序操作称为“文件排序”

Using join buffer：该值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

Impossible where：这个值强调了where语句会导致没有符合条件的行。

Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行
```
总结：
- EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
- EXPLAIN不考虑各种Cache
- EXPLAIN不能显示MySQL在执行查询时所作的优化工作
- 部分统计信息是估算的，并非精确值
- EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划。


### EXPLAIN 实战总结

#### extra
- Using join buffer   在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。
  - (Block Nested Loop) 
  - 
- Using temporary
  - 对驱动表可以直接排序，对非驱动表（的字段排序）需要对循环查询的合并结果（临时表）进行排序（Important!）
- Using filesort 
    - 官方解释：MySQL must do an extra pass to find out how to retrieve the rows in sorted order. The sort is done by going through all rows according to the join type and storing the sort key and pointer to the row for all rows that match the WHERE clause.
    - 一般出现在order by 排序字段没有加索引。mysql会进行一次额外的排序
    - 排序大小：max_length_for_sort_data，如果查询的数据量大于这个值，即便加上索引，还是会进行filesort
 ### left join/join
 