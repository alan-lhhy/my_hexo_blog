---
title: vue源码学习-模板编译
date: 2019-06-13 16:59:55
tags: vue
---

接上一篇的数据代理，接下来实现一个简单的模板编译功能。..
<!-- more --> 
```html
<body>
    <ul id='app'>
        <li>{{name}}</li>
    </ul>
</body>
```
```js
var app = new Vue({
    el:'#app',
    data:{
        name:'joe'
    }
});
```

这里我们会用到一些比较少用的方法，其中有一个createDocumentFragment，操作文档碎片，这个方法和平时所用的dom操作的区别在于，在内存中操作dom节点，不会导致浏览器回流，实现较高性能的dom操作。可以查看[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createDocumentFragment)中的解释。


```html
<body>
    <ul id='app'>
        <li>{{name}}</li>
    </ul>
</body>
```
```js
        class myVue {
            constructor(opt){
                this.opt = opt;
                this.data = opt.data;
                this.el = document.querySelector(opt.el);
                //开始数据代理
                this.proxyData();
                //获取所有的Fragment对象
                this.fragment = this.getAllFragment();
                //初始化解析模板
                this.compile(this.fragment);
                //重新写入到dom中
                this.el.appendChild(this.fragment)
            }
            getAllFragment(){
                let fragmenChild;
                let fragmentList = document.createDocumentFragment();
                //取出fragmen
                while(fragmenChild = this.el.firstChild){
                    fragmentList.appendChild(fragmenChild);
                }
                return fragmentList;
            }
            compile(fragment){
                let reg = /^\{{2}(.*)\}{2}$/;
                [].slice.call(fragment.childNodes).forEach(item=>{
                    //nodeType为文本节点时，并且匹配到{{}}括号则执行编译，
                    if(item.nodeType == 3 && reg.test(item.textContent)){
                        //获取data中的属性名
                        let key = reg.exec(item.textContent)[1];
                        //取出data中的值赋值
                        if(this[key]) item.textContent = this[key];
                    }
                    //如果node节点还有子节点则递归解析编译模板
                    if(item.childNodes){
                        this.compile(item)
                    }
                });
            }
            proxyData(){
                //遍历data中所有的key，根据key生成代理
                Object.keys(this.data).forEach(item => {
                    Object.defineProperty(this, item, {
                        get: () => {
                            //当访问this.name时，返回data中的name
                            return this.data[item]
                        },
                        set: (value) => {
                            //当设置this.name时，同时设置data中的name
                            this.data[item] = value;
                        }
                    })
                });
            }
        }
        var app = new myVue({
            el:"#app",
            data: {
                name: 'joe',
            }
        });

```


