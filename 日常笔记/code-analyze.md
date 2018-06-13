### zheng 工程分析记录

#### zheng-admin
- war包工程,纯html工程
- 待确认
    - [ ] maven中surefire插件作用?
        ```
        Maven通过Maven Surefire Plugin插件执行单元测试
        资料:http://www.cnblogs.com/pixy/p/4718176.html
        ```
    - [ ] 项目中后台登录页只有框架,左侧菜单何处加载?
    - [ ] login页  后台登录地址 "/sso/login"

#### zheng-api

- zheng-api-common      #jar `Apiresult相关类`
- zheng-api-rpc-api
    ```
    maven dependency标签>type说明:
    http://rickqin.blog.51cto.com/1096449/1738774
    ```
    ```
    rpc-api中的ApiService 只有hello空实现?
    ```
- zheng-api-rpc-service
    ```Apiservice的实现,靠main函数启动,项目当中使用maven-assembly插件```

- zheng-api-server
    ```
    war包,mq依赖,启动后监听/test
    有用到消息队列,启动后会解压zheng-admin.jar 至resources目录.
    ```
    - [ ] api-server的使用场景?


#### zheng-cms
```
├── zheng-cms-admin   #war包,mq依赖
    - 单点登录配置zheng.oss.aliyun.oss.policy
├── zheng-cms-common  #jar
    - 放result相关类
├── zheng-cms-dao     #dao, 代码生成
├── zheng-cms-job     #war,mq依赖
├── zheng-cms-rpc-api #jar 纯api接口+mock实现
├── zheng-cms-rpc-service  #jar,service实现,main方法运行,通过此项目向dubbo中注册service
├── zheng-cms-search
├── zheng-cms-web

```
- 待确认
    - [ ] 熟悉shiro框架,RequiresPermissions("cms:page:update"),数据库如何储存"cms:page:update"??
    或只能硬编码到代码中?
    - [ ] 每个war包项目(web项目)中都有mq,目的?
