---
title: 块状和行内元素
date: 2015/08/03
tags: [HTML]
author: jizhi.w77@foxmail.com
comments: true
---

对于块状元素和行内元素，从概念上了解他们的异同，知道大体上块级元素有哪些、行内元素有哪些，然后重点了解行内元素的padding和margin有哪些特殊的地方，基本上就可以了。

## 概念

### MDN总结

- 概念
块级元素占据其父元素（容器）的整个空间，默认宽度为父元素的100%，因此创建了一个“块”。一个行内元素只占据它对应标签的边框所包含的空间。
- 内容
一般情况下，行内元素只能包含数据和其他行内元素。而块级元素可以包含行内元素和其他块级元素。这种结构上的包含继承区别可以使块级元素创建比行内元素更”大型“的结构。
- 格式
默认情况下，行内元素不会以新行开始，而块级元素会新起一行

### 个人总结

行内元素和块级元素的区别无非是在DOM结构渲染形式上所表现出来的不同而已。
- 行内元素
 * 行内元素根据它对应标签所包含的内容`自适应宽度`，不多也不少，刚好够装下它所包含的内容。因此没有必要再提供处理宽高、边距属性的能力，所以行内元素的宽高属性是默认无效的，而且行内元素只要宽度足够就一直从左向右排列不会换新行。
 * 行内元素的padding属性可设置(left和right正常，但padding-top和padding-bottom会有异常效果，见下)，margin只有左右可设置，上下margin默认无效。(那些个ctrl+v党们请验证啊，不然误人又误己)。
- 块状元素
 * 默认占据其父元素的整个空间，也就是说它的宽度默认是父元素的100%，一个块状元素占据一行，即使他的宽度不够父元素容器的100%。一个块级元素占据一行，除非人为指定(比如float，各种position等)，那么一行只会存在一个块级元素。

## 盒模型对比

上面已经说过，行内元素设置宽高属性是无效的，这一点无需再讨论，重要的是行内元素的padding和margin属性的表现。先贴两张图说明问题。

- 行内元素设置padding-bottom

![行内元素设置padding-bottom](../../images/20150803/01.png)

- 块级元素设置padding-bottom

![行内元素设置padding-bottom](../../images/20150803/02.png)

> 总的来说，行内元素只有左右两个方向可以正常设置margin和padding属性，上下margin直接无效，而上下padding会出现如上图所示的异于块状元素的渲染方式。

## 分类

### 行内元素列表

- b, big, i, small, tt
- abbr, acronym, cite, code, dfn, em, kbd, strong, samp, var
- a, bdo, br, img, map, object, q, script, span, sub, sup
- button, input, label, select, textarea

### 块级元素列表

以下是 HTML 中所有的块级元素列表（虽然”块级“在新的 HTML5 元素中没有明确定义）。

- &lt;address&gt;
联系方式信息。
-  &lt;article&gt; `HTML5`
文章内容。
-  &lt;aside&gt; `HTML5`
伴随内容。
-  &lt;audio&gt; `HTML5`
音频播放。
- &lt;blockquote&gt;
块引用。
- &lt;canvas&gt; `HTML5`
绘制图形。
- &lt;dd&gt;
定义列表中定义条目描述。
- &lt;div&gt;
文档分区。
- &lt;dl&gt;
定义列表。
- &lt;fieldset&gt;
表单元素分组。
- &lt;figcaption&gt;  `HTML5`
图文信息组标题
- &lt;figure&gt;  `HTML5`
图文信息组 (参照 &lt;figcaption&gt;)。
- &lt;footer&gt;  `HTML5`
区段尾或页尾。
- &lt;form&gt;
表单。
- &lt;h1&gt;, &lt;h2&gt;, &lt;h3&gt;, &lt;h4&gt;, &lt;h5&gt;, &lt;h6&gt;
标题级别 1-6.
- &lt;header&gt; `HTML5`
区段头或页头。
- &lt;hgroup&gt; `HTML5`
标题组。
- &lt;hr&gt;
水平分割线。
- &lt;noscript&gt;
不支持脚本或禁用脚本时显示的内容。
- &lt;ol&gt;
有序列表。
- &lt;output&gt; `HTML5`
表单输出。
- &lt;p&gt;
行。
- &lt;pre&gt;
预格式化文本。
- &lt;section&gt; `HTML5`
一个页面区段。
- &lt;table&gt;
表格。
- &lt;tfoot&gt;
表脚注。
- &lt;ul&gt;
无序列表。
- &lt;video&gt; `HTML5`
视频。

## 参考文档

- [Block-level_elements](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)
- [Inline_elements](https://developer.mozilla.org/en-US/docs/Web/HTML/Inline_elements)
