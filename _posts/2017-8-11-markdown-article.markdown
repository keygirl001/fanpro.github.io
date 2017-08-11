---
layout:    post
title:     "markdown简单介绍"
subtitle:   " \"一个优雅的编辑器\""
date:       2017/08/06
author:     "CaoFan"
header-img: "img/markdown-article-bg.jpg"
catalog: true
tags:
  - markdown

---

## 前言

对于markdowm来说，我很久之前就听说过，但是没有完整的读过文档自己学习过，今天我集中学习了一下，把自己的学习成果记录下来。以后有更多的学习体会，我会回来补充的，嘻嘻。


## 正文

markdown官方给出的是致力于使易于阅读和易于创作文档变得可行.（Markdown is intended to be as easy-to-read and easy-to-write as is feasible.），其实看了markdown的 [中文文档](https://markdown-zh.readthedocs.io/en/latest/) ，它的语法是比较简单掌握的。出于对一个知识掌握来总结一下它的优点和适用范围：  

* 它的出现是为了让文字工作者更加方便，你只需要单纯的编写文本，会帮助你自动排版，你只需了解它简单的语法
* markdown对于格式的转换比较的方便
* 跨平台
* 适用与文字工作者，或者需要编写大量文字，比如博客编写，公众号。。
* 现在越来越多的网站都支持Markdown，足以说明它的好

### 常用语法总结  

####1.标题 

markdown：  

> 标题1   
  =====   
  标题2     
  -----    

来表示一级标题和二级标题，其中=和-可以写任意多个。对于html来说，相当于h1和h2标签。   

> #一级标题  
  ##二级标题  

来表示多级标题1-6个井号，表示6级标题。对于html来说就是相当于h1-h6标签。   

####2.段落  

markdown:  

段落内强制换行换行： 只需要在末尾打两个个的空格或者tab，再回车继续打下面的内容，即完成了换行。  
相当于html里面的<br/>标签。  

####3.区块引用  

markdown:  

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


