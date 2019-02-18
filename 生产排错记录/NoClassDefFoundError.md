## 1 问题描述

### 1.1 接口地址
~~~
http://www.yhlng.com/manage/widget/getBoxUserByType?t=1550023385337&pageId=1&pageSize=30&boxType=2&boundBox=1
~~~
- [#线上环境报错](http://192.168.1.116/lng/lng-microservice/issues/801)

### 1.2 问题截图
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190214101538.png)

![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190214101710.png)

## 2 :排查流程

### 2.1 参考资料
- [关于怎么解决java.lang.NoClassDefFoundError错误](https://www.cnblogs.com/xyhz0310/p/6803950.html)

![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218091957.png)

### 2.2 引入的jar版本
是正确的
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218091311.png)

### 2.3 反编译class文件
jar包中的class文件反编译后代码正常
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218091541.png)
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218091705.png)

### 2.3 使用pm2 启动
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218095932.png)
只有日志 不会覆盖了原来的classpath环境变量

### 2.4 排查ExceptionInInitializerError
这是在启动静态初始化模块错误 在1月28号更新异常 没有错误信息
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218092354.png)

### 2.4 排查NoClassDefFoundError
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218093149.png)

2月13号 有业务系统的异常
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218093430.png)

![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218094317.png)

![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218095039.png)

![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218095159.png)

### 参考资料
- [解决spring boot启动报错java.lang.NoClassDefFoundError](https://webcache.googleusercontent.com/search?q=cache:tC0vZfFD3lQJ:https://www.aliyun.com/jiaocheng/774230.html+&cd=7&hl=en&ct=clnk&gl=tw)
![](https://riverluooo.oss-cn-beijing.aliyuncs.com/img/20190218095403.png)