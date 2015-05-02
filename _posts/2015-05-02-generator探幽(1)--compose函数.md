---
layout: post
title:  "generator探幽(1)--compose函数"
date:   2015-05-02 17:21:26
categories: /
---
#本系列旨在通过对compose，co，koa等库源码的研究,进而理解generator在异步编程中的重大作用

##compose
[compose](http://https://github.com/koajs/compose)库只用了几十行代码，就实现了koa中间件的机制  
可以看我的compose.js，我在它的源码上做了些说明  
compose函数最关键的一段代码    

    while(i--){
        next = middleware[i].call(this, next);
    }
    yield *next;

这段代码把middleware中传进来的generator数组逆序取出并依次执行，每次执行的时候把上次generator执行返回的iterator当作参数传进去，所有generator执行完之后，调用第一个iterator  
因此，在generator中间件函数中，其实yield后面跟的是个iterator，即```yield* next```
下面用一个例子来说明这个过程  

    var middleware = [];
    var arr = []; 
    middleware.push(function* m1(next){
        arr.push(1);
        yield* next; //位置1
        arr.push(3);
    });
    middleware.push(function* m2(next){
        arr.push(1);
        arr.push(2);
    });
    var gen = compose(middleware); //将middleware中的两个generator'组合'成一个新的generator函数
    var it = gen();
    it.next(); //执行后会停在m1函数的位置1处
    it.next(); //执行后从m1函数暂停处进入m2函数，执行完整个m2函数后再次回到m1函数继续执行剩下的部分
    console.log(arr); //[1,2,3,4]
