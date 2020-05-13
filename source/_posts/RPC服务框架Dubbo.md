---
title: RPC服务框架Dubbo
abbrlink: b7aae4ad
date: 2017-12-14 15:55:02
tags: [JAVA,框架,dubbo]
---

[Dubbo](http://dubbo.io/) |dʌbəʊ| 是一个高性能，基于Java的RPC（Remote Procedure Call：远程过程调用）框架，由阿里巴巴开源。和许多RPC系统一样，dubbo基于定义一个服务的思想，指定可以通过参数和返回类型远程调用的方法。在服务器端，服务器实现这个接口并运行一个dubbo服务器来处理客户端调用。在客户端，客户端有一个存根，提供与服务器相同的方法。
<!-- more -->
**注意：**
dubbo |dʌbəʊ| 音译`达博`，而不是`肚博`，专业名词还是念对比较好。

### 一、dubbo模块介绍
- **dubbo-common** 公共逻辑模块，包括 Util 类和通用模型。
- **dubbo-remoting** 远程通讯模块，相当于 Dubbo 协议的实现，如果 RPC 用 RMI 协议，则不需要使用此包。
- **dubbo-rpc** 远程调用模块，抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理。
- **dubbo-cluster** 集群模块， 将多个服务提供方伪装为一个提供方， 包括： 负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。
- **dubbo-registry** 注册中心模块，基于注册中心下发地址的集群方式，以及对各种注册中心的抽象。
- **dubbo-monitor** 监控模块，统计服务调用次数，调用时间的，调用链跟踪的服务。
- **dubbo-config** 配置模块，是 Dubbo 对外的 API，用户通过 Config 使用 Dubbo，隐藏Dubbo 所有细节。
- **dubbo-container** 容器模块，是一个 Standlone 的容器，以简单的 Main 加载 Spring启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。

```
<modules>
    <module>hessian-lite</module>
    <module>dubbo-common</module>   <!-- 公共模块 -->
    <module>dubbo-container</module><!-- 容器 -->
    <module>dubbo-remoting</module>
    <module>dubbo-rpc</module>      <!-- rpc远程调用 -->
    <module>dubbo-filter</module>
    <module>dubbo-cluster</module>  <!-- 集群 -->
    <module>dubbo-registry</module> <!-- 注册中心 -->
    <module>dubbo-monitor</module>
    <module>dubbo-config</module>   <!-- 配置spring，提供dubbo的api -->
    <module>dubbo</module>
    <module>dubbo-simple</module>
    <module>dubbo-admin</module>    <!-- 配置dubbo-admin可视化界面 -->
    <module>dubbo-demo</module>     <!-- 用户测试demo模块 -->
    <module>dubbo-plugin</module>
</modules>
```

**参考：**
http://blog.csdn.net/u011659172/article/details/51491518