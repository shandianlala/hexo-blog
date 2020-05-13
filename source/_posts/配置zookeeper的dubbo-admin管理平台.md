---
title: 配置zookeeper的dubbo-admin管理平台
tags:
  - dubbo
  - zookeeper
  - dubbo-admin
abbrlink: fda57de0
date: 2017-12-13 15:37:32
---
因为zookeeper的启动是通过命令来操控的，我们无法看到注册中心存在了哪些`provider(提供者)`或`consumer（消费者）`，这时就需要借助dubbo-admin可视化管理平台进行实时的查看，同时也可以通过这个平台来管理`provider(提供者)`或`consumer（消费者）`。
<!-- more -->
### 一、本机环境
> jdk:1.8
操作系统：windows 7
zookeeper :zookeeper-3.4.8
tomcat：apache-tomcat-8.0.31
dubbo-version：2.5.8
maven：3.3.9

### 二、dubbo-admin 源码下载、编译
- 在github上下载dubbo源码，[alibaba/dubbo](https://github.com/alibaba/dubbo)，下载下来后进行解压，本文与2017年12月13日记录，后期dubbo版本的更新可能会影响本文所描述的效果呈现。下图是解压后的路径。
![dubbo解压后的目录](http://ozux0lqfa.bkt.clouddn.com/dubbo-master%E8%A7%A3%E5%8E%8B%E5%90%8E%E7%9B%AE%E5%BD%95.png)
- 进入 `dubbo-admin` 文件夹，点击文件夹外的空白处，按住 `shift` 键，鼠标点击右键，选择`在此处打开命令行窗口`，进入cmd命令行。输入一下命令
```
mvn package -Dmaven.skip.test=true
```
出现如下结果，说明打包成功；
![dubbo-admin编译打包](http://ozux0lqfa.bkt.clouddn.com/dubbo-admin%E7%BC%96%E8%AF%91%E6%88%90%E5%8A%9F%E6%95%88%E6%9E%9C.png)
去`dubbo-admin`文件下的`target`目录找`dubbo-admin-2.5.8.war`的war包，把war改下名为`dubbo-admin`（方便后面浏览器访问）后扔到tomcat里面的webapps目录中，使用`startup.bat`启动tomcat。注意端口冲突。解决tomcat的端口冲突这里就不细讲了。
**注意：**
先启动zookeeper，再启动项目dubbo-admin的tomcat
此处是我编译打好的war包：[dubbo-admin.war](http://download.csdn.net/download/shandianlala/10157122)

### 三、测试安装效果
打开浏览器输入`http://localhost:8080/dubbo-admin/`，提示需要输入用户名和密码。用户名和密码在部署`dubbo-admin`的tomcat目录里面。进入目录 `apache-tomcat-8.0.31\webapps\dubbo-admin\WEB-INF` 中，打开`dubbo.properties`配置文件，文件内容如下：
```
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root      #账户:root，密码:root
dubbo.admin.guest.password=guest    #账户:guest，密码:guest
```
这里你随便选择一个账户密码进行登录就行。
![dubbo-admin登录成功首页](http://ozux0lqfa.bkt.clouddn.com/dubbo-admin%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F%E5%90%8E%E9%A6%96%E9%A1%B5.png)

**参考：**
http://blog.csdn.net/evankaka/article/details/47858707#comments