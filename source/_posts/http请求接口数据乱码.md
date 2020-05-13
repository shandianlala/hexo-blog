---
title: http请求接口数据乱码
tags:
  - 乱码
  - JAVA
  - HTTP
abbrlink: 922954aa
date: 2017-11-30 22:07:29
---

今天因为项目里的一个天气接口挂掉了，然后更换新的接口，发现了一个较稳定、免费的接口；但在用java解析数据的时候，始终是乱码,很头疼，以此记录。
<!-- more -->
- **天气接口地址** 
    [天气接口地址](http://wthrcdn.etouch.cn/WeatherApi?city=%E7%8F%A0%E6%B5%B7)
> 天气接口返回的数据
![](http://ozux0lqfa.bkt.clouddn.com/%E4%B8%AD%E5%8D%8E%E4%B8%87%E5%B9%B4%E5%8E%86%E5%A4%A9%E6%B0%94%E6%8E%A5%E5%8F%A3%E8%BF%94%E5%9B%9E%E7%9A%84%E6%95%B0%E6%8D%AE.png)
> 后台java代码
```
URL url = new URL("http://wthrcdn.etouch.cn/WeatherApi?city=" + cityName); 
connectionData = url.openConnection();
connectionData.setConnectTimeout(1000);
try { 
    br = new BufferedReader(
        new InputStreamReader(connectionData.getInputStream(), "UTF-8")); 
    sb = new StringBuilder(); 
    String line = null; 
    while ((line = br.readLine()) != null) 
        sb.append(line); 
} catch (SocketTimeoutException e) { 
    System.out.println("连接超时"); 
} catch (FileNotFoundException e) { 
    System.out.println("加载文件出错"); 
} 
String datas = sb.toString();  
```
但`datas`里面的数据是乱码，很是头疼。
- 仔细检查，发现问题
![](http://ozux0lqfa.bkt.clouddn.com/%E6%8E%A5%E5%8F%A3%E8%AF%B7%E6%B1%82%E7%9A%84%E6%95%B0%E6%8D%AE%E4%B9%B1%E7%A0%81%E6%B3%A8%E6%84%8F%E7%9C%8B%E8%BF%94%E5%9B%9E%E7%9A%84%E4%BF%A1%E6%81%AF%E7%BC%96%E7%A0%81.png)
每次请求返回的数据的 Content Encoding是**gzip**;故需要**GZIPInputStream**封装下`connectionData.getInputStream()`。
调整如下：
```
br = new BufferedReader(new InputStreamReader(
            new GZIPInputStream(connectionData.getInputStream()), "UTF-8")); 
```
再次调试表示问题解决了，在使用第三方提供的接口的时候要注意他们返回信息的编码格式，然后使用对应的编码格式解析,谨记！



