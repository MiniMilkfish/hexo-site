---
title: JavaScript 的相等性判断
date: 2015/11/03
tags: [javascript]
excerpt: 简单地说，两等号判等会在比较时进行类型转换；三等号判等不会进行类型转换（如果类型不同会直接返回 false ）； Object.is 在三等号判等的基础上特别处理了 NaN 、 -0 和 +0 ，保证 -0 和 +0 不再相同，但 Object.is(NaN, NaN) 会返回 true。（像其他数值一样比较 NaN ——由于 IEEE 754 的规范，无论使用双等号或三等号，比较 NaN 都会得到 false ）但请注意，此外，这三个运算符的原语中，没有一个会比较两个变量是否结构上概念类似。对于任意两个不同的非原始对象，即便他们有相同的结构， 以上三个运算符都会计算得到 false 。
---

## 操作符

JavaScript 提供三种不同的比较操作符：

- 严格相等，使用 ===
- （非严格）相等，使用 ==
- 以及 Object.is （ECMAScript 6 新特性）

简单地说，两等号判等会在比较时进行类型转换；三等号判等不会进行类型转换（如果类型不同会直接返回 false ）； Object.is 在三等号判等的基础上特别处理了 NaN 、 -0 和 +0 ，保证 -0 和 +0 不再相同，但 Object.is(NaN, NaN) 会返回 true。（像其他数值一样比较 NaN ——由于 IEEE 754 的规范，无论使用双等号或三等号，比较 NaN 都会得到 false ）但请注意，此外，这三个运算符的原语中，没有一个会比较两个变量是否结构上概念类似。对于任意两个不同的非原始对象，即便他们有相同的结构， 以上三个运算符都会计算得到 false 。

## 严格相等 ===

 在日常中使用全等操作符几乎总是正确的选择。对于除了数值之外的值，全等操作符使用明确的语义进行比较：一个值只与自身全等。对于数值，全等操作符使用略加修改的语义来处理两个特殊情况：第一个情况是，浮点数 0 是不分正负的。区分 +0 和 -0 在解决一些特定的数学问题时是必要的，但是大部分境况下我们并不用关心。全等操作符认为这两个值是全等的。第二个情况是，浮点数包含了 NaN 值，用来表示某些定义不明确的数学问题的解，例如：正无穷加负无穷。全等操作符认为 NaN 与其他任何值都不全等，包括它自己。（等式 (x !== x) 成立的唯一情况是 x 的值为 NaN）

## 非严格相等 ==

有些开发者认为，最好永远都不要使用相等操作符。全等操作符的结果更容易预测，并且因为没有隐式转换，全等比较的操作会更快。

关于非严格相等的具体内容可以通过下面的比较图来了解。

## 比较模型图

在 ES2015 以前，你可能会说双等和三等是“扩展”的关系。比如有人会说双等是三等的扩展版，因为他处理三等所做的，还做了类型转换。例如 6 == "6" 。反之另一些人可能会说三等是双等的扩展，因为他还要求两个参数的类型相同，所以增加了更多的限制。怎样理解取决于你怎样看待这个问题。

但是这种比较的方式没办法把 ES2015 的 Object.is 排列到其中。因为 Object.is 并不比双等更宽松，也并不比三等更严格，当然也不是在他们中间。从下表中可以看出，这是由于 Object.is 处理 NaN 的不同。注意假如 Object.is(NaN, NaN) 被计算成 false ，我们就可以说他比三等更为严格，因为他可以区分 -0 和 +0 。但是对 NaN 的处理表明，这是不对的。 Object.is 应该被认为是有其特殊的用途，而不应说他和其他的相等更宽松或严格。

x | y | == | === | Object.is
---|---|---|---|---
undefined  | undefined	|true	|true	|true
null	|null	|true	|true	|true
true	|true	|true	|true	|true
false	|false	|true	|true	|true
"foo"	|"foo"	|true	|true	|true
{ foo: "bar" }	|x	|true	|true	|true
0	|0	|true	|true	|true
+0	|-0	|true	|true	|false
0	|false	|true	|false	|false
""	|false	|true	|false	|false
""	|0	|true	|false	|false
"0"	|0	|true	|false	|false
"17"	|17	|true	|false	|false
[1,2]	|"1,2"	|true	|false	|false
new String("foo")	|"foo"	|true	|false	|false
null	|undefined	|true	|false	|false
null	|false	|false	|false	|false
undefined	|false	|false	|false	|false
{ foo: "bar" }	|{ foo: "bar" }	|false	|false	|false
new String("foo")	|new String("foo")	|false	|false	|false
0	|null	|false	|false	|false
0	|NaN	|false	|false	|false
"foo"	|NaN	|false	|false	|false
NaN	|NaN	|false	|false	|true

看眼花了是吗？那我们来一张色情版~

![图片版](http://ojd8i48oc.bkt.clouddn.com/blog-javascript-equal.png)

上图总结了三种不同的比较操作符对于各种情况的不同处理，不是很全面，比如 `'0' == false; // true ` 而 `'0' === false; // false `


## 什么时候使用 Object.is 或是三等

总的来说，除了对待NaN的方式，Object.is唯一让人感兴趣的，是当你需要一些元编程方案时，它对待0的特殊方式，特别是关于属性描述器，即你的工作需要去镜像Object.defineProperty的一些特性时。如果你的工作不需要这些，那你应该避免使用Object.is，使用===来代替。即使你需要比较两个NaN使其结果为true，总的来说编写使用NaN 检查的特例函数(用旧版本ECMAScript的isNaN方法)也会比想出一些计算方法让Object.is不影响不同符号的0的比较更容易些。

> 参考文章：

[参考文章1](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness)