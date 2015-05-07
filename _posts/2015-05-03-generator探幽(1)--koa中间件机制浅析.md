---
layout: post
title:  "generator探幽(1)--koa中间件机制浅析"
category: generator
tags: javascript
---
###本系列旨在通过对co，koa等库源码的研究,进而理解generator在异步编程中的重大作用(ps:所有代码请在node --harmony或者iojs环境中运行)

##koa中间件的形式
相信用过[koa](https://github.com/koajs/koa)的小伙伴一定很熟悉下面这段代码  

    var app = require('koa')(),
        router = require('koa-router')();
    
    app.use(function *(next){
        console.log(1);
        yield next;
        console.log(5);
    });
    app.use(function *(next){
        console.log(2);
        yield next;
        console.log(4);
    });
    app.use(function *(){
        console.log(3);
    });
    
    app.listen(3000);

当一个请求到达的时候，控制台会依次输出`1 2 3 4 5`，这就是koa中强大的middleware特性，撇开koa本身，我们来扯扯middleware这东西是怎么实现的

##yield *iterator
不熟悉`generator`和`iterator`的小伙伴可以先看下阮一峰先生编写的《ES6入门》的[generator函数](http://es6.ruanyifeng.com/#docs/generator)这一节  
假设你已经熟悉了generator和iterator,现在我们来看一段代码

    function *m1(){
        console.log(1);
        yield *m2Iterator;
        console.log(3);
    }
    
    function *m2(){
        console.log(2);
    }
    var m1Iterator = m1(),
        m2Iterator = m2();
    
    m1Iterator.next();//
    
上面代码运行后，控制台会依次输出`1 2 3`，这是因为在`m1`函数内部，`yield`后面跟的是一个`iterator`。
`m1Iterator.next()`调用后，`m1`函数开始执行。
当`m1`运行到`yield *m2Iterator`时，不会暂停，而是直接跳进`m2`函数执行`m2`函数内的代码。
由于`m2`函数中没有`yield`，因此会一直执行完`m2`函数中的代码，并返回至`m1`函数中执行`yield *m2Iterator`后面的代码。

##middleware中间件的雏形
理解了上面的过程后，我们接着看下面这段代码

    function *m1(){
        console.log(1);
        yield *m2();
        console.log(5);
    }
    
    function *m2(){
        console.log(2);
        yield *m3();
        console.log(4);
    }
    
    function *m3(){
        console.log(3);
    }
    
    m1().next(); //控制台依次输出1 2 3 4 5
    
是不是形式跟我们一开始看到的`koa`的中间件有点接近了？

##compose
[compose](http://https://github.com/koajs/compose)是`koa`里用来实现中间件机制的模块  
`compose`函数最关键的一段代码 

    while(i--){
        next = middleware[i].call(this, next);
    }
    yield *next;
    
这段代码把`middleware`中传进来的`generator`数组逆序取出并依次执行，每次执行的时候把上次`generator`执行返回的`iterator`当作参数传进去，所有`generator`执行完之后，调用第一个`iterator`  
因此，在`generator`中间件函数中，其实`yield`后面跟的是个`iterator`，即`yield *next`  
下面用一个例子来说明这个过程  

    function *m1(next){
        console.log(1);
        yield *next;
        console.log(5);
    }
    
    function *m2(next){
        console.log(2);
        yield *next;
        console.log(4);
    }
    
    function *m3(){
        console.log(3);
    }
    
    var gen = compose([m1, m2, m3]),
        it = gen();
    //compose([m1, m2, m3])()可以想象成它做了这么一些事(伪代码)：
    //1 var it3 = m3();
    //2 var it2 = m2(it3);
    //3 var it1 = m1(it2);
    //4 gen = function(){
    //          yield *it1;  
    //        }
    
    it.next(); //控制台依次输出1 2 3 4 5
    
现在就很像`koa`的中间件的写法了。唯一的差别是`koa`中使用的是`yeild next`而我们这里用的是`yield *next`
其实在`koa`中`yeild next`和`yeild *next`效果是等价的，这主要得益于[co](https://github.com/tj/co)库，我们以后再来好好扯扯这小玩意～
