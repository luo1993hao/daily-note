

## 为什么使用dubbo

---

> - 服务越来越多-> url配置管理越来越难，硬件负载均衡器越来越大-> 需要一个注册中心，动态的注册和发现服务。
>- 服务之间依赖关系越来越复杂->需要自动的画出应用的依赖关系
>- 服务量越来越大->需要记录调用量，显示时间，调整权重。

###  Dubbo 的架构图解
![](https://i.loli.net/2019/01/16/5c3f119de2570.png)

- 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
- 注册中心，服务提供者，服务消费者三者之间均为长连接
- 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
- 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
- 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者
- 服务提供者无状态，任意一台宕掉后，不影响使用
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复
### 工作原理
![](https://i.loli.net/2019/01/16/5c3f16757f926.png)
- 第一层：service层，接口层，给服务提供者和消费者来实现的
- 第二层：config层，配置层，主要是对dubbo进行各种配置的
- 第三层：proxy层，服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton
- 第四层：registry层，服务注册层，负责服务的注册与发现
- 第五层：cluster层，集群层，封装多个服务提供者的路由以及负载均衡，将多个实例组合成一个服务
- 第六层：monitor层，监控层，对rpc接口的调用次数和调用时间进行监控
- 第七层：protocol层，远程调用层，封装rpc调用
- 第八层：exchange层，信息交换层，封装请求响应模式，同步转异步
- 第九层：transport层，网络传输层，抽象mina和netty为统一接口
- 第十层：serialize层，数据序列化层。网络传输需要。
#### 细节知识
JVM启动-D参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需
改变协议的端口。
1. XML次之，如果在XML中有配置，则dubbo.properties中的相应配置项无效。
2. Properties最后，相当于缺省值，只有XML没有配置时，dubbo.properties的相应配置项
3. 才会生效，通常用于共享公共配置，比如应用名。
4. 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 
```
<dubbo:annotation package="com.foo.bar.service" />
```
5. 关闭所有服务的启动时检查：
没有提供者时报错
```
<dubbo:consumer check="false" />
```
6. 引用缺省是延迟初始化的，只有引用被注入到其它Bean，或被getBean()获取，才会初始化。
如果需要饥饿加载，即没有人引用也立即生成动态代理，可以配置：
```
<dubbo:reference interface="com.foo.BarService" init="true" />
```
7. 直连模式只适合开发及测试环境下
8. zk数据存储目录不能放在/temp中（默认，得改）,否则会引起数据丢失。

9. 不同服务不同协议。比如：不同服务在性能上适用不同协议进行传输，比如大数据用短连接协议，小数据大并发
用长连接协议 

10. 结果缓存，用于加速热门数据的访问速度，Dubbo提供声明式缓存，以减少用户加缓
存的工作量

11. 回声测试用于检测服务是否可用，回声测试按照正常请求流程执行，能够测试整个调
用是否通畅，可用于监控。

所有服务自动实现EchoService接口，只需将任意服务引用强制转型为EchoService，
即可使用。
```
<dubbo:reference id="memberService" interface="com.xxx.MemberService" />
MemberService memberService = ctx.getBean("memberService"); // 远程服务引用
EchoService echoService = (EchoService) memberService; // 强制转型为EchoService
String status = echoService.$echo("OK"); // 回声测试可用性
assert(status.equals("OK"));
```
12. 每次发生rpc调用，上下文都会变化
13. 隐射传参，用于框架集成，不建议使用
14. 本地调用：injvm，每个服务默认都会在本地暴露。在引用服务的时候，默认优先引用本地
服务
15. 一个接口有多种实现的时候，可以用group分组
16. 按组合并返回结果，比如菜单服务，接口一样，但有多种实现，用group区分，现在消
费方需从每种group中调用一次返回结果，合并结果返回，这样就可以实现聚合菜单项

---


---

## 支持的协议
- dubbo(缺省)
  - 单一长链接和nio异步通讯，适合小数据量大并发的服务调用，不适合传送大数据量
- rmi:
- hession:传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件
- HTTP:基于 HTTP 表单的远程调用协议，采用 Spring 的 HttpInvoker 实现,需同时给应用程序和浏览器 JS 使用的服务
- WebService:系统集成，跨语言调用
- thrift:
- memcached ,redis
## 注册中心
- Multicast 注册中心不需要启动任何中心节点，只要广播地址一样，就可以互相发现
组播受网络结构限制，只适合小规模应用或开发阶段使用
- zookeeper
- Redis
## 容错策略
- Failover Cluster（缺省）失败自动切换
- Failfast Cluster：快速失败，只发起一次调用，失败立即报错，一般用于非幂等操作
- Failsafe Cluster：失败安全，出现异常时，直接忽略
- Failback Cluster：
- Forking Cluster：并行调用，只要有一个成功返回。实时性高，浪费资源
- Broadcast Cluster：广播，逐一调用
## 负载均衡
- Random LoadBalance：随机，按权重设置随机概率。
- RoundRobin LoadBalance：轮询，存在慢的提供者累积请求的问题
- LeastActive LoadBalance：最少活跃调用数，
- ConsistentHash LoadBalance ：一致性Hash，相同参数的请求总是发到同一提供者。

