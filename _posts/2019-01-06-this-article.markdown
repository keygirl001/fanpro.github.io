---
layout:    post
title:     "有关this那点事"
subtitle:   " \"js基础知识\""
date:       2019/01/06
author:     "CaoFan"
header-img: "img/arr-object-func.png"
catalog: true
tags:
  - javascript

---

>"It's begining!" 

## 前言
关于this这个理解，越学越会有更多的理解，在这里想把自己学习到的，理解的写下来。

<p id="build"></p>
---

## 正文

this到底是什么呢？我理解的是：this是代表当前函数作用域的一个对象。(可能总结不到位，有瑕疵，后面会一一补充论证)

### 全局的this
全局的this对象即代表windows的全局作用域的环境。
1.在全局中使用；

    var a = 1;
    console.log(this.a); // 1


    我们可以看到这里的打印出来的是1，表示这个this指代的是全局windows；

2.在函数中使用；

    var a = 2;
    function test() {
      // 会有一个隐式的this
      console.log(this.a); // 2
    }

    在上面的这个test函数中，当它生成的时候就会有一个隐式的this，但并没有使用它，所以这里的this还是指向我们全局的this。（个人理解：涉及到作用域链的问题）


    var a = 1;
    var b = 2;
    function test() {
      this.a = 3;
      var b = 4;
      console.log(this.a); // 3
      console.log(this.b); // 2
    }

    上面提到我理解的作用域的问题：现在的作用域链上有两个作用域，从上到下是test函数作用域和全局作用域；首先我们在test作用域中找this对象下的a属性找到了，打印；接着找this下面的b属性，没找到就会往下面的作用域去找（全局作用域），找到了b打印（windows的变量会自动挂载到this下）；
