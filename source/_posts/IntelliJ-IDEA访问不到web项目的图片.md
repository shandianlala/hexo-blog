---
title: IntelliJ IDEA访问不到web项目的图片
tags:
  - IntelliJ IDEA
  - 配置
abbrlink: 739343a9
date: 2017-12-06 20:56:03
---

> Idea被认为是业界最好的Java开发工具，要使得它乖乖听话，任我摆布，还需要花时间调教调教它。

今天在使用idea维护之前的博客项目的时候，发现原来在eclipse上跑的好好的项目突然在idea上就不好使了，部署的web项目，富文本编辑器的图片上传报错，页面访问不到图片。下面开始我解决问题的整个步骤。
<!-- more -->
### 运行项目，控制台打印的日志乱码。
在tomcat的配置页面中`VM options` 选项中加入
![](http://ozux0lqfa.bkt.clouddn.com/vmoption.png)
```
-Dfile.encoding=UTF-8
```

### 项目成功跑起来了，但用富文本编辑器上传图片，页面报错。
- idea中tomcat发布项目的默认路径是项目所在地里的target目录里面。
我的做法是更改项目发布路径到tomcat目录下的webapps目录下面，下面更改发布路径到tomcat。
![](http://ozux0lqfa.bkt.clouddn.com/idea_%E4%BF%AE%E6%94%B9%E5%8F%91%E5%B8%83%E8%B7%AF%E5%BE%84%E4%B8%BATomcat%E7%9A%84webapps1.png)
![](http://ozux0lqfa.bkt.clouddn.com/idea-tomcat%E5%8F%91%E5%B8%83%E7%9B%AE%E5%BD%952.png)
这样之后，在重新运行项目，项目就会发布到tomcat的webapps目录里。

### 配置图片资源文件
- **sdll-blog**项目的图片资源目录我是存储在*upload*文件里面，
![](http://ozux0lqfa.bkt.clouddn.com/idea%E5%9B%BE%E7%89%87%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95.png)
- 现在我需要在idea里面配置upload文件资源。
![](http://ozux0lqfa.bkt.clouddn.com/idea%E9%85%8D%E7%BD%AE%E8%B5%84%E6%BA%90%E7%9B%AE%E5%BD%951.png)
**注意：**`Application context`里面的值配置为`/upload`

![](http://ozux0lqfa.bkt.clouddn.com/idea%E9%85%8D%E7%BD%AE%E8%B5%84%E6%BA%90%E7%9B%AE%E5%BD%952.png)
完成之后，再次运行项目，还是报找不到文件错误，查看控制台打印的日志，发现一个坑，发现在tomcat的安装路径中有部分有空格。文件夹**IDEA SERVER**控制条输出日志空格被解析成**IDEA%2%SERVER**，发现这个问题后，重新配置了个tomcat,重新运行项目，
- 浏览器访问资源文件
![](http://ozux0lqfa.bkt.clouddn.com/idea%E6%B5%8F%E8%A7%88%E5%99%A8%E8%AE%BF%E9%97%AE%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6yes.png)
- 正常访问upload文件夹里面的图片资源了，富文本编辑器也不报错了哦。