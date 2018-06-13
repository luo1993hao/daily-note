# zheng部署手册
## 开发工具
1. mysql：数据库
2. tomcat
3. Nginx
4. IntelliJ IDEA
5. Jetty

## 开发环境
- jdk7+(1.7.0_79)
- mysql5.5+(5.6.10)（三台，主从模式）
- redis(4.0.2)（在zheng框架中目前只支持单机模式）
- zookeeper(3.5.2)(三台集群模式)
- activeMQ(5.9.0)
- nginx(1.5.0)
- dubbo-admin(2.8.4)
- 括号内为服务器部署使用版本与模式。所有部署成单机模式程序也可以正常运行
## 相关文档
nginx安装下载

http://www.cnblogs.com/qianzf/p/6804395.html
```
nginx依赖以下模块：

gzip模块需要 zlib 库

rewrite模块需要 pcre 库

ssl 功能需要openssl库

安装pcre
获取pcre编译安装包，在http://www.pcre.org/上可以获取当前最新的版本
解压缩pcre-xx.tar.gz包。
进入解压缩目录，执行./configure –prefix==/usr/local/pcre。
make & make install
1.2. 安装openssl
获取openssl编译安装包，在http://www.openssl.org/source/上可以获取当前最新的版本。
解压缩openssl-xx.tar.gz包。
进入解压缩目录，执行./config。
make & make install
1.3.安装zlib
获取zlib编译安装包，在http://www.zlib.net/上可以获取当前最新的版本。
解压缩openssl-xx.tar.gz包。
进入解压缩目录，执行./configure。
make & make install
1.4.安装nginx
获取nginx，在http://nginx.org/en/download.html上可以获取当前最新的版本。
解压缩nginx-xx.tar.gz包。
进入解压缩目录，执行./configure
make & make install

启动nginx之后，浏览器中输入http://localhost可以验证是否安装启动成功
```


redis部署

集群：

http://www.cnblogs.com/wuxl360/p/5920330.html

单机：

http://blog.csdn.net/thinkercode/article/details/46577197

mysql主从配置

