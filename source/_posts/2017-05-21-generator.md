---
title: Generator 函数的异步应用
date: 2017/05/20
tags: [Generator, ES6]
excerpt:  上一篇整理了应用Promise实现链式调用避免回调函数嵌套的内容，回调函数本身并没有问题，它的问题出现在多个回调函数嵌套。但是Promise也并非十全十美，then的链式调用将代码的语义化降低，一长串的then使得代码逻辑不是很直观。
---

上一篇整理了应用Promise实现链式调用避免回调函数嵌套的内容，回调函数本身并没有问题，它的问题出现在多个回调函数嵌套。但是Promise也并非十全十美，then的链式调用将代码的语义化降低，一长串的then使得代码逻辑不是很直观。

这一篇就接着异步应用的话题整理一下`Generator`函数的内容。

# 回调嵌套和Promise

**异步操作的回调嵌套写法：**

```js
var fs = require('fs');

fs.readFile('/usr/lib/public/index1.js', 'utf-8', function(error, data) {
  fs.readFile('/usr/lib/public/index2.js', 'utf-8', function(error, data) {
    fs.readFile('/usr/lib/public/index3.js', 'utf-8', function(error, data) {
      // ...
    });
  });
});
```

**异步操作的Promise链式调用写法：**

```js
var readFile = require('fs-readfile-promise');

readFile('/usr/lib/public/index1.js').then(function(data1) {
  console.log(data1);
  return readFile('/usr/lib/public/index3.js');
}).then(function(data2) {
  console.log(data2);
  return readFile('/usr/lib/public/index3.js');
}).then(function(data3) {
  console.log(data3);
  return readFile('/usr/lib/public/index3.js');
});
```

`fs-readfile-promise`模块会将`fs`模块的readFile方法包装为Promise构造形式，`readFile('/usr/lib/public/index1.js')`会执行原本的读取文件操作，并封装了resolve和reject，本身返回的是Promise实例，所以可以调用then方法。

结合上一节Promise的内容，上面两种写法很好理解。

# Generator 函数

本身Generator函数的设计初衷并不是针对解决异步问题的，它的原意是通过将程序分段执行从而实现程序执行的不同时期会有不同的状态(个人理解，可能表述上欠妥当)，可以简单理解为一个状态机。但是我们通过他的这些特性，正好可以设计用来解决异步问题。

```js
function *gene1() {
  console.log('gene1 start...');
  yield gene2();
  console.log('gene2 end...');
  yield gene3();
  console.log('gene3 end...');
  console.log('gene1 end...');
}

function gene2() {
  console.log('gene2 start...');
}

function gene3() {
  console.log('gene3 start...');
}

var gen = gene1();
gen.next(); {value:undefined,done:false}
gen.next(); {value:undefined,done:false}
gen.next(); {value:undefined,done:true}
```

**代码解读**

如上的代码，第一步执行gene1()返回遍历器对象，第二步调用遍历器对象的next()依次从函数头部开始执行，直到遇到第一个yield表达式，此时的执行结果为`gene1 start...和gene2 start...`，第三步再次调用next()便从上次结束的位置依次往下执行，直到碰到第二个yield表达式，此时的执行结果为`gene2 start...和gene3 start...`，第四步再次调用next()后，代码执行到结束位置，因此整个流程结束，对应遍历器对象的返回值对象的done为false，此时的执行结果为`gene3 start...和gene3 start...`，

**Generator函数说明**

Generator 函数是一个普通函数，但是有如下特征：

- function关键字与函数名之间有一个星号
- 函数体内部使用yield表达式，定义不同的内部状态

Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象。

下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。换言之，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。


为了更好的理解Generator，引入一个新的概念-协程。协程的意思是多个线程互相协作，完成异步任务

