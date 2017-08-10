---
title: webpackBootstrap 分析
date: 2017/03/23
tags: [webpack]
excerpt: 以webpack github库中的多入口example看下webpack构建出来的代码结构，以及chunk和bundle是什么等一些基础webpack知识
---

# 前言

乍一看，bootstrap这个名字感觉好高大上有没有？

webpackBootstrap 从字面意思理解即为webpack的引导程序，就像计算机启动一样，引导程序是所有模块加载的充分条件。所以引导程序必然是webpack打包构建资源的首个加载项的首个执行程序。什么意思？具体到代码我们知道，webpack每次构建都会先生成bootstrap，这个bootstrap包含了`webpackJsonpCallback` 和 `__webpack_require__` 等一系列函数去管理整个webpack的模块加载的事情。

借用bootstrap里的一句注释解释就是`This file contains only the entry chunk`，也就是说bootstrap是一个负责引导模块加载等功能的chunk而已，他应该是整个应用的入口。

# 多入口例子

下面结合webpack github 的 example [multiple-entry-points](https://github.com/webpack/webpack/tree/webpack-1/examples/multiple-entry-points) 及由其打包出来的代码详细理一遍。

官方给出的这个例子展示了一个带有commons chunk的多入口配置。

例子中有两个页面`pageA`和`pageB`，针对这两个页面我们有如下诉求：
- 为每个页面单独打包出自己的bundle
- 创建一个在每个页面都有使用的公共模块`shared`bundle(这里类比实际开发中的公共模块，一般体量都会很大但是不会轻易更改)。
- 使用代码分割以便实现按需加载。

其实这个例子本身是为了用来说明多入口配置的，通过这个例子我们也可以很好的学习到如何配置多入口和通过 CommonsChunkPlugin 插件提取公共chunk。

## 资源文件

**1. pageA.html**:

```HTML
<html>
	<head></head>
	<body>
		<script src="js/commons.js" charset="utf-8"></script>
		<script src="js/pageA.bundle.js" charset="utf-8"></script>
	</body>
</html>
```

**2. pageB.html**:

```HTML
<html>
    <head></head>
    <body>
        <script src="js/commons.js" charset="utf-8"></script>
        <script src="js/pageB.bundle.js" charset="utf-8"></script>
    </body>
</html>
```

**3. common.js**:

```JS
// Other code
···
···
···
module.exports = "Common";
```

**4. shared.js**:

```JS
module.exports = function(msg) {
    console.log(msg);
};
```

**5. pageA.js**:

```JS
var common = require('./common');
require(["./shared"], function(shared) {
	shared("This is page A");
});
// Other code
···
···
···
```

**6. pageB.js**:

```JS
var common = require("./common");
require.ensure(["./shared"], function(require) {
	var shared = require("./shared");
	shared("This is page B");
});
// Other code
···
···
···
```

**7. webpack.config.js**:

```JS
var path = require("path");
var CommonsChunkPlugin = require("../../lib/optimize/CommonsChunkPlugin");
module.exports = {
	entry: {
		pageA: "./pageA",
		pageB: "./pageB"
	},
	output: {
		path: path.join(__dirname, "js"),
		filename: "[name].bundle.js",
		chunkFilename: "[id].chunk.js"
	},
	plugins: [
		new CommonsChunkPlugin("commons") // also use webpack.optimize object
	]
}
```

以上便是我们要分析的原始案例情景。

- 两个页面需要独自打包构建，目的是模块拆分和减小资源体量。
- 两个页面共同引用common公共模块
- 两个页面共同以require的形式引用shared模块

现在我们要通过配置多入口达到文章刚开始设定的目标。

## 构建结果

webpack成功构建出如下四个文件：
- `0.chunk.js`
- `commons.bundle.js`
- `pageA.bundle.js`
- `pageB.bundle.js`

**commons.bundle.js**

```js
(function (modules) { // webpackBootstrap
    // install a JSONP callback for chunk loading
    var parentJsonpFunction = window["webpackJsonp"];
    window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
        // add "moreModules" to the modules object,
        // then flag all "chunkIds" as loaded and fire callback
        var moduleId, chunkId, i = 0, resolves = [], result;
        for (; i < chunkIds.length; i++) {
            chunkId = chunkIds[i];
            if (installedChunks[chunkId])
                resolves.push(installedChunks[chunkId][0]);
            installedChunks[chunkId] = 0;
        }
        for (moduleId in moreModules) {
            if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
                modules[moduleId] = moreModules[moduleId];
            }
        }
        if (parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
        while (resolves.length)
            resolves.shift()();
        if (executeModules) {
            for (i = 0; i < executeModules.length; i++) {
                result = __webpack_require__(__webpack_require__.s = executeModules[i]);
            }
        }
        return result;
    };

    // The module cache
    var installedModules = {};

    // objects to store loaded and loading chunks
    var installedChunks = {
        3: 0
    };

    // The require function
    function __webpack_require__(moduleId) {

        // Check if module is in cache
        if (installedModules[moduleId])
            return installedModules[moduleId].exports;

        // Create a new module (and put it into the cache)
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };

        // Execute the module function
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

        // Flag the module as loaded
        module.l = true;

        // Return the exports of the module
        return module.exports;
    }

    // This file contains only the entry chunk.
    // The chunk loading function for additional chunks
    __webpack_require__.e = function requireEnsure(chunkId) {
        if (installedChunks[chunkId] === 0)
            return Promise.resolve();

        // a Promise means "currently loading".
        if (installedChunks[chunkId]) {
            return installedChunks[chunkId][2];
        }

        // setup Promise in chunk cache
        var promise = new Promise(function (resolve, reject) {
            installedChunks[chunkId] = [resolve, reject];
        });
        installedChunks[chunkId][2] = promise;

        // start chunk loading
        var head = document.getElementsByTagName('head')[0];
        var script = document.createElement('script');
        script.type = 'text/javascript';
        script.charset = 'utf-8';
        script.async = true;
        script.timeout = 120000;

        if (__webpack_require__.nc) {
            script.setAttribute("nonce", __webpack_require__.nc);
        }
        script.src = __webpack_require__.p + "" + chunkId + ".chunk.js";
        var timeout = setTimeout(onScriptComplete, 120000);
        script.onerror = script.onload = onScriptComplete;
        function onScriptComplete() {
            // avoid mem leaks in IE.
            script.onerror = script.onload = null;
            clearTimeout(timeout);
            var chunk = installedChunks[chunkId];
            if (chunk !== 0) {
                if (chunk) chunk[1](new Error('Loading chunk ' + chunkId + ' failed.'));
                installedChunks[chunkId] = undefined;
            }
        };
        head.appendChild(script);

        return promise;
    };

    // expose the modules object (__webpack_modules__)
    __webpack_require__.m = modules;

    // expose the module cache
    __webpack_require__.c = installedModules;

    // identity function for calling harmony imports with the correct context
    __webpack_require__.i = function (value) {
        return value;
    };

    // define getter function for harmony exports
    __webpack_require__.d = function (exports, name, getter) {
        if (!__webpack_require__.o(exports, name)) {
            Object.defineProperty(exports, name, {
                configurable: false,
                enumerable: true,
                get: getter
            });
        }
    };

    // getDefaultExport function for compatibility with non-harmony modules
    __webpack_require__.n = function (module) {
        var getter = module && module.__esModule ?
            function getDefault() {
                return module['default'];
            } :
            function getModuleExports() {
                return module;
            };
        __webpack_require__.d(getter, 'a', getter);
        return getter;
    };

    // Object.prototype.hasOwnProperty.call
    __webpack_require__.o = function (object, property) {
        return Object.prototype.hasOwnProperty.call(object, property);
    };

    // __webpack_public_path__
    __webpack_require__.p = "";

    // on error function for async loading
    __webpack_require__.oe = function (err) {
        console.error(err);
        throw err;
    };
})

([
    /* 0 */,
    /* 1 */
    /***/ (function (module, exports) {

        module.exports = "Common";

    })
]);
```

**0.chunk.js**

```js
webpackJsonp([0],[
/* 0 */
/***/ (function(module, exports) {

module.exports = function(msg) {
    console.log(msg);
};

/***/ })
]);
```

**pageA.bundle.js**

```js
webpackJsonp([2],{

/***/ 2:
/***/ (function(module, exports, __webpack_require__) {

var common = __webpack_require__(1);
__webpack_require__.e/* require */(0).then(function() { var __WEBPACK_AMD_REQUIRE_ARRAY__ = [__webpack_require__(0)]; (function (shared) {
    shared("This is page A");
}.apply(null, __WEBPACK_AMD_REQUIRE_ARRAY__));}).catch(__webpack_require__.oe);

/***/ })

},[2]);
```

**pageB.bundle.js**

```js
webpackJsonp([1], {
    3: (function (module, exports, __webpack_require__) {
        var common = __webpack_require__(1);
        __webpack_require__.e/* require.ensure */(0/* duplicate */).then((function (require) {
            var shared = __webpack_require__(0);
            shared("This is page B");
        }).bind(null, __webpack_require__)).catch(__webpack_require__.oe);

    })
}, [3]);
```


## 结果分析

Code Splitting 是Webpack的核心功能，其主要作用为：

- 管理加载顺序
- 合并相同代码
- 模块管理
- 按需加载

没错，上述构建结果中展示的commons.bundle.js大部分的内容都是为了实现这些功能，也就是我们所要讨论的bootstrap的内容，所以common.bundle.js必须是入口文件，因为他负责全部的模块加载工作。其实源码中就有很详细的注释了，英文过的去的话大体上过一遍就应该能知道webpack的构建之后的文件加载和执行的基本流程。下面从commons中与挑几个bootstrap相关的比较重要的点总结。

**webpackJsonp**

```js
var parentJsonpFunction = window["webpackJsonp"];
window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {}
```

webpackJsonp 也就是具名的 webpackJsonpCallback 函数。我们找到其他的chunk或者bundle可以发现，其实经过webpack构建完之后的每个bundle或者chunk都是执行了一个webpackJsonp函数而已。

那这个webpackJsonp 函数到底做了什么？

- add "moreModules" to the modules object
- then flag all "chunkIds" as loaded and fire callback

没错，就是这两句注释。当加载到某个bundle或者chunk的时候，一旦代码开始执行，webpackJsonp函数的功能就是将当前模块添加到总模块对象中，并且标记当前的chunk为已加载状态，最后执行回调。当然webpack在这期间调用了__webpack_require__(moduleId)来完成这个功能。

> 由源码可以看出，chunk和bundle的区别是chunk不会主动执行而bundle会主动执行。chunk是异步加载或者按需加载的功能模块，webpack可以将其单独打包出来，它的 webpackJsonp 函数没有 executeModules ，也就是说它只会被加载，而没有可以执行的部分。而bundle则会生成 executeModules ，也就是说他会执行。由这个例子可以知道，commons中的bootstrap部分和0.chunk都属于chunk而其他则视为bundle更合适。


**__webpack_require__(moduleId)**

该函数完成模块加载的功能。首先会检查待加载模块是否已在缓存中，如果存在则直接返回对应模块的export导出；如果不在缓存中，则新生成一个模块并将其加入模块总对象中并执行模块函数，然后置为loaded状态并返回该模块的export导出。

**__webpack_require__.e**

这个函数的第一句注释成功的引起了我的注意~。。。 `This file contains only the entry chunk.` 这句话什么意思，其实又印证了上边chunk和bundle区别的结论，就是说webpackBootStrap其实就是一个chunk，而且是个入口级别的。

言归正传再说回 __webpack_require__.e 。其实这个函数更简单了，就是实现按需加载的功能。也就是在代码中如果有异步require的情况，webpack就会处理成jsonp的形式异步去获取对应的chunk。

**__webpack_require__.m**

将modules这个总的模块暴露出去。

**__webpack_require__.c**

将cache暴露出去。

**__webpack_require__.i**

注释的意思是处理错误的import模块引入，但是他只是返回了原有值而已，怎么就处理了？下面紧接着的 __webpack_require__.n 和 __webpack_require__.d 也是为了处理兼容的模块引入。

**__webpack_require__.p**

这个就厉害了。记得chunk的异步jsonp加载吗？其中script标签的src就是由这个path和chunkId拼出来的。这个是配置文件中output里的publicPath属性值。

**__webpack_require__.oe**

异步加载的抛错。


> 总结来看：每个webpack构建流程都会生成一个专门的bootstrap，bootstrap的作用是为了管理所有与模块加载相关的事情，他是webpack打包代码执行的基础，所以webpackBootStrap是入口chunk，自动执行。