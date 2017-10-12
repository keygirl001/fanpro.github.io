---
layout:    post
title:     "javascript基本类型及其方法"
subtitle:   " \"js基础知识\""
date:       2017/09/11
author:     "CaoFan"
header-img: "img/arr-object-func.png"
catalog: true
tags:
  - javascript

---

>"It's begining!"  


## 前言

前几天面试被打击到了，我觉的对于校招的我们毕业生，最重要的是基础知识，突然觉得自己一直在学在总结框架的源码，该来复习总结一下前面的基础知识了。今天想总结一下数组、字符串和对象的常用方法以及一些实现方法。

<p id="build"></p>
---

## 正文       

1.`基本值类型：`number、string、boolean、undefined、function、object，前五个为不可改变的原始值不可改变（栈内存，first in last out），后一种为引用值（堆内存）           
2.`判断类型：`typeof()打印number、string、boolean、undefined、function、object（null，数组，对象）；其中如果一个变量未经过定义就typeof()打印undefined，但是不会报错；typeof('undefined')打印string，typeof(undefined)打印undefined。                     
`数组和对象：`Object.prototype.toString.call([])打印'[object Array]'； [] instanceof Array打印false；               
3.`类型转换：`
* 显示类型转换：     
> Number()  转换为数字，如果不能即为NaN；Number(false/null)打印0，Number(undefiend)打印NaN。     
  String()  转换为字符串，如果参数为数组会变为字符串，对象则会打印"[object Object]"。             
  Boolean()  转换为boolean值。           
  parseInt()  转换数字，进制转换；会以字符串的第一位向后找，找到非数字为截止，并且把之前返回；如果第一位就非整数，直接返回NaN；parseInt(null/undefiend/false/true)都打印NaN；parseInt(str, radix),其中radix表示进制转换2-36，0表示还是十进制转换，1表示解析结果为NaN。        
  num.toString()  转换为字符串，num为undefined和null是会报错，同时数组会变为字符串，对象则会打印"[object Object]"。          

* 隐式类型转换：   
> `++/--`：转换为数字，否则为NaN；num++表示先赋值再自增，++num表示先自增再赋值。其中`1++`，会报错，因为原始值不能改变，这时就会有一个问题了，那我var num = 1；num++就不会报错，是通过var声明的变量num实际上是window的一个属性，对象的属性是可以改变的；执行Number()。                  
`+/-`： 转换为数字；执行Number();              
`+`：转换为字符串；字符串链接，执行String()，其中有一个字符串，否则为正常的加减。      
`*、/、-、%`：转换为数字，执行Number()。              
`/>、<、>=、<=、==、!=`：转换为数字，执行Number().           

`其中不发生类型转换的：===、！==`

               


### Array   
1.`构造方法：`      
+ 字面量：var arr = [1,2,3];     
+ 系统构造函数：var arr = new Array(1,2,3)；如果写一个参数表示生成n个undefined；            
+ es6新增的方法：Array.of(1,2,3);解决了上面的问题，Array.of(1)就表示[1];      

2.`es5新增的数组的方法：`         
 `forEach()`：回掉func有三个参数，第一个参数为数组的每个元素，第二个参数为元素的索引，第三个参数为数组本身array必须这样填写。      

    var arr = [1,2,3,4];
    arr.forEach(function (item, index, array) {
      if (item < 4) {
        console.log(item, array);
      }
    })    

  结果：   
![img](/img/in-post/arr-object-func/arr-foreach1.png)     
array.forEach(callback,[thisObject]),第一个参数为回掉函数，第二个参数表示可以改变this指向。             

    var obj = {
      arr: [1,2,3,4,5],
      callback: function (item) {
        if (this.num(item)) {
          console.log('及格' + item);
        }
      },
      num: function (item) {
        return item > 3;
      }
    }
    obj.arr.forEach(obj.callback, obj);

  结果：
![img](/img/in-post/arr-object-func/arr-foreach2.png)        
其中在callback函数中调用的this指向的是obj，foreach的第二个参数填入obj，改变了this指向。      

当数组元素为空，undefiend，null，false这些值的时候，只有空不会打印出来，但它还是数组的一位。    

    var arr = [1,2,,undefined,null,false];
    arr.forEach(function (item) {
      console.log(item, arr.length);
    })

  结果： 
![img](/img/in-post/arr-object-func/arr-foreach3.png) 
 `map()`:array.map(callback,[ thisObject]);map和forEach的用法相似。       

map来说，callback函数需要return，能够用变量接住处理的元素，将这些元素放到一个数组里面。     

    var arr = [1,2,3,4,5,6,7,8,9];
    var newArr = arr.map(function (item, index) {
      return item;
    })
    console.log('newArr', newArr);

    var forArr = arr.forEach(function (item) {
      return item;
    })
    console.log('forArr', forArr);

  结果：    
![img](/img/in-post/arr-object-func/arr-map1.png)      

 `filter()`：array.filter(callback,[thisObject]);用于筛选，过滤元素。用法和map相似。     
callback中如果元素进行判断返回true，则表示通过，false表示pass。     

看一下map和filter的区别？       

    var arr = [1,2,3,4,5,6,7,8,9];
    var mapArr = arr.map(function (item, index) {
      return item > 5;
    })
    console.log('mapArr', mapArr);

    var filArr = arr.filter(function (item) {
      return item > 5;
    })
    console.log('filArr', filArr);

结果：   
![img](/img/in-post/arr-object-func/arr-filter1.png)   

 `some()`：表示只要有一个callback函数返回true即可。      

    var arr = [1,2,3,4,5,6,7,8,9];
    var num = 5;
    arr.some(function (item) {
      if(item>num) {
        console.log('满足')
      }
    })

结果：   
![img](/img/in-post/arr-object-func/arr-some1.png) 

 `every()`:表示所有的元素是否都返回true,只要其中有一个元素不满足，就返回false；   

    var arr = [1,2,3,4,5,6];
    var flag = arr.every(function (item, index) {
      return item > 5;
    });
    console.log(flag);

结果为false，因为不是所有的元素都大于5；        

  `reduce()`: array.reduce(callback, initial);          
  callback中有四个参数：accumulator()，currentValue(当前数组的元素值)，currentIndex(表示正在处理的数组元素的索引)，array(表示当前的数组)

`indexOf(search, fromIndex)`:表示这个search元素，在这个数组中从fromIndex位置开始从前向后找到的第一个serach元素的索引，如果没有则返回-1。       
indexOf(serach)：表示从第0位开始寻找；fromIndex默认为0；       
indexOf(serach, '2'): 这时候会把字符串2转化为数字2，即从第2位开始寻找；       

`lastIndexOf(search, fromIndex)`:表示这个search元素，在这个数组中从fromIndex位置开始从后向前找到的第一个serach元素的索引，如果没有则返回-1。    
lastIndexOf(serach)：表示从第(arr.length - 1)位开始寻找；fromIndex默认为(arr.length - 1)；      
