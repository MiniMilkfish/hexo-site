---
title: webpack devtool7种SourceMap
date: 2017/03/10
tags: [webpack]
republish: https://juejin.im/post/58293502a0bb9f005767ba2f
excerpt: 我们在使用 webpack 打包我们的工程模块时，经常会需要 devtool 开启 sourceMap 让我们可以调试代码，但是 webpack 文档中关于 devtool 给出了7种模式，文档也写得非常简略，初学者很难上手。本文将这7种模式的区别作详细介绍，希望能对你使用有帮助。
---

![](https://dn-mhke0kuv.qbox.me/1ed270d460fc749c87b5.png)

文章转载自掘金，作者@滴滴公共前端团队 - 水乙。[[webpack] devtool里的7种SourceMap模式是什么鬼？](https://juejin.im/post/58293502a0bb9f005767ba2f)

> 我们在使用 webpack 打包我们的工程模块时，经常会需要 devtool 开启 sourceMap 让我们可以调试代码，但是 webpack 文档中关于 devtool 给出了7种模式，文档也写得非常简略，初学者很难上手。本文将这7种模式的区别作详细介绍，希望能对你使用有帮助。

# 概念有何不同

我们先来看看文档对这7种模式的解释：

模式 | 解释
---|---
eval | 每个 module 会封装到 eval 里包裹起来执行，并且会在末尾追加注释 //@ sourceURL.
source-map | 生成一个 SourceMap 文件.
hidden-source-map | 和 source-map 一样，但不会在 bundle 末尾追加注释.
inline-source-map | 生成一个 DataUrl 形式的 SourceMap 文件.
eval-source-map	| 每个 module 会通过 eval() 来执行，并且生成一个 DataUrl 形式的 SourceMap .
cheap-source-map | 生成一个没有列信息（column-mappings）的 SourceMaps 文件，不包含 loader 的 sourcemap（譬如 babel 的 sourcemap）
cheap-module-source-map	| 生成一个没有列信息（column-mappings）的 SourceMaps 文件，同时 loader 的 sourcemap 也被简化为只包含对应行的。

> **注1**：webpack 不仅支持这 7 种，而且它们还是可以任意组合上面的 eval、inline、hidden 关键字，就如文档所说，你可以设置 souremap 选项为 cheap-module-inline-source-map。

> **注2：**如果你的 modules 里面已经包含了 SourceMaps ，你需要用 source-map-loader 来和合并生成一个新的 SourceMaps 。

# 使用结果有何不同

下面我们将列出这7种模式打包编译后的结果，从中看看他们的异同。

## eval

```javascript
webpackJsonp([1],[
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceURL=webpack:///./src/js/index.js?'
    )
  },
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceURL=webpack:///./src/static/css/app.less?./~/.npminstall/css-loader/0.23.1/css-loader!./~/.npminstall/postcss-loader/1.1.1/postcss-loader!./~/.npminstall/less-loader/2.2.3/less-loader'
    )
  },
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceURL=webpack:///./src/tmpl/appTemplate.tpl?"
    )
  },
...])
```

> 这样看很直观了，正如上文表格中的概念中写到，eval 模式会把每个 module 封装到 eval 里包裹起来执行，并且会在末尾追加注释。

## source-map

```javascript
webpackJsonp([1],[
  function(e,t,i){...},
  function(e,t,i){...},
  function(e,t,i){...},
  function(e,t,i){...},
  ...
])
//# sourceMappingURL=index.js.map
```

> 与此同时，你会发现你的 output 目录下多了一个 index.js.map 文件。

我们可以把这个 index.js.map 格式化一下，方便我们在下文的观察比较：

```javascript
{
  "version":3,
  "sources":[
    "webpack:///js/index.js",
    "webpack:///./src/js/index.js",
    "webpack:///./~/.npminstall/css-loader/0.23.1/css-loader/lib/css-base.js",
    ...
  ],
  "names":["webpackJsonp","module","exports"...],
  "mappings":"AAAAA,cAAc,IAER,SAASC...",
  "file":"js/index.js",
  "sourcesContent":[...],
  "sourceRoot":""
}
```

关于 sourceMap 行列信息如何映射源代码的详解，此处不是我们要重点讨论的话题，从略。感兴趣的同学可以参考阮一峰老师的科普文：[JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)

## hidden-source-map

```javascript
webpackJsonp([1],[
  function(e,t,i){...},
  function(e,t,i){...},
  function(e,t,i){...},
  function(e,t,i){...},
  ...
])
```
> 与 source-map 相比少了末尾的注释，但 output 目录下的 index.js.map 没有少.

## inline-source-map

```javascript
webpackJsonp([1],[
  function(e,t,i){...},
  function(e,t,i){...},
  function(e,t,i){...},
  function(e,t,i){...},
  ...
])
//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9...
```
> 可以看到末尾的注释 sourceMap 作为 DataURL 的形式被内嵌进了 bundle 中，由于 sourceMap 的所有信息都被加到了 bundle 中，整个 bundle 文件变得硕大无比。

## eval-source-map

```javascript
webpackJsonp([1],[
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceMappingURL=data:application/json;charset=utf-8;base64,...
    )
  },
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceMappingURL=data:application/json;charset=utf-8;base64,...
    )
  },
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceMappingURL=data:application/json;charset=utf-8;base64,...
    )
  },
  ...
]);
```

> 和 eval 类似，但是把注释里的 sourceMap 都转为了 DataURL 。

## cheap-source-map

> 和 source-map 生成结果差不多。output 目录下的 index.js 内容一样。
但是cheap-source-map生成的 index.js.map 的内容却比 source-map 生成的 index.js.map 要少很多代码，我们对比一下上文 source-map 生成的index.js.map 的结果，发现 source 属性里面少了列信息，只剩一个"webpack:///js/index.js"。

```javascript
// index.js.map
{
  "version":3,
  "file":"js/index.js",
  "sources":["webpack:///js/index.js"],
  "sourcesContent":[...],
  "mappings":"AAAA",
  "sourceRoot":""
}
```

## cheap-module-source-map

```javascript
// index.js.map
{
  "version":3,
  "file":"js/index.js",
  "sources":["webpack:///js/index.js"],
  "mappings":"AAAA",
  "sourceRoot":""
}
```
> 在 cheap-module-source-map 下 sourceMap 的内容更少了，sourceMap 的列信息减少了，可以看到 sourcesContent 也没有了。

# 这么多模式用哪个好？

开发环境推荐：

cheap-module-eval-source-map

生产环境推荐：

cheap-module-source-map （这也是下版本 webpack 使用-d命令启动 debug 模式时的默认选项）

原因如下：
1. **使用 cheap 模式可以大幅提高 souremap 生成的效率。**大部分情况我们调试并不关心列信息，而且就算 sourcemap 没有列，有些浏览器引擎（例如 v8） 也会给出列信息。
2.**使用 eval 方式可大幅提高持续构建效率。**参考官方文档提供的速度对比表格可以看到 eval 模式的编译速度很快。
3. **使用 module 可支持 babel 这种预编译工具**（在 webpack 里做为 loader 使用）。
4. **使用 eval-source-map 模式可以减少网络请求。**这种模式开启 DataUrl 本身包含完整 sourcemap 信息，并不需要像 sourceURL 那样，浏览器需要发送一个完整请求去获取 sourcemap 文件，这会略微提高点效率。而生产环境中则不宜用 eval，这样会让文件变得极大。

SourceMap 模式效率对比图：
![](https://dn-myg6wstv.qbox.me/a2e245898b08cdc389a2.jpeg)