- 第一步，协程A开始执行。
- 第二步，协程A执行到一半，进入暂停，执行权转移到协程B。
- 第三步，（一段时间后）协程B交还执行权。
- 第四步，协程A恢复执行。
- 第五步，重复上述操作，直到执行到结束位置或者显示的return。

协程的概念和Generator的理念不谋而合。

# Generator的协程

Generator 函数不同于普通函数的一个地方，即执行它不会返回结果，返回的是指针对象。调用指针g的next方法，会移动内部指针，指向每一次遇到的yield语句。也就是说Generator函数内部的每一个yield语句都需要对应的指针对象的next方法调用，一次next只能执行一套yield语句。

换言之，next方法的作用是分阶段执行Generator函数。每次调用next方法，会返回一个对象，表示当前阶段的信息（value属性和done属性）。value属性是yield语句后面表达式的值，表示当前阶段的值，所以将yield表达式赋值给其他变量是无效的；done属性是一个布尔值，表示 Generator 函数是否执行完毕，即是否还有下一个阶段。

# Generator的数据交换和错误处理

Generator 函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。除此之外，它还有两个特性，使它可以作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制。

#### 数据交换

遍历器对象的next返回值是一个对象，对象包括value和done两个字段。其中done标识当前指针是否移动到末尾，而value则是yield后跟随的表达式的返回值，这就构成了从Generatord的分段yield表达式向next传参的路径。

同时，next在调用的时候也可以显示传参，传入的参数作为上一次的yield表达式的结果值。我们知道在上面有说过，yield后的表达式的执行结果作为next的结果值，而整个的yield表达式是不会返回值的。但是当在调用next的时候进行显示传参，则可以将这个参数赋值给上一次的整个的yield表达式。

```js
function *gene1() {
  var yieldVal1 = yield gene2();
  console.log(yieldVal1);
}

function gene2() {
  return 1;
}

var gen = gene1();
gen.next(); // {value:1,done:false}
gen.next(2); // {value:undefined,done:true}
```

如上代码所示：第一次next调用的时候，执行gene2()，返回值为1，这个值作为指针对象的value属性的值，这是从Generator内部yield表达式向外部传递数据。

第二次调用next的时候，我们显示传入了参数2，这个2参数被当作`yield gene2()`整个表达式的返回值。要知道，如果不传参调用next的话，这个表达式是无值的。这是从外部向Generator内部yield表达式传递数据。

#### 错误处理

Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。

```js
function* gen(x) {
  try {
    var y = yield x + 2;
  } catch (e) {
    console.log('1:' + e);
  }
  try {
    var z = yield x + 2;
  } catch (e) {
    console.log('2:' + e);
  }
}

var g = gen(1);
g.next();
g.throw('出错了'); // 对应于 var y = yield x + 2;
g.next();
g.throw('又出错了'); // var z = yield x + 2;
```

注意g.throw的抛错位置很关键，因为他依赖于g这个指针当前的位置。比如上面的代码中，Generator中有两个yield表达式，对应于两个next调用，同时不同的yield表达式对应于不同的异常捕获，因此不同位置的throw对应不同的异常捕获代码。

# Generator函数封装异步任务

其实通过上面错误处理的示例便可以看出，Generator很好，但唯一比较烦的就是这个next的调用。假如我们定义了一个具有100个yield表达式的Generator函数，那就要手动执行100+1次next。这期间还要穿插各种异常处理的逻辑。就定位不同的位置都能把人搞蒙。

因此，要想通过Generator实现异步任务，首要的任务就是实现自动执行next。

#### Thunk函数

Thunk 函数是关于求值策略中的`传名调用`的一种实现策略，用函数来替换函数参数中的表达式。将函数表达式参数转换为一个临时的求值函数传入，这样传入的就是一个函数引用，从而不需要再对表达式进行先求值，当函数执行的时候，哪里用到这个参数，再执行这个Thumk函数即可。具体的请查看[Thunk函数](http://es6.ruanyifeng.com/#docs/generator-async#Thunk-函数)




