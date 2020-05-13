---
title: 使用Nginx和Tomcat来搭建高性能负载均衡集群
abbrlink: b7aae4ad-nginx-tomcat-cluster
date: 2018-02-23 19:00:02
tags: [JAVA,Tomcat,nginx]
---

Nginx (engine x) 是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的。
<!-- more -->
### 一、准备工作。

下载`nginx`和`tomcat`包。请下载对应你的操作系统的包。
- [nginx下载](http://nginx.org/en/download.html)
- [tomcat下载](http://tomcat.apache.org/download-70.cgi)

### 二、实现目标。

实现高性能负载均衡的Tomcat集群：
![](http://ozux0lqfa.bkt.clouddn.com/nginx&&tomcat%E9%9B%86%E7%BE%A4.png)

### 三、实现步骤
- 把解压的tomcat复制两份，如下图所示
![解压目录图](http://ozux0lqfa.bkt.clouddn.com/tomcat%E4%B8%8Enginx%E8%A7%A3%E5%8E%8B%E5%9B%BE.png)
- 把`nginx`包也解压出来。
为了便于管理，把tomcat的与nginx放在同一文件夹目录下。
- 分别修改两个tomcat服务器的端口，避免端口冲突，从而使得两个`tomcat`顺利启动，以下举例修改第一个`tomcat`的端口；打开`conf\server.xml`,修改以下端口。
```
1、
<Server port="8005" shutdown="SHUTDOWN">
改成：
<Server port="18005" shutdown="SHUTDOWN">

2、
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
改成：
<Connector port="18080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

3、
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
改成：
<Connector port="18009" protocol="AJP/1.3" redirectPort="8443" />
```
![](http://ozux0lqfa.bkt.clouddn.com/%E4%BF%AE%E6%94%B9tomcat%E7%9A%84%E6%AC%A2%E8%BF%8E%E9%A1%B5.png)
至此两个`tomcat`顺利启动起来。

- 下面开始修改nginx的配置文件。打开`nginx-1.10.3\conf\nginx.conf`文件。

```
http {
    include       mime.types;
    default_type  application/octet-stream;
    
    # 服务器的集群
    upstream sdll.club { #服务器集群的名字： sdll.club
		# 集群的服务器列表，weight是权重的意思，值越大，分配的概率越大。
		server	127.0.0.1:18080	weight=1;
		server	127.0.0.1:28080	weight=2;
	}

    server {
        listen       9000; #监听端口，可以改成其他端口。
        server_name  localhost; #当前服务的主机

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # location / {
        #    root   html;
        #    index  index.html index.htm;
        # }
		
		location / {
            proxy_pass   http://sdll.club; #这里的值要与服务器集群的值一样
            proxy_redirect  default;
        }
        
```
- 启动`nginx`。

在nginx目录下输入命令`start nginx`,启动`nginx`，紧接着在浏览器中输入`localhost:9000/index.jsp`，如下结果。
![](http://ozux0lqfa.bkt.clouddn.com/nginx%E9%9B%86%E7%BE%A4%E6%B5%8B%E8%AF%95.png)
不停的刷新页面，页面会在**Home 11**和**Home 22**切换中，随着访问次数的增加，**Home 22**出现的次数与**Home 11**出现次数的之比接近为**2:1**，这是因为权重起作用了。
```
upstream sdll.club {
	# 集群服务器列表，最终请求会被转发到这里来。
	server	127.0.0.1:18080	weight=1;
	server	127.0.0.1:28080	weight=2;
}
```
### 四、总结
至此，整个步骤结束。

**参考引用：**
http://blog.csdn.net/xlgen157387/article/details/49781487