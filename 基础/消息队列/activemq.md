### jms
Java Message Service，java ee规范之一。消息的发送应该是异步的、非阻塞的JMS只是Java EE中定义的一组标准API，它自身并不是一个消息服务系统，它是消息传送服务的一个抽象，也就是说它定义了消息传送的接口而并没有具体实现
类似于jdbc
### 基本知识
基本要素:1、生产者producer ; 2、消费者consumer ; 3、消息服务broker
消费消息有2种方法，一种是调用consumer.receive()方法，该方法将阻塞直到获得并返回一条消息。这种情况下，消息返回给方法调用者之后就自动被确认了。另一种方法是采用listener回调函数，在有消息到达时，会调用listener接口的onMessage方法。在这种情况下，在onMessage方法执行完毕后，消息才会被确认，此时只要在方法中抛出异常，该消息就不会被确认。

####  交互模型
1. 点对点
```
session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE); 
点对点时，创建session时，设置是否使用事务以及确认策略
```
2. 发布/订阅
![](https://i.loli.net/2018/12/27/5c24b9ad1ddb6.png)

#### 确认策略
- AUTO_ACKNOWLEDGE:自动确认 
- CLIENT_ACKNOWLEDGE:客户端确认 
- SESSION_TRANSACTED:事务确认,如果使用事务推荐使用该确认机制 
- AUTO_ACKNOWLEDGE:懒散式确认,消息偶尔不会被确认,也就是消息可能会被重复发送.但发生的概率很小 
#### 消息类型
1. TextMessage
2. ObjectMessage
3. MapMessage
4. BytesMessage
5. StreamMessage
#### 消息持久化
ActiveMQ默认是持久化的，消息在没有被消费前都会被写入本地磁盘kahadb文件中保存起来。

通常的情况下，非持久化消息是存储在内存中的。内存告急的时候。持久化消息是存储在文件
无论是持久化消息还是非持久化消息都会写入文件，重启后，非持久化消息的文件会被删除。
- 持久化为kahadb文件
- 持久化为mysql
- 持久化oracle


### 问题

1. 丢消息

原因：心跳机制，数据大量，服务器->客户端。超过时间没回答，抛异常，缓冲区数据丢失
解决方案：用事务，事务策略会负责任的等待服务器的返回.。用持久化消息

2. 消息不均匀
3. 死信队列
4. 服务器宕机