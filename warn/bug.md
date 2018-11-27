### zookeeper
问题：无法启动

日志描述：
```
2017-10-25 14:24:55,473 [myid:] - INFO  [main:QuorumPeerConfig@318] - clientPortAddress is 0.0.0.0/0.0.0.0:2181
2017-10-25 14:24:55,474 [myid:] - INFO  [main:QuorumPeerConfig@322] - secureClientPort is not set
2017-10-25 14:24:55,498 [myid:] - ERROR [main:QuorumPeerMain@86] - Invalid config, exiting abnormally
org.apache.zookeeper.server.quorum.QuorumPeerConfig$ConfigException: Address unresolved: zkslave01.com:3888  
        at org.apache.zookeeper.server.quorum.QuorumPeer$QuorumServer.<init>(QuorumPeer.java:242)
        at org.apache.zookeeper.server.quorum.flexible.QuorumMaj.<init>(QuorumMaj.java:89)
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.createQuorumVerifier(QuorumPeerConfig.java:524)
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parseDynamicConfig(QuorumPeerConfig.java:557)
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.setupQuorumPeerConfig(QuorumPeerConfig.java:530)
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parseProperties(QuorumPeerConfig.java:353)
        at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parse(QuorumPeerConfig.java:133)
        at org.apache.zookeeper.server.quorum.QuorumPeerMain.initializeAndRun(QuorumPeerMain.java:110)
        at org.apache.zookeeper.server.quorum.QuorumPeerMain.main(QuorumPeerMain.java:79)
Invalid config, exiting abnormally

```
问题原因

配置文件中存在空格，无法识别配置文件中的地址

解决办法

去掉空格 但是启动后仍然无法启动

问题原因

当首次启动zookeeper成功过后，zoo.cfg中的地址配置会转移到一个动态文件，里面内容为地址配置。而zoo.cfg会自动添加一行配置。地址指向该动态文件 
```
  dynamicConfigFile=/usr/local/lng/zookeeper/conf/zoo.cfg.dynamic.300000000
```
zoo.cfg.dynamic.3000000里面内容为地址
```
server.1=zkmaster.com:2888:3888:participant
  2 server.2=zkslave01.com:2888:3888:participant
  3 server.3=zkslave02.com:2888:3888:participant

```
### controller
#### 问题1：

##### 描述
在使用swagger测试更新接口时（传参为实体类）
```
ERROR [com.c503.hthj.lng.common.base.BaseController] - 统一异常处理：
org.springframework.http.converter.HttpMessageNotReadableException: Could not read document: Unexpected character ('"' (code 34)): was expecting comma to separate OBJECT entries
 at [Source: java.io.PushbackInputStream@188a9c31; line: 3, column: 4]; nested exception is com.fasterxml.jackson.core.JsonParseException: Unexpected character ('"' (code 34)): was expecting comma to separate OBJECT entries
 at [Source: java.io.PushbackInputStream@188a9c31; line: 3, column: 4]
```
##### 原因：

字符串格式错误

##### 解决办法：

检查json串中的“与，是否为英文字符

##### 参考链接

http://blog.csdn.net/zhouyingge1104/article/details/50232661

### dubbo
dubbo.properties里面配置了地址！！
### redis
```
[WARNING] 
org.apache.jasper.JasperException: javax.servlet.ServletException: redis.clients.jedis.exceptions.JedisMovedDataException: MOVED 1108 192.168.1.124:7003
	at org.apache.jasper.servlet.JspServletWrapper.handleJspException(JspServletWrapper.java:585)
	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:455)
	at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:405)
	at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:349)
	at org.eclipse.jetty.jsp.JettyJspServlet.service(JettyJspServlet.java:107)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:800)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1669)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:101)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1652)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:585)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:595)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1061)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
	at org.eclipse.jetty.server.Dispatcher.forward(Dispatcher.java:191)
	at org.eclipse.jetty.server.Dispatcher.forward(Dispatcher.java:72)
	at org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:168)
	at org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:303)
	at org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1282)
	at org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1037)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:980)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:897)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970)
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:861)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:687)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:800)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1669)
	at org.apache.shiro.web.servlet.ProxiedFilterChain.doFilter(ProxiedFilterChain.java:61)
	at org.apache.shiro.web.servlet.AdviceFilter.executeChain(AdviceFilter.java:108)
	at org.apache.shiro.web.servlet.AdviceFilter.doFilterInternal(AdviceFilter.java:137)
	at org.apache.shiro.web.servlet.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:125)
	at org.apache.shiro.web.servlet.ProxiedFilterChain.doFilter(ProxiedFilterChain.java:66)
	at org.apache.shiro.web.servlet.AbstractShiroFilter.executeChain(AbstractShiroFilter.java:449)
	at org.apache.shiro.web.servlet.AbstractShiroFilter$1.call(AbstractShiroFilter.java:365)
	at org.apache.shiro.subject.support.SubjectCallable.doCall(SubjectCallable.java:90)
	at org.apache.shiro.subject.support.SubjectCallable.call(SubjectCallable.java:83)
	at org.apache.shiro.subject.support.DelegatingSubject.execute(DelegatingSubject.java:383)
	at org.apache.shiro.web.servlet.AbstractShiroFilter.doFilterInternal(AbstractShiroFilter.java:362)
	at org.apache.shiro.web.servlet.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:125)
	at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:346)
	at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:262)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1652)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:197)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1652)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:585)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:577)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1061)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:215)
	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:110)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:97)
	at org.eclipse.jetty.server.Server.handle(Server.java:497)
	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:310)
	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:245)
	at org.eclipse.jetty.io.AbstractConnection$2.run(AbstractConnection.java:540)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:635)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:555)
	at java.lang.Thread.run(Thread.java:722)
Caused by: javax.servlet.ServletException: redis.clients.jedis.exceptions.JedisMovedDataException: MOVED 1108 192.168.1.124:7003
	at org.apache.jsp.WEB_002dINF.jsp.index_jsp._jspService(index_jsp.java:104)
	at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:432)
	... 69 more
Caused by: redis.clients.jedis.exceptions.JedisMovedDataException: MOVED 1108 192.168.1.124:7003
	at redis.clients.jedis.Protocol.processError(Protocol.java:115)
```

