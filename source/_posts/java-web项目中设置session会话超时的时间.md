---
title: java web项目中设置session会话超时的时间
abbrlink: e95b6d7d
date: 2017-12-18 20:12:39
tags: [JAVA]
---
一般系统登录后，都会设置一个当前session失效的时间，以确保在用户长时间不与服务器交互，自动退出登录，销毁session，下面我罗列三种具体设置的方法：
<!-- more -->
### 一、在web容器中设置session失效的时间（以tomcat为例）
在`apache-tomcat-8.0.72\conf\web.xml`中设置，以下是`apache-tomcat-8.0.72`中默认配置：
```
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```
tomcat默认session超时时间为30分钟，可以根据需要修改，负数或0为不限制session失效时间。
### 二、在项目工程的web.xml中设置session失效的时间
```
<session-config>
    <session-timeout>1</session-timeout>
</session-config>
```
这里配置1分钟后session失效。
### 三、通过java代码设置session失效的时间
```
request.getSession().setMaxInactiveInterval(1*60);//以秒为单位，即在没有活动1分钟后，session将失效
```
**注意：**三种方式优先等级：`1 < 2 < 3`

### 四、监听session失效
这里就需要用到监听器了，当session因为某种原因失效后，监听器就可以监听到，然后执行监听器中定义好的程序就可以了；监听器接口为：`HttpSessionListener`，有`sessionCreated()`和`sessionDestroyed()`两个方法，自己可以新建一个类实现`HttpSessionListener`接口，然后分别重写这两个方法：
- `sessionCreated()`方法：
在session创建完毕后执行的；
- `sessionDestroyed()`方法：
在session失效时执行的；
#### 示例代码
```
package com.sdll.blog.utils;
import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;

/**
 * session监听器类，监听sesssion的创建和失效
 * @author chengxw
 */
public class UserOnlineListener implements HttpSessionListener{

    @Override
    public void sessionCreated(HttpSessionEvent se) {

        System.out.println("session is sessionCreated ======================session创建完毕");
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent se) {

        System.out.println("session is sessionDestroyed =====================session销毁完毕");
    }
}
```
上面的监听器类写好后，在项目工程的`web.xml`添加`listener`进行声明，如下：
```
<listener>
    <listener-class>com.sdll.blog.utils.UserOnlineListener</listener-class>
</listener>
```

**参考：**
https://www.cnblogs.com/diewufeixian/p/4221747.html