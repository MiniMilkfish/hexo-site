---
title: script标签的defer 和 async
date: 2015/08/16
tags: [HTML]
excerpt: script标签在HTML文档中出现的时机决定着对应js脚本的加载时间。默认的，当script加载的时候页面是不会进行渲染的，也就是传统意义上的加载堵塞，这样很容易引起页面假死。所以我们经常将script标签放入body最底部。
---

在前端模块化、文件合并、打包概念等没有出现的年代，原始方式的脚本引入有很多地方值得玩味，就比如到底将script插入到哪才是最合适的。众所周知，script标签在HTML文档中出现的时机决定着对应js脚本的加载时间，默认的，当script加载的时候页面是不会进行渲染的，也就是传统意义上的加载堵塞，这样很容易引起页面假死。所以我们经常将script标签放入body最底部。

可见，脚本的出现、加载、执行时机对于页面加载影响很大。而script的defer和async就是与js脚本的加载、运行时机相关的两个属性。

下面详细分析这两个属性到底会对脚本的加载执行时机产生什么样的影响。

## 分析

**当浏览器碰到 script 脚本的时候：**
1. `<script src="script.js"></script>`
默认的，在没有 defer 或 async 属性的时候，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。
2. `<script async src="script.js"></script>`
有 async，加载和渲染后续文档元素的过程将和 script.js 的`加载与执行并行进行（异步）`。
3. `<script defer src="myscript.js"></script>`
有 defer，加载后续文档元素的过程将和 script.js 的`加载并行进行（异步）`，但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。

## 图示
![defer和async](http://ojd8i48oc.bkt.clouddn.com/blog-defer%E5%92%8Casync.jpg)
蓝色线代表网络读取，红色线代表执行时间，这俩都是针对脚本的；绿色线代表 HTML 解析。

## 总结

- defer 和 async 在网络读取（下载）这块儿是一样的，都是异步的（相较于 HTML 解析）
- 它俩的差别在于脚本下载完之后何时执行，显然 defer 是最接近我们对于应用脚本加载和执行的要求的。关于 defer，上图未尽之处在于它是按照加载顺序执行脚本的，这一点要善加利用
- async 则是一个乱序执行的主，反正对它来说脚本的加载和执行是紧紧挨着的，所以不管你声明的顺序如何，只要它加载完了就会立刻执行，而不像defer，是要在DomContentLoaded事件之后按照script标签的原有出现顺序执行。
