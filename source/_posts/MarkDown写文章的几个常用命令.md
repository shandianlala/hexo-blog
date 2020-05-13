---
title: MarkDown写文章的几个常用命令
tags:
  - MarkDown命令
abbrlink: cc0e19bd
date: 2017-11-25 11:11:39
---
初次接触MarkDown的时候，记住这几个常用的命令，可以达到事半功倍的效果。
<!-- more -->

- 文章太长，在想截断的地方换行加上：
> `<!-- more -->`
- 引用，与内容保留一个空格:
> "`>`"
- 标题，#和「一级标题」之间建议保留一个字符的空格
> `# 一级标题`              
> `## 二级标题`
> `### 三级标题`
> `#### 四级标题`
> `##### 五级标题`
> `###### 六级标题`
    
- 图片
> `![你好世界](https://www.sdll.club/img/hellowold.jpg)`
- 链接
> `[闪电拉拉个人博客](http://www.sdll.club)`
**效果** **:**  [闪电拉拉个人博客](http://www.sdll.club)
- 代码
单行代码引用(左右各一个符号，把代码包裹起来)：
`print('代码符号在左上角esc键下面！');`
多行代码引用(上下各三个符号，把代码包裹起来)：
```
public static String getName() {
    System.out.println("代码符号在左上角esc键下面！！！")
}
```
- 列表，与内容之间保持一个空格
> `-` 你好，我是无序列表
> `1.` 我是有序列表
- 粗体斜体
`*闪电侠*`        效果(斜体)：*闪电侠*
`**闪电侠**`     效果(粗体)：**闪电侠**
- 表格
```
dog | bird | cat
----|------|----
foo | foo  | foo
bar | bar  | bar
baz | baz  | baz
```
> **效果**：

dog | bird | cat
----|------|----
foo | foo  | foo
bar | bar  | bar
baz | baz  | baz

参考文章：
http://www.isetsuna.com/hexo/writing-image/
http://www.jianshu.com/p/q81RER

**推荐两款MarkDown在线编辑器：**
[StackEdit在线编辑器](https://stackedit.io/editor#)
[Cmd Markdown 编辑阅读器](https://www.zybuluo.com/mdeditor)