问





```
org.apache.jasper.JasperException: javax.servlet.ServletException: redis.clients.jedis.exceptions.JedisClusterException: CLUSTERDOWN The cluster is down
```  
### nginx
问题：
加载静态资源成功但是不显示样式，本应该是html，但是却显示文本格式。

解决办法及原因：

http://blog.csdn.net/justnow_/article/details/52628022





# 代办Bug
```
WARN  [com.alibaba.dubbo.remoting.transport.AbstractClient] -  [DUBBO] client reconnect to 192.168.120.1:20881 find error . url: dubbo://192.168.120.1:20881/com.zheng.upms.rpc.api.UpmsSystemService?anyhost=true&application=zheng-upms-server&check=false&codec=dubbo&default.check=false&dubbo=2.5.6&generic=false&heartbeat=60000&interface=com.zheng.upms.rpc.api.UpmsSystemService&methods=selectByExampleWithBLOBsForStartPage,selectByExampleWithBLOBs,updateByExampleSelective,updateByPrimaryKeyWithBLOBs,initMapper,selectByExampleWithBLOBsForOffsetPage,deleteByExample,selectFirstByExample,selectByExampleForOffsetPage,selectByExampleForStartPage,countByExample,selectByPrimaryKey,insertSelective,deleteByPrimaryKeys,updateByExampleWithBLOBs,updateByPrimaryKey,deleteByPrimaryKey,updateByPrimaryKeySelective,insert,updateByExample,selectFirstByExampleWithBLOBs,selectByExample&mock=true&pid=11196&remote.timestamp=1509083739608&revision=1.0.0&side=consumer&timeout=10000&timestamp=1509074321100, dubbo version: 2.5.6, current host: 192.168.109.201
com.alibaba.dubbo.remoting.RemotingException: client(url: dubbo://192.168.120.1:20881/com.zheng.upms.rpc.api.UpmsSystemService?anyhost=true&application=zheng-upms-server&check=false&codec=dubbo&default.check=false&dubbo=2.5.6&generic=false&heartbeat=60000&interface=com.zheng.upms.rpc.api.UpmsSystemService&methods=selectByExampleWithBLOBsForStartPage,selectByExampleWithBLOBs,updateByExampleSelective,updateByPrimaryKeyWithBLOBs,initMapper,selectByExampleWithBLOBsForOffsetPage,deleteByExample,selectFirstByExample,selectByExampleForOffsetPage,selectByExamp
```
```
 ERROR [com.alibaba.dubbo.monitor.dubbo.DubboMonitor] -  [DUBBO] Unexpected error occur at send statistic, cause: Failed to invoke the method collect in the service com.alibaba.dubbo.monitor.MonitorService. No provider available for the service com.alibaba.dubbo.monitor.MonitorService from registry zkserver:2181 on the consumer 192.168.109.201 using the dubbo version 2.5.6. Please check if the providers have been started and registered., dubbo version: 2.5.6, current host: 192.168.109.201
com.alibaba.dubbo.rpc.RpcException: Failed to invoke the method collect in the service com.alibaba.dubbo.monitor.MonitorService. No provider available for the service com.alibaba.dubbo.monitor.MonitorService 
```
