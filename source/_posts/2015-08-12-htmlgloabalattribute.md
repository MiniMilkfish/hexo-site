---
title: HTML Global Attributes
date: 2015/08/12
tags: [HTML]
excerpt: HTML 定义了少量的全局属性，这些属性适用于所有的HTML elements。这些属性可以用在所有的element中，尽管有些属性对一些element没什么作用。对于一些`非规范`的元素这些属性应该依然有效。
author: jizhi.w77@foxmail.com
comments: true
---

## 概念

HTML 定义了少量的全局属性，这些属性适用于所有的HTML elements。也就是说这些属性可以用在所有的element中，尽管有些属性对一些element没什么卵用。

对于一些`非规范`的元素这些属性应该依然有效，因为谁让他们是全局的呢，比如对于`<my-tag id='tagId'></my-tag>`，我自定义了一个`my-tag`标签，其中该标签有id这个全局属性，但这一切都是可执行的。

## Global Attributes List

- accesskey
这个属性提供了一种使用快捷键访问当前元素的途径，怎样操作激活快捷键取决于浏览器实现。
- class
这个属性咱就不说了吧
- contenteditable
翻译过来即内容是否可编辑。这个可枚举的属性值表示element是否可被编辑，小心点，这个属性值会继承！
- contextmenu
表示没看懂而且浏览器的实现情况也很不乐观。
- data-\*
自定义属性，这个属性首先是一个整体的，即`data-*`是一个整体。被设置了这个属性的element，可以通过HTMElement的接口访问所有的自定义数据。HTMLElement.dataset属性提供了访问他们的权限。
注意：HTMLElement.dataset是一个 StringMap。一个名叫data-test-value的自定义属性可以通过HTMLElment.dataset.testValue来访问，属性的名字中的中线(U+002D)被挨着它字母的大写字母代替了（驼峰命名）。
- dir
文本书写方向属性，文档里建议文本书写方式的定义要运用此属性而不是写到css里的direction，不解为何！
- draggable
元素是否可拖动。默认只有text selection和images、links才能被拖动。可以结合DragDrop API来开发拖动程序。
- dropzone
该属性决定丢到一个element上的内容类型，copy、move和link。
- hidden
当一个网页元素有了hidden属性后，它的表现跟CSS的`display: none;`作用非常相似，元素将会消失，而且不占用任何页面空间。
- id
这个属性是唯一的标识，它在整个document里应该是唯一的。当需要链接（使用片段标识符，锚点），执行脚本，控制样式时，可以用它来定位识别元素。
-itemid
- itemprop
- itemref
- itemscope
- itemtype
- slot
可类比vuejs里的插槽
- lang
指定元素的语言
- style
内联样式，尽量避免！
- tabindex
该属性决定元素是否可以获得焦点和获得焦点的顺序，取值必须为整数。负数表示属性无效、0表示元素可通过键盘获取焦点，但是顺序依赖于平台、整数则直接定义了获取焦点的顺序，值越大则优先级越高。
- title
悬浮标题提示。
- translate
这是一个可枚举的属性，用于确定当页面进行本地化(localized)时，元素的属性值以及元素的文本(Text)子节点中的内容是否要进行翻译。其可取的值如下：空字符串(empty)或者 "yes"，表示这个元素相关的内容将会被翻译；"no"，表示这个元素相关的内容不会被翻译。

## MDN文档
- [Global_attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes)</div>