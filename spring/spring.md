### 注解标签说明
 1. @ResponseBody
 
把方法返回的结果给放在responseBody里面。直接返回给前台。而不是跳转页面。

### mybatis细节知识



1. 传递参数：参数替换分两种：#{id} => 预处理语句的？ || '${id}' => 直接替换为变量
2. 
Criteria

包含一个Cretiron的集合,每一个Criteria对象内包含的Cretiron之间是由AND连接的,是逻辑与的关系。

oredCriteria

Example内有一个成员叫oredCriteria,是Criteria的集合,就想其名字所预示的一样，这个集合中的Criteria是由OR连接的，是逻辑或关系。oredCriteria就是ORed Criteria。

### springmvc细节知识
1.  整个DispatcherServlet初始化的过程和做了些什么事情，具体主要做了如下两件事情：
1、初始化Spring Web MVC使用的Web上下文，并且可能指定父容器为（ContextLoaderListener加载了根上下文）；
2、初始化DispatcherServlet使用的策略，如HandlerMapping、HandlerAdapter等。
### spring aop理解
- 用于解剖封装好的对象内部，找出其中对多个对象产生影响的公共行为，并将其封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），切面将那些与业务无关，却被业务模块共同调用的逻辑提取并封装起来，减少了系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性
- 静态代理
  - 编译时增强
- 动态代理 动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法
  - jdk动态代理 通过反射。必须为接口
  - CGLIB动态代理。代码生成的类目。动态生成制定累的一个子类对象没并且覆盖特定方法。其实就是通过继承的方式做的动态代理

静态与动态的区别在于aop生成的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。
### spring 中的线程安全

