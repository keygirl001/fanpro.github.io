---
layout:    post
title:     "原始值和引用值的理解"
subtitle:   " \"js基础知识\""
date:       2019/01/23
author:     "CaoFan"
header-img: "img/arr-object-func.png"
catalog: true
tags:
  - javascript

---

>"It's begining!" 

## 前言
对于js的基础来说，值得类型是最基本的，也是必须要掌握的，对他们不理解的话，对于数据的处理就会有大麻烦，我就把自己所理解一些值类型中的点写下来，可能会有理解不当的地方，随着后期的学习，我会将他们更加完美、完善；

<p id="build"></p>
---

## 正文

首先，js的类型分为六种：number、string、boolean、object、undefined、null；       
其中object包括：数组和函数两种特殊的对象；       
js的又分为原始值和引用值：原始值（number、string、boolean、undefined、null）、引用值（array、obj）；
原始值不可改变、引用值是可变；        

### 原始值不可变，引用值可变
看一个例子：     
    
    var num = 10;
    num1 = num;
    num1 = 20;
    console.log(num, num1); // 10 20

我们可以看到虽然num把值赋给了num1，但是num1的改变跟num没有任何关系；     
所有的赋值都是定义一个新的变量，`它们指向的地址是不同的`，和之前的值没有任何的关系；    

再看一个例子：
    
    var arr = [1, 2, 3];
    arr1 = arr;
    arr1[0] = 2;
    console.log(arr, arr1);  // [2, 2, 3]  [2, 2, 3]

我们看到arr1的值发生变化了，arr的值也跟着发生变化了；       
对于数组来说它属于引用值，即它的赋值，`相当于将arr的引用地址和arr1的引用地址指向了同一个`，所以它们是息息相关的；    

结合着看一个例子：

    function f(a, b, c) {
      a = 20;
      b.value = 'changed';
      c = { value: 'changed' };
    }
    var num = 10;
    var obj1 = { value: 'unchanged' };
    var obj2 = { value: 'unchanged' };
    f(num, obj1, obj2);
    console.log(num, obj1, obj2); // ? ? ?

* 首先num是10，因为它是原始值，不会改变；    
* obj1呢，它等于{ value: 'changed' }，是因为它是一个对象，它的值是可变的，当obj1当作实参传进来的时候，形参中的b就相当于赋值给obj1，那么它们的引用是指向同一个地址，所以b的改变也会导致obj1发生变化；        
* obj2呢，它等于{ value: 'unchanged' }，是因为实际它内部执行过程是，先c = obj2；然后又将c = { value：changed }又重新指向了一个地址，所以obj2并不会发生改变；      

### 原始值和引用值的克隆
克隆就是给本体克隆一个副本，它们之间互相没有联系；
#### 原始值
对于原始值来说，它的克隆就是直接赋值给一个新的变量；
#### 引用值
1.浅克隆（就是只进行一层克隆）：      
`数组：`    
\*使用扩展运算符：*\
    
    var a = [1, 2, 3];
    var a1 = [...a1];
    a1[0] = 2;
    console.log(a, a1); // [1, 2, 3] [2, 2, 3]

\*利用数组的循环遍历：*\

    var arr = [1, 2, 3];
    var newArr = [];
    arr.forEach((item, index) => {
      newArr[index] = item;
    });
    arr[0] = 2;
    console.log(arr, newArr); // [2, 2, 3] [1, 2, 3]

当然除了forEach还可以用其他的数组的遍历的方法；     
主要是利用了将数组里的每一个原始值重新赋值，赋值的就和之前的每一个值都互不干扰；    

`对象：`     
\*使用扩展运算符：*\   

    var obj = {a: 1};
    var obj1 = {...obj};
    obj1.a = 2;
    console.log(obj, obj1); // {a: 1} {a: 2}

\*for..in遍历对象：*\   

    var obj = {a: 1};
    var obj1 = new Object();
    for(var props in obj) {
      obj1[props] = obj[props];
    }
    obj1.a = 2;
    console.log(obj, obj1); // {a: 1} {a: 2}

同数组原理一样；将对象的每一个原始值都重新赋值；