### 相关概念
- JoinPoint
- Advice  需要增强的逻辑
  - before
  - after
  - afterReturning
  - throws
  - around
- PointCut 定义哪些方法哪些类需要advice
- Advisor 切面。 组合pointCut+advice
  - 需要增强的目标方法列表，这个通过切入点(Pointcut)来指定
  - 需要在目标方法中增强的逻辑，这个通过(Advice)通知来指定
### 使用方式
- Spring 1.2 基于接口的配置：最早的 Spring AOP 是完全基于几个接口的，想看源码的同学可以从这里起步。
- Spring 2.0 schema-based 配置：Spring 2.0 以后使用 XML 的方式来配置，使用 命名空间 <aop />
- Spring 2.0 @AspectJ 配置：使用注解的方式来配置，这种方式感觉是最方便的，还有，这里虽然叫做 @AspectJ，但是这个和 AspectJ 其实没啥关系。
### 底层原理
原理很简单，通过动态代理来创建代理对象，通过代理对象来访问目标对象，而代理对象中融入了增强的代码，最终起到对目标对象增强的效果。
### 思考
如果让我们自己设计完成aop的流程，需要解决三个问题。1.目标对象的提取 2.找到目标对象的pointCut，也就是在哪儿进行增强 3.如何增强，也就是将advice的逻辑如何织入
所以我们的大概逻辑应该是
1. 创建实例对象
2. 提取切面信息，(advice+pointCut)
3. 在pointCut上织入advice，创建代理对象
### 整体流程
1. 解析标签，注册代理者
2. 创建代理对象
3. 代理对象方法调用时候，通过JDK或者CGLIB进行增强调用
### 详细流程
#### 解析标签，注册代理者
#### 创建代理对象
- 筛选出所有适合当前Bean的通知器，也就是所有的Advisor、Advise、Interceptor。
- 选择使用JDK还是CGLIB来进行创建代理。
- 使用具体的代理实现来创建代理。
#### 代理的使用
- 获取当前调用方法的拦截器链，包含了所有将要执行的advice。
- 如果没有任何拦截器，直接执行目标方法。
- 如果有拦截器存在，则将拦截器和目标方法封装成一个MethodInvocation，递归调用proceed方法进行调用。
