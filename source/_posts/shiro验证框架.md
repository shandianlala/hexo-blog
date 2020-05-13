---
title: shiro验证框架
tags:
  - JAVA
  - 框架
  - shiro
abbrlink: 61afe343-shiro-authority-frame
date: 2018-03-23 09:20:17
---
Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码学和会话管理。使用Shiro的易于理解的API,您可以快速、轻松地获得任何应用程序,从最小的移动应用程序到最大的网络和企业应用程序。
<!-- more -->
### 配置介绍
- shiro 添加 pom.xml 相关依赖
```
<!-- Shiro SECURITY begin -->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>${shiro.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>${shiro.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>${shiro.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-ehcache</artifactId>
        <version>${shiro.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-quartz</artifactId>
        <version>${shiro.version}</version>
    </dependency>
<!-- Shiro SECURITY end -->
```
- web.xml配置shrio
```
<!-- Shiro安全管理   begin-->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!-- Shiro安全管理    end -->
```
在web.xml中加载spring配置文件applicationContext.xml,在spring配置文件中导入加载shiro.xml，如下所示。
```
<import resource="../shiro/shiro.xml"/>
```
- shiro.xml 配置文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-3.0.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd"
    default-lazy-init="true">
    <description>Shiro Configuration</description>

    <bean id="customRealm" class="club.sdll.shiro.CustomRealm">
        <property name="credentialsMatcher" ref="customCredentialsMatcher" />
    </bean>

    <bean id="customCredentialsMatcher" class="club.sdll.shiro.CustomCredentialsMatcher">
    </bean>
        
    <bean id="simpleCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
       <constructor-arg name="name" value="rememberMe"></constructor-arg>
       <property name="maxAge" value="2592000"/>
    </bean>
    
    <bean id="cookieRememberMeManager" class="org.apache.shiro.web.mgt.CookieRememberMeManager">
       <property name="cookie" ref="simpleCookie"></property>
    </bean>

    <bean id="securityWebManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="customRealm" />
        <property name="cacheManager" ref="cacheManager" />
        <property name="rememberMeManager" ref="cookieRememberMeManager"></property>
    </bean>
    
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityWebManager" />
        <property name="loginUrl" value="/index.jsp?toTop=true" />
        <property name="successUrl" value="/index.jsp" />
        <property name="unauthorizedUrl" value="/index.jsp" />

        <property name="filters">
            <util:map>
                <entry key="authc">
                    <bean
                        class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter" />
                </entry>
            </util:map>
        </property>
        <property name="filterChainDefinitions"><!--authc  -->
            <value>
                /resources/** = anon
                /common/** = anon
                /plugins/framework/** = anon
                /plugins/login/** = anon
                /plugins/sysmgr/**  = authc
                /plugins/comm/**  = authc
                /plugins/business/**  = authc
                /sysmgr/** = authc
                /business/** = authc
                /comm/** = authc
                /main/** = authc  <!-- 用户必须登录才能访问 -->
                /login/** = anon <!-- 任何人都可以访问 -->
                /kaptcha/** = anon
        <!--    /authc/admin =roles[admin]  需要用户有用admin权限才能访问 -->
            </value>
        </property>
    </bean>
    
    <bean id="cacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager" />
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
</beans>
```
- 自定义realm 类继承 `AuthorizingRealm`
```
/**
 * 
 */
package club.sdll.blog.sysmgr.shiro;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import com.eshore.icthala.sysmgr.pojo.bean.authority.SysUser;
import com.eshore.icthala.sysmgr.service.authority.ISysUserService;

/**
 * 
 * Shiro领域
 */
public class CustomRealm extends AuthorizingRealm {
    
    @Autowired ISysUserService sysUserService;

    /**
     * 授权验证
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
         //Object object = this.getAvailablePrincipal(principals);
         SimpleAuthorizationInfo principal = new SimpleAuthorizationInfo(); 
         //principal.setRoles(new HashSet<String>());
         //principal.setStringPermissions(new HashSet<String>(){});
        return principal;
    }

    /**
     * 获取登录验证信息
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo athcInfo = null;
        UsernamePasswordToken athcToken = (UsernamePasswordToken) token;
        try {
            SysUser sysUser = sysUserService.getUserByLoginName(athcToken.getUsername());
            if(null!=sysUser){  //验证是否存在用户
                athcInfo =  new SimpleAuthenticationInfo(sysUser.getLoginName(),sysUser.getLoginPwd(), getName());
            }
            return athcInfo;
        } catch (Exception ex) {
            throw new AuthenticationException(ex);
        }
    }

}

```
- shiro证书匹配
```
/**
 * 
 */
package club.sdll.blog.sysmgr.shiro;

import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authc.credential.SimpleCredentialsMatcher;

import com.eshore.icthala.common.utils.crypt.MD5Util;

/**
 * @author chengxiwang
 * 2017年09月13日 下午2:56:55
 * shiro证书匹配
 */
public class CustomCredentialsMatcher extends SimpleCredentialsMatcher {
    /**
     * 登录密码验证
     */
    @Override
    public boolean doCredentialsMatch(AuthenticationToken token,AuthenticationInfo info) {
        UsernamePasswordToken authcToken = (UsernamePasswordToken) token;
        String passwordFromAthcToken = String.valueOf(authcToken.getPassword()); //
        Object accountCredentials = null;
        try {
            accountCredentials = MD5Util.crypt(getCredentials(info).toString());
        } catch (Exception e) {
            return Boolean.FALSE.booleanValue();
        }
        return equals(passwordFromAthcToken, accountCredentials);
    }

}
```

### 二、使用
```
/**
 * 
 * 登录操作
 * @param page
 * @return
 * @throws ControllerException
 * @throws UtilException
 */
@RequestMapping(value = "loginOn", method = { RequestMethod.GET})
    public ModelAndView loginOn(LoginForm loginForm,RedirectAttributes attrs) throws ControllerException{
         try {
             //验证验证码
            if(!checkKaptcha(loginForm)){
                attrs.addFlashAttribute("message", "验证码错误!");
                return toLogin(attrs);
            }
            //验证用户名和密码
            if(!checkUser(loginForm)){
                attrs.addFlashAttribute("message", "密码错误!");
                return toLogin(attrs); 
            }else{
                formSession(loginForm);
            }
            return toHome();
        }catch (ControllerException ex){
            attrs.addFlashAttribute("message", "登录过程发生异常!请联系系统管理员");
            return toLogin(attrs);
        }
   }
   
   
   /**
     * 
     * 用户名密码验证
     * @param loginForm
     * @return
     */
    private Boolean checkUser(LoginForm loginForm){
        Boolean checkResult = Boolean.FALSE;
        try{
            UsernamePasswordToken token = new UsernamePasswordToken(loginForm.getUserName(),loginForm.getPassword());
            Subject currentUser = SecurityUtils.getSubject();
            currentUser.login(token);
            checkResult = currentUser.isAuthenticated();
            token.setRememberMe(false);
        }catch(Exception ex){
            checkResult = Boolean.FALSE;
        }
        return checkResult;
    }
```



**引用参考：**
https://www.cnblogs.com/learnhow/p/5694876.html