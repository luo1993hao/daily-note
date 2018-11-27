# maven使用笔记

## pom.xml配置

pom 代表项目对象模型。它是工作在 Maven 的基本单位。是一个 xml 文件。它始终保存在该项目基本目录中的 pom.xml  文件。

pom 包含的项目是使用 Maven 来构建的，以及它也包含各种配置信息。

POM 也包含了目标和插件。在执行任务或目标时，Maven 会使用当前目录中的pom。读取 pom，得到所需要的配置信息。pom 配置：

属性名称 | 含义
---|---|
modelVersion | 指定了当前 Maven 模型的版本号，对于 Maven2 和 Maven3 来说，它只能是4.0.0。
groupId | 这个应该是公司名或是组织名。一般来说groupId是由三个部分组成，每个部分之间以"."分隔，第一部分是项目用途，比如用于商业的就是"com"，用于非营利性组织的就是"org"；第二部分是公司名；第三部分是你的项目名。
artifactId | Maven构建的项目名，比如项目中有子项目，就可以使用"项目名-子项目名"的命名方式。
version | 版本号。
install | mvn install 是将你打好的 jar 包安装到你的本地库中，一般没有设置过是在用户目录下的 .m2/ 下面
packaging | 项目打包的类型，可以使 jar、war、rar、ear、pom，默认是jar。
build | 表示与构建相关的配置
dependencies | 为了一个项目可以 build 或运行所需要的依赖。
properties | 定义一些配置属性。
repositories | 在project内定义其他的连接的远程仓库。
pluginRepositories | TODO

### 基本属性  

通过 groupId、artifactId、version 三个属性可以定位到一个 maven project 的基本坐标。

注：version 值为 x.x.x-SNAPSHOT 表示该项目还在开发中。

### dependencies

Maven 的一个重要作用就是统一管理jar包。在一个 project 中一代的其他 jar 包在 maven 中称为 dependency。配置如下：

```xml
<dependencies\>
  <dependency\>
    <groupId\>junit</groupId\>
    <artifactId\>junit-dep</artifactId\>
    <version\>4.12</version\>
  </dependency\>
</dependencies\>
```

配置好 dependencies 后， Maven 会自动去远程仓库中下载jar包到本地仓库中。

### properties

properties是用来定义一些配置属性的，例如project.build.sourceEncoding（项目构建源码编码方式），可以设置为UTF-8，防止中文乱码，也可定义相关构建版本号，便于日后统一升级。

### build

build表示与构建相关的配置，比如 build 下有 finalName ，表示的就是最终构建之后的名称。

TODO

## 常用命令

command | 作用
---|---
mvn clean | 清理项目生产的临时文件,一般是模块下的 target 目录
mvn compile | 编译源代码，一般编译模块下的 src/main/Java 目录
mvn package | 项目打包工具,会在模块下的 target 目录生成jar或war等文件
mvn test | 测试命令,或执行 src/test/java/ 下的测试用例.
mvn install | 将打包的 jar/war 文件复制到你的本地仓库中,供其他模块使用
mvn deploy | 将打包的文件发布到远程参考,提供其他人员进行下载依赖
mvn site | 生成项目相关信息的网站
mvn eclipse:eclipse | 将项目转化为Eclipse项目
mvn dependency:tree | 打印出项目的整个依赖树  
mvn archetype:generate | 创建Maven的普通java项目
mvn tomcat:run | 在tomcat容器中运行web应用
mvn jetty:run | 调用 Jetty 插件的 Run 目标在 Jetty Servlet 容器中启动 web 应用
mvn versions:set -DnewVersion=<new_version> | 更新版本

**注意：运行 maven 命令的时候，首先需要定位到 maven 项目的目录，也就是项目的 pom.xml 文件所在的目录。否则，必以通过参数来指定项目的目录。**


## parameters

1. 跳过单元测试

```bash
mvn install -DskipTests
mvn install -Dmaven.test.skip=true
# TODO 添加这两个参数的区别
```
