---
title: hexo博客安装教程
abbrlink: fac581d3-hexo-install
date: 2017-12-30 22:04:05
tags: [hexo博客配置,教程]
---
其实一直都没有准备写这个安装教程，是因为一个搞程序的估计大家都会这些，看看文档大概都能明白；今天写这个教程的缘由是因为想把搭建在公司分配的电脑上的博客迁移到自己的电脑上来，毕竟公司的电脑不是自己的，所以，你们懂就好。哈哈！
<!-- more -->
### 1.下载git安装
[windows环境git安装包下载](https://git-scm.com/download/win)

### 2. 下载Node.js安装
[windows环境Node.js安装包下载](http://nodejs.org/)

### 3.安装好git后，在本地目录hexo(没有的话新建一个)，在文件夹里面点鼠标右键，选择`Git Bash`
输入命令： 
`npm install -g hexo-cli`

### 4.验证是否安装成功，可以看下版本
`hexo version`

### 5.初始化环境
命令：
`hexo init .`

### 6.“.” 是当前目录，
初始化后，目录下会自动生成一些文件

### 7.再用命令(可省略)
`npm isntall` 安装一遍

### 8.启动服务（电脑可能会弹防火墙，确定即可）
命令:
`hexo server`

### 9.好了，现在可以访问了 ： 
`http://localhost:4000/` （命令结果提示是`http://0.0.0.0:4000/`）

### 10.打开hexo目录下的_config.yml文件，最后一行那块
```
------------修改开始--------------------
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:shandianlala/shandianlala.github.io.git
  branch: master
  message: '站点更新:{{now("YYYY-MM-DD HH/mm/ss")}}'
------------修改结束--------------------
```

10-1、配置本机全局git环境(都是命令)
`git config --global user.email "you@example.com"`
`git config --global user.name "Your Name"`

### 11.生成SSH秘钥
命令：
`ssh-keygen -t rsa -C example@gmail.com`



- 11-1.如果没有报错，提示（enter file in which to save the key），一路回车enter

- 11-2.搜索电脑里的文件：id_rsa.pub
用notpad++记事本打开,全选里面的内容，复制。

- 11-3.打开  https://github.com/settings/keys
点击 New SSH key 按钮，title输入框自定义，把刚才复制的内容粘贴到key输入框；

- 11-4.测试连接GitHub、coding
ssh -T git@github.com
ssh -T git@git.coding.net
提示输入yes/no,输入yes,回车。

### 12.使用命令（即 hexo generate 和 hexo deploy）
hexo g  
hexo d  

- 12-1.执行 hexo d 后报错，ERROR Deployer not found: git; 如果没有报错跳过进行下一步
解决：npm install hexo-deployer-git --save
然后再次执行  
`hexo d`

13.生成并发布到github，发布后再github会看到，打开地址访问 yourname.github.io  

14.结束