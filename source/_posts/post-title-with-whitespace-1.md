---
title: vue源码学习-数据代理
date: 2019-06-13 16:59:55
tags: vue
---

一直都想找个时间研究一下vue源码，无奈一直没有时间。最近找工作，顺便把vue源码学习的过程给写下来。

<!-- more --> 
#### 什么是数据代理？
先来一个vue简单的例子，我们通常可以用vue实现以下的功能，即:可以使用app.name访问到app.$data的name属性,同时可以直接运用app.name = 'leo'改变app.$data中的name属性。

```js
var app = new Vue({
    data:{
        name:'joe'
    }
});
console.log(app.name) //joe
console.log(app.$data.name) //joe
app.name = 'leo'
console.log(app.name) //leo
console.log(app.$data.name) //leo
```

原理：Vue内部利用Object.defineProperty实现了数据代理，把$data中的属性直接暴露到实例属性中，这样的好处在于，我们在内部操作的时候可以方便属性访问，减少代码冗余，例如 this.$data.name = 'leo',我们可以简写成 this.name = 'leo',以下是内部实现的过程


```js
class myVue{
    constructor(opt){
        this.data = opt.data;
        //开始数据代理
        this.proxyData();
    }
    proxyData(){
        //遍历data中所有的key，根据key生成代理
        Object.keys(this.data).forEach(item => {
            Object.defineProperty(this,item,{
                get:()=>{
                    //当访问this.name时，返回data中的name
                    return this.data[item]
                },
                set:(value)=>{
                    //当设置this.name时，同时设置data中的name
                    this.data[item] = value;
                }
            })
        });
    }
}
var app = new myVue({
    data:{
        name:'joe',
    }
});
console.log(app.name) //joe
app.name = 'leo'
console.log(app.name) //leo
```


