# zheng框架代码阅读笔记（11.1）

## 项目结构
常用的业务系统模块应该项目结构如下
```

├── zheng-xxx-common  
    - 放result相关类
├── zheng-xxx-dao     #dao, 代码生成
├── zheng-xxx-rpc-api #jar 纯api接口+mock实现
├── zheng-xxx-rpc-service  service实现,main方法运行,通过此项目向dubbo中注册service
├── zheng-xxx-web controller实现，通过此项目向dubbo消费服务
```
## 常见配置文件说明
- 在zheng框架中，配置文件一般存在于service模块与web（server模块），大于大多数业务系统来说，不同只是dubbo的配置文件
- 配置文件使用${}，总的修改在profiles的具体选择开发模式的配置文件修改，而选择的模式通过mvn命令，缺省的为dev。
- 


```
service中

│  config.properties 
│  ehcache.xml
│  jdbc.properties
│  log4j.properties
│  redis.properties
│
├─META-INF
│  └─spring
│          applicationContext-dubbo-provider.xml

  服务提供者，zookeeper，以接口形式发布
│          applicationContext-ehcache.xml
│          applicationContext-jdbc.xml
#不仅jdbc配置，整合了spring与mybatis
│          applicationContext-listener.xml
     初始化mapper
│          applicationContext.xml 
初始化一个可以读取任何bean的springUtil,
spring类中使用到的ApplicationContextAware说明
http://blog.csdn.net/majian_1987/article/details/11170841

└─profiles
        dev.properties
        prod.properties
        test.properties
```




```
web模块中
│  applicationContext-activemq.xml
│  applicationContext-dubbo-consumer.xml
服务消费者
│  applicationContext-ehcache.xml
│  applicationContext-threadpool.xml
│  applicationContext-zhengAdmin.xml
解压前台模板到resource目录下
servletcontext相关知识：
http://blog.csdn.net/lvzhiyuan/article/details/4664624
│  config.properties
│  ehcache.xml
│  log4j.properties
│  redis.properties
│  springMVC-servlet.xml
│  zheng-admin-client.properties
│  zheng-oss-client.properties
│  zheng-upms-client.properties
│
├─i18n
│      messages_en_US.properties
│      messages_zh_CN.properties
│
└─profiles
        dev.properties
        prod.properties
        test.properties
```
## zheng-common



## zheng-api

**zheng-api-common**
系统返回值

**zheng-api-rpc-api** 只有一个apiService类，这里应全是接口
**zheng-api-rpc-service**
只有一个apiserviceimpl

**zheng-api-server**

**疑惑**
- 整个api模块的作用
- activemq的作用
- shrioFliter执行次序。
- 为什么界面上新增用户界面正常。使用Url访问。只有框。没有界面。
- 
- 

## zheng-upms
#### zheng-upms-client
权限验证客户端，使用shrio框架，

