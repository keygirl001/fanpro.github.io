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
全局的this对象即指代全局作用域window。   
1.在全局中使用；

    var a = 1;
    console.log(this.a); // 1


我们可以看到这里的打印出来的是1，表示这个this指代的是全局window；    

2.在函数中使用；  

    var a = 2;
    function test() {
      console.log(this.a); // 2
    }


在上面的这个test函数中，当它生成的时候就会有一个隐式的this，但并没有使用它，
所以这里的this还是指向我们全局的this。（个人理解：涉及到作用域链的问题）    


    var a = 1;
    var b = 2;
    function test() {
      this.a = 3;
      var b = 4;
      console.log(this.a); // 3
      console.log(this.b); // 2
    }


上面提到我理解的作用域的问题：现在的作用域链上有两个作用域，从上到下是test函数作用域和全局作用域；
首先我们在test作用域中找this对象下的a属性找到了，打印；
接着找this下面的b属性，没找到就会往下面的作用域去找（全局作用域），找到了b打印（window的变量会自动挂载到this下）；        

3.setTimeout和setInterval中的this永远指向window；因为setTimeout和setInterval是挂载在window下面的；


    var x = 1;
    var obj = {
      x: 2,
      fn: setTimeout(function () {
        console.log(this.x);
      }, 1000)
    }
    obj.fn(); // 打印1


### 构造函数相关的this    
构造函数的原理：会隐式的生成this对象，最后返回this对象；    


    var name = 'cao';
    function Person(name, age) {
      // var this = this;
      this.name = name;
      this.age = age;
      // return this;
    }
    const op1 = new Person('fan', 18); // {name: 'fan', age: 18}
    console.log(op1.name) // 'fan'


通过Person构造函数生成的对象op1，这时this指向op1；   


### 对象函数中的this指向
一般是谁调用的它，this就指向谁；


    var obj = {
      x: 1,
      fn: function () {
        console.log(this.x);
      }
    }
    obj.fn(); // 1


这里我们可以看到是obj调用的fn，那么fn里的this就指向了obj，所以打印1；我们再改进一下：    


    var x = 1;
    var obj = {
      x：2，
      fn: function () {
        console.log(this.x);
      }
    }
    obj.fn(); // 很明显，这里打印2，因为是obj调用的fn
    var fn1 = obj.fn;
    fn1(); // 打印？


打印的应该是1，因为我们将fn的引用拿出来了赋值给fn1，其实就是在全局执行fn1()，所以这个this就指向全局的this；     


### 使用call，apply和bind改变this指向
call，apply和bind的第一个参数都是去指定当前的this指向；


    var x = 3;
    var o1 = {
      x: 1,
      fn: function () {
        console.log(this.x);
      }
    }
    var o2 = {
      x: 2,
      fn: function () {
        console.log(this.x);
      }
    }
    o2.fn.call(o1); // 1
    o2.fn(); // 2
    o2.fn.call(); // 3
    o2.fn.call({x: 4}); // 4


我们可以看到当call的第一个参数不填或者填写null/undefined的时候都是指向window；     
额外讲一嘴，既然提到了call，apply和bind，我就稍微回忆一下这三个的区别（他们的共同点就是第一个参数）：    
call(): 第二个参数以及后面的n个参数，可以作为函数的参数传递，以，逗号的形式隔开；         
apply(): 第二个参数也可作为函数的参数传递，但是以数组的形式，[a,b,c];       
bind(): bind()执行之后返回的是当前的函数；接着上面的那个例子；   


    o2.fn.bind(o1)(); // 1
    console.log(o2.fn.bind()); // 返回fn函数


### 箭头函数中的this
箭头函数的this指向的是定义时所在的this对象，而不是执行时所在的this对象；    
来看几个例子：   


    var x = 1;
    var obj = {
      x: 2,
      fn1: () => {
        console.log(this.x);
      },
      fn2: function () {
        console.log(this.x);
      }
    }
    obj.fn1(); // 打印？
    obj.fn2(); // 2


fn2执行打印2很明显，是因为obj调用的它，所以this指向obj；
fn1的执行打印1，因为首先在obj中定义的fn1，其次箭头函数本身没有this，依赖外层代码块的this，对于obj来说它的this是指向window的，所以fn1里的this也指向window，打印1；


    var a = 1;
    function Test() {
      this.a = 2;
      this.fn1 = () => {
        console.log(this.a);
      };
      this.fn2 = function () {
        console.log(this.a);
      }
    }
    var test = new Test();
    test.fn1();
    test.fn2(); // 2


fn1执行打印的也是2，首先fn1的定义是在Test()执行的时候，这时候呢fn1的this指向了Test，所以this.a就打印2；    


    function Test() {
      setTimeout(() => {
        console.log('箭头函数', this.a); // ?
      });
      setTimeout(function() {
        console.log('普通函数', this.a); // 1
      })
    }
    var a = 1;
    Test.call({a:2});


你会发现箭头函数打印的是2，而普通函数打印的是1；对于setTimeout来说它的this始终指向widnow的，但是箭头函数的this指向他定义的时候的外层代码块的this，即Test；    


     var obj = {
      x : 1,
      fn1: function () {
        document.addEventListener('click', (e) => {
          this.fn2(e.type);
        })
        document.addEventListener('click',function (e) {
          this.fn2(e.type);
        })
      },
      fn2: function (type) {
        console.log('这里的', type, this.x);
      }
    }
    obj.fn1();


这里借鉴阮老师的一个例子，对于第一个箭头函数的使用，会正确console，因为箭头函数的this指代的是fn1所指向的this，fn1是由obj调用的所以this指向obj；
而对于第二个普通函数使用来说，this指向的是document，所以会报错，找不到fn2；  


### class类的this
对于class类来说，它的this默认指向类的实例；


    class App {
      fn1() {
        this.fn2();
      }
      fn2() {
        console.log('这里');
      }
    }
    const app = new App();
    const { fn1 } = app;
    fn1(); // 报错，找不到fn2


这里把fn1拿出来执行，就会找不到this，this指向当前执行的环境；解决方法：


    (1) bind：在构造函数中绑定当前this；
    class App {
      constructor() {
        this.fn1 = this.fn1.bind(this);
      }
      // ...
    }

    (2) 箭头函数
    class App {
      constructor() {
        this.fn1 = () => {
          this.fn2();
        }
      }
      // ...
    }


对于react中的一些事件的方法，比如click方法，也要使用bind绑定当前this；否则jsx传递onCick方法时会当作一个第三者环境的变量，将this丢失；


    import React from 'react';
    export default class App extends React.Component {
      constructor(props) {
        super(props);
        // this.onhandleClick = this.onhandleClick.bind(this);  // bind(this)
      }
      onhandleClick = () => { // 或者箭头函数
        console.log(1111);
      }
      render() {
        <div>
          <button onClick={this.onhandleClick}/>
        </div>
      }
    }

## 小结
通过这些例子，总结几点：
对于普通函数来说，this总指向window；但是谁调用它，this就指向它；call，apply，bind也会改变this指向；不要忘了setTimeout和setInterval的this永远指向window（使用箭头函数就要仔细考虑了）；
对于构造函数来说，this总指向它构造的对象；
箭头函数，this固定化，指向它定义时所在的this指向（this没有自己的this，所以通常由它外层代码块决定）；
对于class类中的方法，最好都是用bind或者箭头函数，显示绑定实列的this；
