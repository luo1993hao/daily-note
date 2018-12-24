
# 原理
 - MySQL的主从复制是一个异步的复制过程（虽然一般情况下感觉是实时的），数据将从一个Mysql数据库（我们称之为Master）复制到另一个Mysql数据库（我们称之为Slave），在Master与Slave之间实现整个主从复制的过程是由三个线程参与完成的。其中有两个线程（SQL线程和IO线程）在Slave端，另一个线程（I/O线程）在Master端。
 主上有一个log dump线程，用来和从的I/O线程传递binlog
 
 从上有两个线程，其中I/O线程用来同步主的binlog并生成relaylog，另外一个SQL线程用来把relaylog里面的sql语句执行一遍

-  要实现MySQL的主从复制，首先必须打开Master端的binlog记录功能，否则就无法实现。因为整个复制过程实际上就是Slave从master端获取binlog日志，然后再在Slave上以相同顺序执行获取的binlog日志中的记录的各种SQL操作
 

# MYSQL数据库同步流程  

**本次流程中11为主节点，12为从节点**
**前提是主从数据库已经建好**
# master节点
## 1.修改配置mysql主配置文件
```
vim /usr/my.cnf
```

```
log-bin=mysql-bin 	#启动二进制日志系统
binlog-do-db=test 	#二进制需要同步的数据库名,如果需要同步多个库,例如要再同步 westos
		         库,再添加一行“binlog-do-db=westos”,以此类推
server-id=1       	#必须为 1 到 232–1 之间的一个正整数值
binlog-ignore-db=mysql	 #禁止同步 mysql 数据库

    systemctl restart mysqld.service    #重启数据库（cen7 cen6中为service mysqld restart）
    show master status;   # 查看状态
```
## 2.新建一个用户用于同步
```
GRANT REPLICATION SLAVE ON *.* TO westos@'192.168.0.12'IDENTIFIED BY 'westos';
```

# slave节点
修改配置文件新增
```
server-id=2(与master不一致)
然后重启

```
```bash
mysql> change master to master_host='192.168.0.11', master_user='westos',
master_password='westos', master_log_file='mysql-bin.000001', master_log_pos=106;

mysql> start slave;
mysql> show slave status\G;


重点是
    Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            二者必须为yes

```
#### 至此，正常的主从已经完成



## 3.添加一个需要同步的新库
### 1. 从服务上，停掉slave数据库。
```
stop slave;
```
### 2. 主服务器上，导出新数据库。
```
mysqldump -uroot -p --master-data 
--single-transaction -R 
--databases [newdb] > newdb.sql
```
### 3. 修改主服务器my.cnf文件

主服务器上，修改my.cnf文件，添加新库到binlog-do-db
参数，重启mysql。
```
/etc/init.d/mysql restart
```
### 4. 查找当前的日志文件以及位置

在导出的newdb.sql里面查找当前的日志文件以及位置（change master to …)
然后让slave服务器执行到这个位置。
```
start slave until
MASTER_LOG_FILE="mysql-bin.000001",
MASTER_LOG_POS=1222220;
```
```
stop slave
```
其中MASTER_LOG_FILE以及MASTER_LOG_POS在导出的数据库newdb.sql顶部位置查找。

### 5. 导入新库到从服务器上。

mysql < newdb.sql

### 6. 启动从服务器
```
start slave
```

数据库复制   一个时间段：
```
mysqlbinlog  --start-datetime='2016-10-18 15:26:33' 
mysql-bin.000009 | mysql -pbuzhidao
```
### 7.报错
Slave_SQL_Running: No
1.程序可能在slave上进行了写操作
2.也可能是slave机器重起后，事务回滚造成的.

一般是事务回滚造成的：
解决办法：
```
mysql> slave stop;
mysql> set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
mysql> slave start;
```
解决办法二、
首先停掉Slave服务：slave stop
到主服务器上查看主机状态：
记录File和Position对应的值
进入master
mysql> show master status;

然后到slave服务器上执行手动同步：
```
mysql> change master to
> master_host='master_ip',
> master_user='user',
> master_password='pwd',
> master_port=3306,
> master_log_file=localhost-bin.000094',
> master_log_pos=33622483 ;
```