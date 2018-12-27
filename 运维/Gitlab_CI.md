## Gitlab runner
因构建过程消耗系统资源，是由gitlab runner来执行构建任务。

Runner类型：\
GitLab-Runner可以分类两种类型：
**Shared Runner**（共享型）和**Specific Runner**（指定型）。

Shared Runner：这种Runner是所有工程都能够用的。只有系统管理员能够创建Shared Runner。

Specific Runner：这种Runner只能为指定的工程服务。拥有该工程访问权限的人都能够为该工程创建Shared Runner。

#### 安装gitlab-runner
[国内镜像](https://mirrors.tuna.tsinghua.edu.cn/help/gitlab-ci-multi-runner/)

[参考](https://www.jianshu.com/p/2b43151fb92e)
##### 安装
```
yum install gitlab-ci-multi-runner
```
##### 注册
安装好 GitLab Runner 之后，启动 Runner 然后和 CI绑定就可以了：

- 打开GitLab中的项目页面，在项目设置中找到 runners

运行`gitlab-ci-multi-runner register`
```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
your url
Please enter the gitlab-ci token for this runner:
your token
Please enter the gitlab-ci description for this runner:
[opstest]: 
Please enter the gitlab-ci tags for this runner (comma separated):
opsdoc  
Whether to run untagged builds [true/false]:
[false]:true  #此处我选择的是true，不然每次push还得弄tag 
Whether to lock Runner to current project [true/false]:
[false]: 
Registering runner... succeeded
```

- 输入 CI URL

- 输入 Token

- 输入 Runner 的描述
- 输入tags 可以有多个

- 选择 Runner 的类型，选 Shell 

**runner 进程**
```
/usr/bin/gitlab-ci-multi-runner run --working-directory /home/gitlab-runner --config /etc/gitlab-runner/config.toml --service gitlab-runner --syslog --user gitlab-runner
```

##### 删除注册
```
gitlab-runner unregister --url http://192.168.1.2/ci --token 387ed6c05fef248d2183f9f45b9cda 
gitlab-runner unregister --name test-runner ## 通过name取消注册 
gitlab-runner unregister --all-runners ##删除所有注册runner 
 ```
 
- 什么情况下需要注册Shared Runner？\
比如，GitLab上面所有的工程都有可能需要在公司的服务器上进行编译、测试、部署等工作，这个时候注册一个Shared Runner供所有工程使用就很合适。

- 什么情况下需要注册Specific Runner？\
比如，我可能需要在我个人的电脑或者服务器上自动构建我参与的某个工程，这个时候注册一个Specific Runner就很合适。

- 什么情况下需要在同一台机器上注册多个Runner？\
比如，我是GitLab的普通用户，没有管理员权限，我同时参与多个项目，那我就需要为我的所有项目都注册一个Specific Runner，这个时候就需要在同一台机器上注册多个Runner。

- 若已经配置好了gitlab-runner了，执行commit，pipeline状态一直是pending，并且提示： 
`This build is stuck, because the project doesn't have any runners online assigned to it. Go to Runners page `\
这个是因为未找到对应的runner导致的，原因一是有可能gitlab-runner注册失败，原因二有可能是.gitlab-ci.yml配置文件里面tags没有匹配到已注册可用的runner。

---
## .gitlab-ci.yml
[官方文档](https://docs.gitlab.com/ee/ci/yaml/README.html#pages)\
http://192.168.1.116/ci/lint 监测语法正确与否
```
Note：
不同分支下会按照该分支所对应的.gitlab-ci.xml文件执行。
```
### 常用语法：
#### when
Define when to run job. Can be `on_success`, `on_failure`, `always` or `manual`

1. `on_failure`:stage中至少有一个失败，则运行当前stage。
2. `always`:不论其他stage成功或失败，都会运行当前stage。
3. `manual`：在这一步停止，需要在pipelines页面手动运行该stage。


```yml
stages:
  - test
  - build
  - fail
  - always
job1:
  stage: test
  script:
    - echo "job1 issue 3"
    - duang
  allow_failure: true  ##允许失败不影响其他部分
  only:
    - master
  tags:
    - tag1
    
#job2需要手动在界面确认。
job2_start-:
  stage: start-
  script:
    - 
  tags:
    - lng_microservice1
  when: manual
    
#有一步失败时就会运行
job3:
  stage: fail
  script:
    - echo "fail test"
    - touch /home/gitlab-runner/fail
  tags:
    - tag2
  when: on_failure

#job4不论如何都会运行
job4:
  stage: always
  script:
    - touch /home/gitlab-runner/always
  when: always
```
#### cache

**cache用来指定需要在job之间缓存的文件或目录。只能使用该项目工作空间内的路径。\
定义需要缓存的文件。每个 Job 开始的时候，Runner 都会删掉 .gitignore 里面的文件**\
如果cache定义在jobs的作用域之外，那么它就是全局缓存，所有jobs都可以使用该缓存。\
每个 Job 开始的时候，Runner 都会删掉 .gitignore 里面的文件。\
缓存了的文件除了可以跨 Jobs 使用外，还可以跨 Pipeline 使用。
```yml
stages:
  - install
  - build
  - kill
  - start
  
cache:
  paths:
    - node_modules/
    - dist/
    
install:
  stage: install
  script:
    - npm install

    
build:
  stage: build
  script:
    - npm run build

kill_tomcat_web:
  stage: kill
  script:
    - lsof -i:8082|grep -v PID|awk '{print $2}' |xargs kill -9
  allow_failure: true    

start_web:
  stage: start
  script:
    - cp -r dist/* /usr/local/tomcat-ci/tomcat-video-web/webapps/ROOT/
    - /usr/local/tomcat-ci/tomcat-video-web/bin/startup.sh
  after_script:
    - lsof -i:8082
```


```yml
stages:
  - build
  - killall
  - start-all
  - start-api
  - start-upms
  - start-message
  - start-task

cache:
  paths:
    - lng-api/lng-api-interface/target/*
    - lng-api/lng-api-model/target/*
    - lng-api/lng-api-service/target/*
    - lng-common/target/*
    - lng-message/target/*
    - lng-pay/target/*
    - lng-upms/lng-upms-interface/target/*
    - lng-upms/lng-upms-service/target/*
    - lng-time-task/target/*

job1_build:
  stage: build
  script:
    - ./lng build
  tags:
    - lng_microservice1

job2_killall:
  stage: killall
  script:
    - ps ax|grep -e "lng-api-service.*jar" -e "lng-message.*jar" -e "lng-time-task.*jar" -e "lng-upms-service.*jar"|grep -v grep|awk '{print $1}'|xargs kill
  allow_failure: true
  tags:
    - lng_microservice1
  when: manual
 

job3_start-all:
  stage: start-all
  script:
    - ./lng startd
  tags:
    - lng_microservice1
  when: manual
  
job4_start-api:
  stage: start-api
  script:
    - ./lng-api/lng-api-service/bin/bootstrap startd
  tags:
    - lng_microservice1
  when: manual

job5_start-upms:
  stage: start-upms
  script:
    - ./lng-upms/lng-upms-service/bin/bootstrap startd
  tags:
    - lng_microservice1
  when: manual

job6_start-message:
  stage: start-message
  script:
    - ./lng-message/bin/bootstrap startd
  tags:
    - lng_microservice1
  when: manual

job7_start-task:
  stage: start-task
  script:
    - ./lng-time-task/bin/bootstrap startd
  tags:
    - lng_microservice1
  when: manual

 


```

---
### gitlab update fork

提示：跟上游仓库同步代码之前，必须配置过 remote，指向上游仓库 。

1. 指定一个上游仓库

#upstream为你自己为同步源取的别名，方便自己记住
```
git remote add upstream git@192.168.1.116:lng/lng-microservice.git
```
（2）从上游仓库获取到分支，及相关的提交信息，它们将被保存在本地的 upstream/master 分支

git fetch upstream\
（3）切换到本地的 master 分支

git checkout master\
（4）把 upstream/master 分支合并到本地的 master 分支，本地的 master 分支便跟上游仓库保持同步了，并且没有丢失你本地的修改

git merge upstream/master\
（5）将本地修改的文件加入git，注意add后面的点“ · ”

git add . \
（6）添加修改注释，简单描述你修改的内容

git commit -m "add Notes" \
（7）同步后的代码仅仅是保存在本地仓库，记得 push 到 Github

git push -u origin master



```yml
stages:
  - kill-all
  - build
  - rebuild
  - start-all
  - check
  
  
cache:
  paths:
    - lng-api/lng-api-interface/target/
    - lng-api/lng-api-model/target/
    - lng-api/lng-api-service/
    - lng-common/target/
    - lng-message/
    - lng-pay/target/
    - lng-upms/lng-upms-interface/target/
    - lng-upms/lng-upms-service/
    - lng-time-task/



kill-all:
  stage: kill-all
  script:
    - ./lng stop
  allow_failure: true

build:
  stage: build
  script:
    - ./lng build
    
build:
  stage: rebuild
  script:
    - rm -fr /home/gitlab-runner/cache/lng/lng-microservice/default/cache.zip
    - ./lng build

start-all:
  stage: start-all
  script:
    - ./lng startd

  
check-status:
  stage: check
  script:
    - lsof -i:1111
    - curl http://192.168.1.123:1111/manage/upms/is-started
    - lsof -i:8989
    - curl http://192.168.1.123:8989/manage/api/is-started
    - lsof -i:8001
    - curl http://192.168.1.123:8001/message/manage/message/is-started
    - lsof -i:8994
    - curl http://192.168.1.123:8994/manage/task/is-started
  allow_failure: true
  when: manual
```

```
stages:
  - kill-all
  - build
  - rebuild
  - start-all
  - check
  
  
cache:
  paths:
    - lng-api/lng-api-service/target/
    - lng-api/lng-api-service/lng-api-log
    - lng-api/lng-api-service/process.pid
    - lng-message/target/
    - lng-message/lng-message-log/
    - lng-message/process.pid
    - lng-upms/lng-upms-service/target/
    - lng-upms/lng-upms-service/lng-upms-log/
    - lng-upms/lng-upms-service/process.pid
    - lng-time-task/target/
    - lng-time-task/lng-time-task-log/
    - lng-time-task/process.pid



kill-all:
  stage: kill-all
  script:
    - ./lng stop
  allow_failure: true

build:
  stage: build
  script:
    - ./lng build
    
rebuild:
  stage: rebuild
  script:
    - rm -fr /home/gitlab-runner/cache/lng/lng-microservice/default/cache.zip
    - ./lng build
  when: on_failure

start-all:
  stage: start-all
  script:
    - ./lng startd

  
check-status:
  stage: check
  script:
    - lsof -i:1111
    - curl http://192.168.1.123:1111/manage/upms/is-started
    - lsof -i:8989
    - curl http://192.168.1.123:8989/manage/api/is-started
    - lsof -i:8001
    - curl http://192.168.1.123:8001/message/manage/message/is-started
    - lsof -i:8994
    - curl http://192.168.1.123:8994/manage/task/is-started
  allow_failure: true
  when: manual
```

jmeter
```
jmeter -n -t ~/jmeter/LNG-v1.jmx -l lng-microservice.csv|grep "Err:"|awk '{print $14 "\t" $15 "\t" $16}'

 grep false lng-microservice.csv|awk 'BEGIN {FS=","} {print $3 "\t" $6}'
 
 
cache:
  paths:
    - lng-api/lng-api-service/target/
    - lng-api/lng-api-service/lng-api-dubbo-cache
    - lng-upms/lng-upms-service/target/
    - lng-upms/lng-upms-service/lng-upms-dubbo-cache
    - lng-message/target/
    - lng-message/lng-message-dubbo-cache
    - lng-time-task/target/
    - lng-time-task/lng-time-task-dubbo-cache

 
```

#### docker gitlab-runner
```
运行runner：
docker run -d --name gitlab-runner --restart always \
    -v /srv/gitlab-runner/config:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:latest
进入：
docker exec -it gitlab-runner gitlab-ci-multi-runner register
```



