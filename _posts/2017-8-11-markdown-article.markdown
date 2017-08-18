---
layout:    post
title:     "markdown简单介绍"
subtitle:   " \"一个优雅的编辑器\""
date:       2017/08/11
author:     "CaoFan"
header-img: "img/markdown-article-bg.jpg"
catalog: true
tags:
  - markdown

---

>"markdown"



## 前言

对于markdown来说，我很久之前就听说过，但是没有完整的读过文档自己学习过，今天我集中学习了一下，把自己的学习成果记录下来。以后有更多的学习体会，我会回来补充的，嘻嘻。


<p id="build"></p>
- - -

## 正文

markdown官方给出的是致力于使易于阅读和易于创作文档变得可行.（Markdown is intended to be as easy-to-read and easy-to-write as is feasible.），其实看了markdown的 [中文文档](https://markdown-zh.readthedocs.io/en/latest/) ，它的语法是比较简单掌握的。出于对一个知识掌握来总结一下它的优点和适用范围：  

* 它的出现是为了让文字工作者更加方便，你只需要单纯的编写文本，会帮助你自动排版，你只需了解它简单的语法
* markdown对于格式的转换比较的方便
* 跨平台
* 适用与文字工作者，或者需要编写大量文字，比如博客编写，公众号。。
* 现在越来越多的网站都支持Markdown，足以说明它的好

### 常用语法总结  

#### 1.标题 

> 标题1   
  =====   
  标题2     
  -----    

来表示一级标题和二级标题，其中`=`和`-`可以写任意多个。对于html来说，相当于h1和h2标签。   

> #一级标题  
  ##二级标题  

来表示多级标题1-6个井号，表示6级标题。对于html来说就是相当于h1-h6标签。   

#### 2.段落  

段落内强制换行换行： 只需要在末尾打两个以上的空格或者tab，再回车继续打下面的内容，即完成了换行。  
相当于html里面的br标签。  

#### 3.区块引用  

>     >balabalabalabala
>     again balabalabala
>     .......   

来表示一段文字，不会强制换行。  
效果   
>balabalabalabala
again balabalabala
.......  

>     >区块
>     >>嵌套    

效果  
>区块
>>嵌套   

#### 4.代码块  

只需要将每一行都缩进4个空格，或者一个tab(对于tab设置为2个空格的人那就是两个喽)。   
效果：    

    function test() {
      return markdown;
    }

代码块不同于普通段落，即你写的什么样展示出来就是什么样。  
对于html，`<pre/>`和`<code/>`来包裹代码块。   

#### 5.列表  

无序列表：使用`星号*`，`加号+`，`减号-`，相当于html的ul，配合li标签。    
效果：   
>* 1
* 2
* 3   

有序列表：使用`数字.`（不论你的数字是不是连续的，有序的，markdown都会解析为序号为1，2，3，4...），相当于html的ul，配合li标签。  
效果：   
>1. 4
2. 5
3. 6   

注意列表内容和列表序号之间至少有一个空格隔开。  

#### 6.黑体，斜体，分割线，反斜杠\，标记符``

>黑体： \**黑体**\ | \__黑体__\   
斜体： \*斜体*\   | \_斜体_\   
分割线：这一行上只有三个以上的`星号*`,`下划线_`(中间有空格也可以，多少个都可以), 还有一个特殊`减号-`,必须有空格。   
反斜杠：反转义，把符号变成普通符号。    
标记符：\``\，把\``\引起来的加块重点标记。  

#### 7.链接   

* \[文本内容](链接，比如http://webcaofan.com/ "title属性值"),  其中首尾都可以写内容。   

效果：   
caofan的[blog](http://webcaofan.com/ "myTitle")欢迎指点。   

* \[文本内容][link]  
其中[link]: 链接 "title属性值"；其实和第一种是一样的，只不过把链接拆开写了，再定义一次，一般紧跟着定义。  

对于html来说相当于a标签。

#### 8.图片

* !\[img|img的alt属性值]\(图片地址路径 "img的title属性值")   
* !\[img|img的alt属性值][ImgId]   
其中[ImgId]：img路径 "mg的title属性值"。  

markdown无法指定图片的尺寸。   


### markdown的编译器

可以参考[这里](https://github.com/fex-team/dora/blob/master/doc/research.md), 对于前端的话，你可以使用里面的js库，直接引入就可以使用了，非常方便。  

## 总结

还有一些高级的用法，目前的自己还用不上，就先记笔记到这，自己这么写一遍感觉对它熟练了一圈，嘻嘻~





