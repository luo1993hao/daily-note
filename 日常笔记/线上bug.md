#### 生产环境=yh
#### 开发环境=123

##### 1. 123上面查询站点列表很快，但到yh上面查询时间达到17S
 1.确认代码相同过后，问题定位到sql上面。开启sql慢查询
 ```
 方法一：全局变量设置
将 slow_query_log 全局变量设置为“ON”状态

mysql> set global slow_query_log='ON'; 
设置慢查询日志存放的位置

mysql> set global slow_query_log_file='/usr/local/mysql/data/slow.log';
查询超过1秒就记录

mysql> set global long_query_time=1;
 ```
 链接
 ```
 https://www.cnblogs.com/luyucheng/p/6265594.html
 ```
 2. tail -f 慢查询日志，发现查询时间慢的sql，然后查看其执行计划
 3. explain 对于没走索引的left join 加索引
 4. 对于许多表链接的查询，多用小型查询，少用大型查询。
 5. 
 
```
lng
├── lng-common -- 框架公共模块
├── lng-message -- 消息模块
├── lng-pay -- 支付模块
├── document -- 开发文档及过程记录
├── script -- 开发所需脚本
├── lng-api -- lng业务代码
|    ├── lng-api-model -- 实体层，包括生成部分与业务对象
|    ├── lng-api-interface -- rpc接口包
|    ├── lng-api-service -- 后台管理[端口:8989]
├── lng-upms -- 用户权限管理系统
|    ├── lng-upms-interface -- rpc接口包
|    ├── lng-upms-service -- 用户权限系统及SSO服务端[端口:1111]

```
```
lng
├── lng-common -- 框架公共模块
├── lng-message -- 消息模块
├── lng-pay -- 支付模块
├── document -- 开发文档及过程记录
├── script -- 开发所需脚本
├── lng-delivery -- lng配送模块
|    ├── lng-delivery-common -- 配送模块公共代码
|    ├── lng-delivery-model -- 配送模块实体层，包括生成部分与业务对象
|    ├── lng-delivery-interface -- rpc接口包
|    ├── lng-delivery-service -- 接口实现层
|    ├── lng-delivery-web -- 后台管理[端口:8995]
├── lng-monitor -- lng监控预警模块
|    ├── lng-monitor-common -- **监控模块公共代码**
|    ├── lng-monitor-model --监控模块实体层，包括生成部分与业务对象
|    ├── lng-monitor-interface -- rpc接口包
|    ├── lng-monitor-service -- 接口实现层
|    ├── lng-monitor-web -- 后台管理[端口:8990]
├── lng-upms -- 用户权限管理系统
|    ├── lng-upms-interface -- rpc接口包
|    ├── lng-upms-service -- 用户权限系统及SSO服务端[端口:1111]

```
```
2018-05-29 14:10:47 ERROR SpringApplication:842 - Application run failed
java.lang.RuntimeException: [source error] getPropertyValue (Ljava/lang/Object;Ljava/lang/String;)Ljava/lang/Object; in com.alibaba.dubbo.common.bytecode.Wrapper11: inconsistent stack height -1
	at com.alibaba.dubbo.common.bytecode.ClassGenerator.toClass(ClassGenerator.java:301)
	at com.alibaba.dubbo.common.bytecode.ClassGenerator.toClass(ClassGenerator.java:255)
	at com.alibaba.dubbo.common.bytecode.Wrapper.makeWrapper(Wrapper.java:237)
	at com.alibaba.dubbo.common.bytecode.Wrapper.getWrapper(Wrapper.java:99)
	at com.alibaba.dubbo.config.ServiceConfig.doExportUrlsFor1Protocol(ServiceConfig.java:443)
	at com.alibaba.dubbo.config.ServiceConfig.doExportUrls(ServiceConfig.java:356)
	at com.alibaba.dubbo.config.ServiceConfig.doExport(ServiceConfig.java:315)
	at com.alibaba.dubbo.config.ServiceConfig.export(ServiceConfig.java:214)
	at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:120)
	at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:49)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:172)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:165)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:400)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:354)
	at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:888)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.finishRefresh(ServletWebServerApplicationContext.java:161)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:553)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:759)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:395)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:327)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1255)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1243)
	at com.c503.lng.api.Application.main(Application.java:22)
Caused by: javassist.CannotCompileException: [source error] getPropertyValue (Ljava/lang/Object;Ljava/lang/String;)Ljava/lang/Object; in com.alibaba.dubbo.common.bytecode.Wrapper11: inconsistent stack height -1
	at javassist.CtNewMethod.make(CtNewMethod.java:79)
	at javassist.CtNewMethod.make(CtNewMethod.java:45)
	at com.alibaba.dubbo.common.bytecode.ClassGenerator.toClass(ClassGenerator.java:280)
	... 24 more
Caused by: compile error: getPropertyValue (Ljava/lang/Object;Ljava/lang/String;)Ljava/lang/Object; in com.alibaba.dubbo.common.bytecode.Wrapper11: inconsistent stack height -1
	at javassist.compiler.Javac.compile(Javac.java:104)
	at javassist.CtNewMethod.make(CtNewMethod.java:74)
	... 26 more