### Spring Cloud Eureka
- 服务治理功能
  - 服务注册中心（单节点，高可用）
  - 服务提供者
  - 服务消费者
  - Region
    - Zone
- 配置
  - 客户端配置
  - Eureka客户端的配置主要分为以下两个方面。
    -  服务注册相关的配置信息， 包括服务注册中心的地址、 服务获取的间隔时间、 可用 区域等。
     -  服务实例相关的配置信息， 包括服务实例的名称、IP地址、 端口号、 健康检查路径  
  - 服务注册类配置
    - 指定注册中心 
    - 其他配置：EurekaClient一 ConfigBean
  - 服务实例类配置
    - 实例名配置
    - 端点配置
    - 健康检测
    - 其他配置：EurekainstanceConfigBean
### Spring cloud Ribbon
- 使用方式
  - 服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务 注册中心。
  - 服务消费者直接通过调用被@LoadBalanced 注解修饰过的 RestTemplate 来实现面 向服务的接口调用。
- RestTemplate
  - 对象会使用 Ribbon 的自动化配置， 同时通过配 置 @LoadBalanced 还能够开启客户端负载均衡
  - get
  - post
  - put
  - delete
- 负载均衡策略
  - Random Rule
  - RoundRobinRule：线性轮询
  - RetryRule
  - WeightedResponseTimeRule
    - 实例运行情况计算权重
  - AvailabilityFilteringRule:先过滤清单，再轮询选择
- 自动化配置
  - IClientConfig:ribbon客户端配置
  - IRule: Ribbon 的负载均衡策略
  - IPing:Ribbon的实例检查策略,默认所有实例都是可用的
  - ServerList<Server>: 服务实例清单的维护机制
  - ServerListFilter<Server>: 服务实例清单过滤机制 
  - ILoadBalancer: 负载均衡器
- 参数配置
  - 全局配置
  - 客户端配置
  - CommonClientConfigKey
### Spring cloud Hystrix
- 服务容错保护
- HystrixCommand
  - 单个返回结果
  - execute(),同步，错误抛异常
- HystrixObservableCommand
  - 多个操作结果
  - observe(),Hot Observable。
    - 它不论 ” 事件源 ” 是否有 “ 订阅者 ”， 都会在创建后对事件 进行发布， 所以对于Hot Observable的每一个“订阅者” 都有可能是从“事件源” 的中途 开始的， 并可能只是看到了整个操作的局部过程。
  - toObservable(),Cold Observable。
    - 在没有“订阅者” 的 时候并不会发布事件， 而是进行等待， 直到有 “ 订阅者 ” 之后才发布事件， 所以对于 Cold Observable 的订阅者， 它可以保证从一 开始看到整个操作的全部过程
- 服务降级
  - 重载 getFallback ()方法
  - 注解实现服务降级只需要使用@HystrixCommand 中的 fallbackMethod 参数来指定具体的服务降级实现方法
- 请求缓存
  -重载 getCacheKey()
- 请求合并
- 属性详解
   - 配置级别
     - 全局默认值
     - 全局配置属性
     - 实例默认值
     - 实例配置属性
### Spring cloud Feign
- 定义
 - 在SpringCloudFeign的实现下， 我们只需创建一个接口并用 注解的方式来配置它，即可完成对服务提供方的接口绑定，简化了在使用 Spring Cloud 伈bbon时自行封装服务调用客户端的开发量
### Spring cloud Zuul
 - 解决的问题
    - 运维人员：降低维护路由规则与服务实例列表的难度 
    - 开发人员：接口访问时各前置冗余问题（校验）
 - 类似于门面模式
   - 就像整个微服务架构系统的门面一样，所有的外部客户端访问都需要经过它来进行调度和过滤。 它除了要实现请求路由、 负载均衡、 校验过滤等功能之外， 还需要更多能力， 比如与服务 治理框架的结合、 请求转发时的熔断机制、 服务的聚合等一系列高级功能
 - 请求过滤
   - 通过在网关中完成校验和过滤，微服务应用端就可以去除各种复杂 的过滤器和拦截器了
   - 继承 ZuulFilter 抽象类并实现它定义 的4个抽象函数
      - 过滤类型
      - 执行顺序
      - 执行条件
      - 具体操作
   
 - 它作为系统的统一入口， 屏蔽了系统内部各个微服务的细节。
 - 它可以与服务治理框架结合，实现自动化的服务实例维护以及负载均衡的路由转发。
 - 它可以实现接口权限校验与微服务业务逻辑的解耦。
 - 通过服务网关中的过炖器， 在各生命周期中去校验请求的内容， 将原本在对外服务
   层做的校验前移， 保证了微服务的无状态性， 同时降低了微服务的测试难度， 让服 务本身更集中关注业务逻辑的处理。
 - 路径匹配
  - 匹配到第一个后就返回，取决于保存顺序（yaml文件）
 - 本地跳转
 
 - @EnableZuulProxy 注解开启api网关服务
 
 ### Spring cloud config
 用来为分布式系统中的 基础设施和微服务应用提供集中化的外部配置支持
 -  服务端
   - 分布式配置中心，独立，为客户端提供配置信息，加密，解密等接口
 - 客户端
 
 - 基础要素
   - 远程git仓库：存储配置文件
   - 服务端：指定所连接git仓库位置，以及账号密码
   - 本地git仓库：远程->本地->服务端 ，如果远程不可用，直接返回本地
   - 客户端：指定服务端地址
   ![](https://i.loli.net/2019/03/28/5c9c344f4e57a.png)
   
   - 高可用
   - 失败快速相应
   - 动态刷新配置
   
   ### Spring cloud Bus
   
   实现了一些消息总线中的常用功能， 比如， 配合 Spring Cloud Config 实现微服务应用配置信息的动态更新等
   - kafka,RabbitMQ
   ![](https://i.loli.net/2019/03/28/5c9c6f7c3f459.png)
   
   ### Spring cloud Stream
    Spring Cloud Stream 本质上就是整合了 Spring Boot 和 Spring Integration, 实现了一套轻量级的消息驱动的微服务框架
    - 目前只支持RabbitMQ,kafka
    - @EnableBinding, 该注解用来指定一个或多个定义了@input或@Output注解 的接口， 以此实现对消息通道( Channel) 的绑定。
    - @StreamListener
    - 对于每一 个Spring CloudStream的应用程序来说， 它不需要知晓消息中间件 的通信细节， 它只需知道Binder 对应程序提供的抽象概念来使用消息中间件来实现业务 逻辑即可， 
    - 结构图
    ![](https://i.loli.net/2019/03/28/5c9c723b1a4a7.png)
    - 绑定器
     - 实现应用程序与消息中间件的隔离
    - 发布-订阅模式
    - 消费组 通过分组，避免消息重复消费
    - todo，后续功能
  ### Spring cloud Sleuth 
  服务跟踪
  - 基本元素
    - TraceID, 它用来标识一条请求链路。 一条请求链路中包含一个TraceID
    - SpanID, 它表示一个基本的工作单元
    - Sampled: 是否被抽样输出的标志， 1 表示需要被输出 ， 0 表示不需要被输出 。
       - 通过 Sampler 接口
       - 可以配置百分比
  - 配合ELK使用
    
    