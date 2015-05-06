---
layout: post
title: "javascript里~的神奇用法"
description: "javasceipt中~的用法"
category: 奇技淫巧
tags: javascript
---
这几天看koa源码的时候，经常看到`if(~notfound.indexOf(err.code)){ doSomeing... }`这种在一个表达式前面加~号的，今天就来扒一扒这已黑魔法。

##~取反操作符
不熟悉原码，反码，补码的小伙伴可以先看一下这篇文章[原码、反码、补码，计算机中负数的表示](http://blog.chinaunix.net/uid-26495963-id-3074603.html)  
在javascript中，假设有一个变量`var a = 1`, 那么`~a + a = -1`, 也就是说现在`~a ＝ -2`

##~在条件判断中的用法
先上一段代码

    var arr = ['zank', 'ywwhack']
    if(~arr.indexOf('zank')){
        console.log('found');
    }else{
        console.log('not found');
    }
这段代码最后会输出`found`，说明`~arr.indexOf('zank')`等价于`arr.indexOf('zank')>-1`  
![](/img/2.jpg)  
还记得刚才的`~a+a = -1`么，其实用的就是这个原理。`arr.indexOf()`调用后，如果没找到会返回`-1`，否则返回一个大于`-1`的整数。
假设`a = arr.indexOf()`，那么如果`arr`中存在所查找的元素时，`a=-1`，那么`~a=0`，上面代码可以改写如下：  

    var arr = ['zank', 'ywwhack'],
        a = arr.indexOf('zank'), //a = 0
        exist = ~a; // exist = -1, 
    //只有当a = -1时，即arr中不存在查找的元素,exist=0,其余的exist都为负值
    if(exist){
        console.log('found');
    }else{
        console.log('not found');
    }
    