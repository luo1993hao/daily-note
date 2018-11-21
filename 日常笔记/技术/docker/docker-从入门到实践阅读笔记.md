### 基本概念
- 镜像
  - “镜像是由多层存储所构成，每一层的东西并不会在下一层被删除”

- 容器
- 仓库
#### 使用镜像
##### 慎用 docker      commit（黑箱操作，维护痛苦，镜像臃肿）
 #### dockerfile
- from指定基础镜像
  - scratch 意味不以任何镜像为基础
- 每一个指令都会建立一层镜像
- 最适于ADD的场合，就是需要自动解压缩的场合
#### 容器
- load与import的区别（import仅保存当时的快照， import保存完整记录）
#### 数据管理
- 两种方式
  - 数据卷
  - 挂载主机目录
#### 镜像加速器
#### docker-compose
-  定义，作用：“它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）”
- 服务，一个应用的容器，实际上可以包括若干相同镜像的容器实例
- 项目，“由一组关联的应用容器组成的一个完整业务单元”       

```sh
/etc/docker/daemon.json
“{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}”
# 之后重启
“sudo systemctl daemon-reload”
“sudo systemctl restart docker”
```