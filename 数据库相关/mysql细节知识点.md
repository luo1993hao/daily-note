#### having与where的区别
- 执行顺序：where>group by >having

HAVING子句可以让我们筛选成组后的各组数据，WHERE子句在聚合前先筛选记录．也就是说作用在GROUP BY 子句和HAVING子句前；而 HAVING子句在聚合后对组记录进行筛选
- 使用场景：当分组筛选的时候 用having，其它情况用where
- 用having就一定要和group by连用，用group by不一有having 
- 一般来说使用where优先级大于having 
  - Where 子句是用来指定 "行" 的条件的，而Having 子句是指定 “组” 的条件的，减少排序的行数，可以增加处理速度
  - Where子句指定条件所对应的列创建索引
#### in与exist
区分in和exists主要是造成了驱动顺序的改变（这是性能变化的关键），如果是exists，那么以外层表为驱动表，先被访问，如果是IN，那么先执行子查询。所以IN适合于外表大而内表小的情况；EXISTS适合于外表小而内表大的情况。