---
title: 在Linux环境下部署java web项目
tags:
  - JAVA
  - Linux
  - web
abbrlink: d54218b1
date: 2017-12-12 22:13:39
---

由于项目需要，最近开发的这个项目的部署工作由我们开发自己来完成，因为项目是部署在Linux环境中，对我来说说还是蛮有吸引力的，之前开发都是在windows环境中完成的，还没有体验过真正项目的部署过程；下面开始记录部署的整个过程。
<!-- more -->
### 一、安装jdk
- 解压jdk安装包
下载linux环境对应的jdk安装包，[jdk下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html),大家选择对应自己电脑的版本下载，这个就不多讲了；下载好了之后解压安装包
```
tar -zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local/java/
```
此处把jdk压缩包解压到指定目录`/usr/local/java/`中
- 配置环境变量



### 二、安装zookeeper
- 首先下载zookeeper的稳定版本。[zookeeper-3.4.10.tar.gz stable版本下载](http://mirrors.hust.edu.cn/apache/zookeeper/stable/)
- 解压压缩包
```
tar -zxvf zookeeper-3.4.10.tar.gz
```
- 把conf目录下的zoo_sample.cfg复制一份,并命名为zoo.cfg
`cp zoo_sample.cfg zoo.cfg`
- 修改zoo.cfg配置文件，此处是单机版zookeeper配置。
```
dataDir=/datatmp/zookeeper/data
dataLogDir=/datatmp/zookeeper/logs
```
保存。如果在对应文件夹`data`、`logs`不存在，请自己在对应的路径下新建。
- 在/etc/profile文件中设置PATH
修改profile文件：`vi /etc/profile`，在文件末尾加入
```
export ZOOKEEPER_INSTALL=/usr/local/zookeeper/zookeeper-3.4.10/
export PATH=$PATH:$ZOOKEEPER_INSTALL/bin
```
到此安装完毕
- 启动zookeeper
```
./zkServer.sh start   #启动服务
./zkServer.sh status  #查看状态STARTED、STOPPED
./zkServer.sh stop    #关闭服务
```
### 三、安装tomcat
- 解压tomcat安装包
```
tar -zxvf apache-tomcat-7.0.77.tar.gz
```
- 启动tomcat
使用`./catalina.sh start`和`./startup.sh`都能启动tomcat。
使用`./catalina.sh stop`或`./shutdown.sh`停止tomcat。

- 查看tomcat的启动日志
```
tail -f catalina.out
```
- 退出查看:`Ctrl + c`,退出tail命令。  

### 四、Centos 7端口、防火墙。
CentOS7使用firewall而不是iptables。所以解决这类问题可以通过添加firewall的端口，使其对我们需要用的端口开放。
#### 4.1、Centos 7 端口
- 查看已经开放的端口：
```
firewall-cmd --list-ports
```
- 开放新的端口
```
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```
**命令含义：**
`zone` #作用域
`add-port=80/tcp` #添加端口，格式为：端口/通讯协议
`permanent` #永久生效，没有此参数重启后失效

#### 4.2、Centos 7防火墙
```
firewall-cmd --state    #查看防火墙状态。得到结果是running或者not running
firewall-cmd --reload   #加载配置,重启firewall
firewall-cmd --permanent --zone=public --list-ports //查看开启的端口
systemctl stop firewalld.service    #停止firewall
systemctl disable firewalld.service     #禁止firewall开机启动
```

### 五、Centos 6.x版本及以下版本
- 开放8080 端口.
用vi打开 `/etc/sysconfig/iptables`,新增如下一行。
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
```
此行必须放在
```
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```
这两行的前面，否则一样无效,编辑完后保存。
重启防火墙，使新增的端口生效。
```
service iptables restart
```
- 查看开启的端口：
```
/etc/init.d/iptables status
```
- 开启、关闭防火墙
    1. 永久性生效，重启后不会复原
        开启： `chkconfig iptables on`
        关闭： `chkconfig iptables off`
    2.  即时生效，重启后复原
        开启： `service iptables start`
        关闭： `service iptables stop`

### 六、linux中切换账户
- 使用root用户切换普通用户时直接`su - 用户名`就可以了；
```
su - sdll 
```
切换到用户:`sdll`, 切换到root用户同理。

> 使用普通用户切换至root用户时`su -`或者`su - root`然后输入root密码就可以了；
在大都的Linux的版本中，都可以使用`su`或者`su -`，但是`su`和`su -`还是有一定的差别的：
su只是切换了root身份，但Shell环境仍然是普通用户的Shell；而`su -`连用户和Shell环境一起切换成root身份了。只有切换了Shell环境才不会出现PATH环境变量错误。`su`切换成root用户以后，pwd一下，发现工作目录仍然是普通用户的工作目录；而用`su -`命令切换以后，工作目录变成root的工作目录了。用`echo $PATH`命令看一下`su`和`su -`以后的环境变量有何不同。以此类推，要从当前用户切换到其它用户也一样，应该使用`su -`命令。

### 七、root权限
在操作的时候尽量不要直接使用root账号来操作，使用其他账号来操作，如果需要权限的话，那就给给自己的账号增加root权限，下面演示如何给`sdll`账号增加root权限。
- 修改`/etc/sudoers` 文件，找到`root`一行，在`root`下面添加一行，如下所示：
```
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
sdll   ALL=(ALL)     ALL
```
修改完毕后，用`sdll`帐号登录，在需要权限才可操作的命令前面加上`sudo `，`sdll`账号以系统管理者的身份执行指令,间接的获取到了root权限。
- Linux sudo命令
Linux `sudo`命令以系统管理者的身份执行指令，也就是说，经由 `sudo` 所执行的指令就好像是 root 亲自执行。
使用权限：在 `/etc/sudoers` 中有出现的使用者。
> $ sudo -u uggc vi ~www/index.html
//以 uggc 用户身份编辑  home 目录下www目录中的 index.html 文件

### 八、vim编辑器
- *Normal Mode* -> *Command-line Mode*
    - `:/filename` 搜索字符串
    - `:w` 保存文件
    - `:w!` 强制保存文件（前提是用户有修改文件访问权限的权限）
    - `:q` 退出缓冲区
    - `:q!` 强制退出缓冲区而不保存
    - `:wq` 保存文件并退出缓冲区
    - `:ZZ`  保存文件并且退出
    - `:wq!` 强制保存文件并退出缓冲区（前提是用户有修改文件访问权限的权限）
    - `:w <filename>` 另存为名为filename文件
    - `:n1,n2 w <filename>` 将n1行到n2行的数据另存为名为filename文件
    - `: x` 如果文件有更改，则保存后退出。否则直接退出。
 
- *Insert Mode* -> *Normal Mode* **OR** *Command-line* -> *Normal Mode*
按下ESC键

### 九、查看进程和杀掉进程
- `ps`命令用于查看当前正在运行的进程。`grep`是搜索,`aux`显示所有状态，例如： 
```
ps -ef|grep java    // 表示查看所有进程里 CMD 是 java 的进程信息
ps -aux|grep java   
```
- `kill` 命令用于终止进程例如： 
```
kill -9 [PID]       // -9 表示强迫进程立即停止
```
通常用 ps 查看进程 PID ，用 kill 命令终止进程.

### 十、输入命令小技巧
有时候会碰到文件夹名特别长的时候，这个时候敲这个文件夹名就有点头疼，现在要进入`apache-tomcat-7.0.82`这个文件夹
```
[sdll@sdll-pc tomcat]$ ls
apache-tomcat-7.0.82
[sdll@sdll-pc tomcat]$ cd apa    //输入 apa 后，按 Tab 键，自动补全 apache-tomcat-7.0.82
```
有木有很方便呀~~~

**参考：**
https://www.cnblogs.com/guzhanyu/p/7921552.html
https://zhidao.baidu.com/question/251895351.html