[mysql主从配置](http://note.youdao.com/noteshare?id=6ad42ae65aabf450ba999bd5a5acdc74&sub=7391C2E6FF2848E2A28E87BD50F7C535)

zookeeper集群配置

http://blog.csdn.net/mark_lq/article/details/52447181
## 软件下载链接
- JDK7
```
http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
```
- Maven
```
http://maven.apache.org/download.cgi
```
- Redis 
```
https://redis.io/download
```
- ActiveMQ
```
http://activemq.apache.org/download-archives.html
```
- ZooKeeper 
```
http://www.apache.org/dyn/closer.cgi/zookeeper/
```
Dubbo管控台
```
http://download.csdn.net/download/shuzheng5201314/9733652
```
Nginx
```
http://nginx.org/en/download.html
```
公司内网资源（jdk,maven）
```
http://192.168.1.38/ambari/dependences/java/
```
## 软件路径（linux下）
192.168.1.123,192.168.1.125
- jdk 
 ```
 /usr/local/lng/jdk
 ```
 - redis
 ```
 /usr/local/lng/redis-4.0.2
 ```
 - redis 配置文件
 ```
 /usr/local/lng/cluster
 ```
 - activeMQ
 ```
 /usr/local/lng/apache-activemq-5.9.0
 ```
 - zookeeper
 ```
 /usr/local/lng/zookeeper
 ```
 - uac
 ```
 /usr/local/lng/uac
 ```
 - nginx
 ```
 /usr/local/lng/nginx
 ```
 - zheng源码
 ```
 /usr/local/zheng
 ```
 - dubbo.war(只在123上面)
 ```
 /usr/local/lng/apache-tomcat-lng
 ```
 - maven
 ```
 /usr/local/lng/apache-maven-3.3.9-aliyun
 ```
 - tomcat解压包
 ```
 /usr/local/lng/apache-tomcat-lng/tomcat.tar.gz/
 ```
 - zheng-service(cms,upms)
 ```
 /usr/local/lng/dubbo-service
 ```
 - zheng-tomcat(cms-admin,upms,cms-web)
 ```
 /usr/local/lng/tomcat-zheng
 ```
 - mysql
 ```
 /var/lib
 ```
 192.168.1.124
 - jdk
 ```
 /usr/local/lng
 ```
 - activeMQ,redis
 ```
 /usr/local/lng/lng
 ```
 - zookeeper
 ```
 /usr/local/lng
 ```
 - zheng
 ```
 /usr/local/zheng
 ```
 - zheng-upms(cms)-service
 ```
 /usr/local/lng/dubbo-service
 ```
 - zheng-tomcat
 ```
 /usr/local/lng/tomcat-zheng
 ```
 
## 部署步骤
#### 安装dubbo管控台

将下载的dubbo-admin.war放在tomcat/webapps下运行即可。运行成功后进入localhost:8080/dubbo-admin，账号密码为root,root

1. 下载安装开发环境里的所有软件并启动。其中Jdk,mysql,zookeeper,activeMQ使用默认端口即可。
#### 软件启动方式（linux）
 nginx
 ```
 启动
[root@localhost ~]# /usr/local/lng/nginx/sbin/nginx 
检查是否启动成功： 
netstat -ano|grep 80 #有结果输入说明启动成功
停止/重启
[root@localhost ~]# /usr/local/lng/nginx/sbin/nginx -s stop(quit、reload)
 ```
 zookeeper
 ```
 /usr/local/lng/zookeeper/bin/zkServer.start
 #三台都启动
 /usr/local/lng/zookeeper/bin/zkServer.sh status
 #查看集群状态，是否成功
 ```
 redis
 
 ```
 /usr/local/lng/redis-4.0.2/src/redis-server redis.conf
 #zheng的redis默认端口为7000，修改redis.conf中的port为7000。
 
 
 
 /usr/local/lng/redis-4.0.2/src/redis-cli 
 #以上命令将打开以下终端：
redis 127.0.0.1:7000>
127.0.0.1 是本机 IP ，7000 是 redis 我们修改过后的服务端口。输入 PING 命令。
redis 127.0.0.1:7000> ping
PONG
 ```
 activeMq
 ```
 cd /usr/local/lng/apache-activemq-5.9.0/bin/
 
 ./activemq start  #启动
 
 ./activemq stop #关闭
  netstat -anp|grep 61616
  #验证是否安装成功，打开Http://IP:8161/admin 账号密码都为admin.出现欢迎界面表示安装成功
 ```
 #### windows启动方式
 - nginx 
 进入到nginx根目录，运行nginx.exe文件（一闪而过）,页面输入localhost 出现界面启动成功

关闭
命令行进入nginx根目录 　nginx -s stop
 - activeMq
 
命令行进入activeMq安装目录下的\bin\win64文件夹，运行activemq.bat，关闭窗口即可关闭。
- zookeeper
命令行进入到zookeeper安装目录的bin目录，
```
zkServer.cmd
```
启动后jps看到QuorumPeerMain的进程代表启动成功
- redis

命令行进入redis根目录，
```
redis-server redis.windows.conf
```
命令行出现redis图像代表启动成功

2. 克隆源代码到本地并使用idea打开，本地编译并安装到本地maven库
```
进入到zheng项目根目录下

mvn install
```

项目地址：
https://gitee.com/shuzheng/zheng.git
3. 修改本地hosts文件

**linux下为/etc/hosts**

**windows下为C:\Windows\System32\drivers\etc\hosts**

 127.0.0.1	ui.zhangshuzheng.cn

 127.0.0.1	upms.zhangshuzheng.cn
 
127.0.0.1	cms.zhangshuzheng.cn

127.0.0.1	pay.zhangshuzheng.cn

127.0.0.1	ucenter.zhangshuzheng.cn

127.0.0.1	wechat.zhangshuzheng.cn

127.0.0.1	api.zhangshuzheng.cn

127.0.0.1	oss.zhangshuzheng.cn

127.0.0.1 config.zhangshuzheng.cn

127.0.0.1	zkserver

127.0.0.1	rdserver

127.0.0.1	dbserver

127.0.0.1	mqserver
5. 导入数据库
 新增zheng数据库（如果使用主从模式，参考上面主从mysql配置链接），导入zheng/project-datamodel文件夹下的zheng.sql
6. zheng框架使用的mysql账号为root，密码为123456，redis使用密码为空（redis默认也为空）。要么修改每个模块下的dev.properties下的mysql,redis为你使用的账号密码，并运行zheng/zheng-common/AESUtil类进行加密后，将控制台输出填写到配置文件中。要么新增一个mysql账号为root,密码为123456

使用root用户进入该数据库的mysql命令行，授权
```
 GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123456' WITH GRANT OPTION; 
```

7. 修改Nginx配置文件。参考zheng/project-tools/severs下的nginx配置。修改zheng-ui下的root为你本地zheng/zheng-ui的路径
```
	location / {
		#root   F:/GitHub/zheng/zheng-ui/;
		root  （这里修改为你本地zheng/zheng-ui的路径）
		index  index.html index.htm;
		add_header Access-Control-Allow-Origin *;
	}
```
## 启动步骤
### idea启动
#### zheng-upms
1. 首先启动 zheng-upms-rpc-service(直接运行src目录下的ZhengUpmsRpcServiceApplication#main方法启动),启动成功后，可以在dubbo管控台看到发布的服务
2. 然后使用jetty运行zheng-upms-server（右侧maven project->zheng-upms-server Maven WebApp->Plugins->jetty->run）
3. 访问 http://upms.zhangshuzheng.cn:1111/
4. 子系统菜单已经配置到zheng-upms权限中，不用直接访问子系统，默认帐号密码：admin/123456
#### zheng-cms
1. 运行zheng-cms-rpc-service下的ZhengCmsRpcServiceApplication类
2. 使用jetty启动zheng-cms-admin，启动成功后可以通过已经启动的系统切换到内容管理模块
3. 使用jetty启动zheng-cms-web（前提是启动nginx并配置好ui路径），启动成功后http:localhost:2224 进入内容管理系统后台
#### 其他业务模块
与上面类似，都是先运行service下的类，目的是通过dubbo在zookeeper上发布服务，然后用Jetty运行对面的web。
### linux服务器启动

总的思想跟idea启动一致，先启动servce（发布服务，提供者），后启动web（消费者）

1. 将整个项目放在linux下。
2. maven install zheng的pom.xml文件
2. 将zheng-upms-rpc-service/target/zheng-upms-assembly.tar.gz  解压
3. 运行解压后的bin/start.sh 发布服务
4. 将zheng-upms-server.war放在tomcat下（必须放在webapp/ROOT目录下）
5. 修改tomcat端口号为1111并启动，成功后访问
 http://upms.zhangshuzheng.cn:1111/
7. cms-admin与cms-web启动方式一致，放在不同的tomcat下，修改cms-admin端口号为2222,cms-web端口号为2224即可。
**cms-web启动失败查看activemq是否启动，cms-web页面不正常查看nignx配置**

**zheng-XXX-service（dubbo服务）发布失败，查看zookeeper集群是否正常**
