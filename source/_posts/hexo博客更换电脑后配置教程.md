---
title: hexo博客更换电脑后配置教程
abbrlink: 99f8ffac-hexo-blog-change-computer-teach
date: 2017-12-27 21:25:57
tags: [配置,hexo博客配置]
---
更换电脑了，需要把hexo博客从一台电脑迁移到另外一台电脑上，该怎么在新电脑上配置 hexo 博客环境呢？
<!-- more -->
在网上也查了好些资料，但遇到好多坑，以下教程会清空掉原来的 commit 记录，因为我发现在迁移前我的 commit 有好多次的，迁移后打开 github 发现 commit 只有1次，但**博客内容不影响**。

### 一、准备工作
- 系统安装 Node.js 和 Git，如果你看到此文，说明这个步骤你是会的，这里就不多讲了，不会的请移步到[hexo博客安装教程](http://sdll.club/blogs/fac581d3-hexo-install/)。
- 本文中的所有命令均在 Git Bash 中执行，而不是在 CMD 中。
- 拷贝旧电脑中的整个 hexo 博客目录
![博客目录](http://ozux0lqfa.bkt.clouddn.com/hexo%E5%8D%9A%E5%AE%A2%E6%A0%B9%E7%9B%AE%E5%BD%95.jpg)


### 二、在新环境上（新电脑）部署新的 Hexo
Node.js 和 Git 安装好后，接着开始以下步骤。
- 在你想要部署 hexo 博客的文件夹里面空白处，点击鼠标右键，选择`git bash`，运行命令：
```
hexo init
```
命令执行完后，会生成一个新的 Hexo 目录。
- 将拷贝的 packge.json 文件复制到新生成的 hexo 目录下，覆盖原来的 packge.json 文件。
- 执行以下命令： 
```
npm install
```
安装相关依赖包。
- 拷贝相关文件夹和文件到**新电脑的对应目录**中
```
需要拷贝以下文件夹：
旧电脑的 hexo/source 文件夹里的所有文件到 新电脑的 hexo/source；
hexo/scaffolds
hexo/themes/ 你的主题文件夹 
需要拷贝文件：
hexo/_config.yml
packge.json  
```
文件替换好了之后，开始下一步
### 三、查看效果及部署。
- 运行 `hexo g` 生成博客。
- 运行 `hexo server` 查看博客。
- 打开 `http://localhost:4000/`，没问题的话，执行下一步。
- 执行 `hexo d` 推送到远程仓库部署。
### 四、结束语
至此，博客的整个迁移过程完美结束。我想说 `git`真的很重要！！！

**参考**
https://www.cnblogs.com/JinyaoLi/p/4672376.html