[shrio运行流程介绍](http://blog.csdn.net/mine_song/article/details/61616259)




以jar包形式被zheng-upms-server引用，提供
```
├─interceptor
│      LogAspect.java
│
├─shiro
│  ├─filter
│  │      UpmsAuthenticationFilter.java
#重写shrio认证过滤器,也就是自定义用户通过的情景
│  │      UpmsSessionForceLogoutFilter.java
│  │
│  ├─listener
│  │      UpmsSessionListener.java
│  │
│  ├─realm
│  │      UpmsRealm.java
#Shiro从从Realm获取安全数据（如用户、角色、权限）
#这就是一个安全数据源
#通常我们只需要继承AuthorizingRealm(授权)
#，因为AuthorizingRealm里面继承了AuthenticatingRealm（认证），所以我们
#只需要继承AuthorizingRealm（授权），
#我们就可以重写授权和认证两个方法了，这两个方法里面就实现权限管理操作
#
│  │
│  └─session
│          UpmsSession.java
│          UpmsSessionDao.java
#将共享session缓存到redis
│          UpmsSessionFactory.java
│
└─util
        RequestParameterUtil.java
        SerializableUtil.java
```

在zheng-upms-server的web.xml中。实现了shrio与spring的整合
```
<!-- shiroFilter : DelegatingFilterProxy作用是自动到spring容器查找名字为shiroFilter（filter-name）的bean并把所有Filter的操作委托给它。然后将shiroFilter配置到spring容器即可 -->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

shiro组件介绍

http://jinnianshilongnian.iteye.com/blog/2018936/
- 疑问
> 为什么cms-admin这个模块也依赖zheng-upms-client

- Q1：**该模块如何被加载的？**

A1：经过验证了以下几点结论：

1.web.xml中以下面这种形式是可以扫描到jar包中的xml文件，并且是按照**名称匹配扫描。**
```
 <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:applicationContext*.xml
        </param-value>
    </context-param>
```
2.继承spring这2个接口的。是在spring初始化的时候起作用，在bean调用的时候并没有影响
```
public class XXX implements InitializingBean, ServletContextAware 
```

- Q2：该模块的流程以及怎样被使用的？ 

见git上

**补充说明**

1.只有upms-type为server。而为server的跳转到登录界面。其他的都为client。client端登录的话。会先去注册中心取code（也就是判断你是否已经登录过，如果没有登录，会跳转到单点登录界面进行登录）所以在系统中切换系统不会重新登录。 

- shiro整个流程包含用户，角色，权限，并且都是多对多的关系，所以在本模块在数据库中有五张基础表（2张关联表）。加上业务扩充表
upms_user，upms_role，upms_permission，upms_user_role，upms_role_permission

#### zheng-upms-rpc-service
- 相较于常规的service（代码生成的），多了一个upmsApiMapper接口与xml。在rpc-api接口模块也多了一个对应的接口。这个接口并没有继承baseService。按照业务需求自己重写的接口
(基本都是基于用户的查询)
**这个模块中，很多业务场景，需要站在用户的维度去获取关于用户资源**

 这是spring与mybtis整合的配置文件，spring都会扫描路径下的文件并装配为Bean。
 
  MapperScannerConfigurer , 它 将 会 查 找 类 路 径 下 的 映 射 器 并 自 动 将 它 们 创 建 成 MapperFactoryBean。把Mapper接口转换成MapperFactoryBean
  
  mapperScannerConfigurer说明
  http://www.cnblogs.com/daxin/p/3545040.html
```
  <!-- 为Mybatis创建SqlSessionFactory，同时指定数据源 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath*:com/zheng/cms/dao/mapper/*Mapper.xml"/>
    </bean>
    <!-- Mapper接口所在包名，Spring会自动查找其下的Mapper -->
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="**.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
```
#### zheng-upms-server
代码结构
```
─java
│  └─com
│      └─zheng
│          └─upms
│              └─server
│                  │  Initialize.java
│                  │
│                  ├─controller
│                  │  │  ManageController.java
                      #shrio验证过后的首页
│                  │  │  SSOController.java
                       #登录验证
│                  │  │
│                  │  └─manage #业务模块，ciud
│                  │          UpmsLogController.java
│                  │          UpmsOrganizationController.java
│                  │          UpmsPermissionController.java
│                  │          UpmsRoleController.java
│                  │          UpmsSessionController.java
│                  │          UpmsSystemController.java
│                  │          UpmsUserController.java
│                  │
│                  └─interceptor
│                          UpmsInterceptor.java
                            ##登录信息拦截器，只实现了预处理，
                            目的就是根据用户名查询用户属性，
                            为后面shiro登录保存用户信息。

```
---

疑问：
每个dev.properties里面都有一个zheng.config.path.。这个的作用？。如果现在zheng还没有实现。那预留这个想做成什么样的效果。？

解答：预留位，并没有实现。目的是做成分布式的配置。在zheng-config这个模块进行所有配置的统一配置。进行配置管理

---
## zheng-cms
### zheng-cms-rpc-service
如果有业务拓展（代码生成的dao不能满足业务需求），就在com.zheng.XXX(模块名).rpc.mapper下面新建拓展结构与对应的Mapper。在impl中写实现
### zheng-cms-admin



##### zheng-> lng 替换步骤
总的思想就是通过shell命令与idea，利用正则表达式批量修改
1. 修改文件名
2. 修改pom文件中的（idea中ctrl+shift+R,输入正则表达式进行替换）
3. 

## todolist
1.为什么cms一定要activemq