---
layout:    post
title:     "Arr and Object"
subtitle:   " \"重新认识vue\""
date:       2017/08/25
author:     "CaoFan"
header-img: "img/arr-object-func.png"
catalog: true
tags:
  - javascript

---
>"javascript数组和对象基本用法"

## 前言

前几天面试被打击到了，我觉的对于校招的我们毕业生，最重要的是基础知识，突然觉得自己一直在学在总结框架的源码，该来复习总结一下前面的基础知识了。今天想总结一下数组和对象的常用方法以及一些实现方法。

## 正文       

1.基本值类型：number、string、boolean、undefined、function、object，前五个为不可改变的原始值不可改变（栈内存，first in last out），后一种为引用值（堆内存）
2.判断类型：typeof()打印number、string、boolean、undefined、function、object（null，数组，对象）；其中如果一个变量未经过定义就typeof()打印undefined，但是不会报错；typeof('undefined')打印string，typeof(undefined)打印undefined。                 
`数组和对象`：Object.prototype.toString.call([])打印'[object Array]'  |  Object.prototype.toString.call([])打印'[object Object]'。    
             [] instanceof Array打印false  |  {} instanceof Array打印true。       
3.类型转换：
* 显示类型转换：     
> Number()  转换为数字，如果不能即为NaN；Number(false/null)打印0，Number(undefiend)打印NaN。     
  String()  转换为字符串，如果参数为数组会变为字符串，对象则会打印"[object Object]"。             
  Boolean()  转换为boolean值。           
  parseInt()  转换数字，进制转换；会以字符串的第一位向后找，找到非数字为截止，并且把之前返回；如果第一位就非整数，直接返回NaN；parseInt(null/undefiend/false/true)都打印NaN；parseInt(str, radix),其中radix表示进制转换2-36，0表示还是十进制转换，1表示解析结果为NaN。        
  num.toString()  转换为字符串，num为undefined和null是会报错，同时数组会变为字符串，对象则会打印"[object Object]"。          

* 隐式类型转换：   
> 
               


### Array
