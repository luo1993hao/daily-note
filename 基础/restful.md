
#### 是什么
REST全称是Representational State Transfer，中文意思是表述性状态转移。(表述性：就是指客户端请求一个资源，服务器拿到的这个资源，就是表述)

REST指的是一组架构约束条件和原则。如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构
#### 主要概念
- 资源与URI
  - 任何事物都可以是资源，URI是资源的标识
- 好的URI设计
  - 使用_或者-让URI可读性
  - 使用/来标示资源的层级关系
  - 使用？来过滤资源
  - ,;表示同级资源
 #### 统一资源接口
统一资源接口要求使用标准的HTTP方法对资源进行操作，所以URI只应该来表示资源的名称，而不应该包括资源的操作
- 合适的Http动作
  - GET
    - 安全幂等
    -  获取表示
  - POST
     - 不安且不幂等
     - 创建资源，更新资源
  - PUT
    - 不安全但是幂等
  - DELETE
    - 不安全但幂等
- 具体约束
  - 每个资源都拥有一个资源标识，每个资源的资源标识可以用来唯一标明该资源
  - 消息的自描述性
  - 资源的自描述性。
  - HATEOAS Hypermedia As The Engine Of Application State(超媒体作为应用状态引擎)
- 选择适当的表示结构
  - json ,xml
- 版本控制
