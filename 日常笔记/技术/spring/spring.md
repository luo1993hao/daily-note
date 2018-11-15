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

