---
title: 再谈 CSS 预处理器
date: 2015-11-01
tags: [CSS]
republish: http://efe.baidu.com/blog/revisiting-css-preprocessors/
excerpt: 网上已经有不少对比目前最主流的三个预处理器 Less、Sass 和 Stylus（按字母顺序排名）的文章了，但是似乎都不是很详细，或者内容有些过时。下面我会更详细地探讨一下这三种预处理器的特性和它们的差异。
---

[文章](http://efe.baidu.com/blog/revisiting-css-preprocessors/)转载自EFE,作者[Justineo](http://weibo.com/u/1143654280)

CSS 预处理器是什么？一般来说，它们基于 CSS 扩展了一套属于自己的 DSL，来解决我们书写 CSS 时难以解决的问题：

- 语法不够强大，比如无法嵌套书写导致模块化开发中需要书写很多重复的选择器；
- 没有变量和合理的样式复用机制，使得逻辑上相关的属性值必须以字面量的形式重复输出，导致难以维护。

所以这就决定了 CSS 预处理器的主要目标：提供 CSS 缺失的样式层复用机制、减少冗余代码，提高样式代码的可维护性。这不是锦上添花，而恰恰是雪中送炭。

网上已经有不少对比目前最主流的三个预处理器 Less、Sass 和 Stylus（按字母顺序排名）的文章了，但是似乎都不是很详细，或者内容有些过时。下面我会更详细地探讨一下这三种预处理器的特性和它们的差异。

下面主要会分为如下几方面来讨论：

- 基本语法
- 嵌套语法
- 变量
- @import
- 混入
- 继承
- 函数
- 逻辑控制

事先声明一下，平时我在开发中主要使用的是 Less，所以可能对 Sass 和 Stylus 的熟悉程度稍差一些，比较时主要参考三者官网的语言特性说明，有一些正在开发的功能可能会遗漏。

本文中对 CSS 语法的话术与 MDN 的 CSS 语法介绍一致。

# 基本语法

Less 的基本语法属于「CSS 风格」，而 Sass、Stylus 相比之下激进一些，利用缩进、空格和换行来减少需要输入的字符。不过区别在于 Sass、Stylus 同时也兼容「CSS 风格」代码。多一种选择在更灵活的同时，在团队开发中也免不了增加更多约定来保持风格统一。而对个人而言，语法风格按自己口味选择即可。

**注：后面的 Sass 代码会用被更多人接受的 SCSS 风格给出。**

Less & SCSS：

```css
.box {
  display: block;
}
```

Sass：

```css
.box
  display: block
Stylus：
```

```css
.box
  display: block
```

# 嵌套语法

三者的嵌套语法都是一致的，甚至连引用父级选择器的标记 `&` 也相同。区别只是 Sass 和 Stylus 可以用没有大括号的方式书写。以 Less 为例：

```css
.a {
  &.b {
    color: red;
  }
}
```

生成的 CSS 为：

```css
.a.b {
  color: red;
}
```

除了规则集的嵌套，Sass 额外提供了一个我个人认为比较另（jī）类（lèi）的「属性嵌套」：

```css
.funky {
  font: {
    family: fantasy;
    size: 30em;
    weight: bold;
  }
}
```

# 选择器引用

三者都支持用 `&` 在嵌套的规则集中引用上层的选择器，这可以是嵌套书写 CSS 时的「惯例」了。语法相同，但是逻辑上有些许差异。在一个选择器中用两次以上 `&` 且父选择器是一个列表时，Less 会对选择器进行排列组合，而 Sass 和 Stylus 不会这么做。

也就是说，假设上层选择器为 `.a, .b`，则内部的 `& &` 在 Less 中会成为 `.a .a, .a .b, .b .a, .b .b`，而 Sass 和 Stylus 则输出 `.a .a, .b .b`。

假设我们要用预处理器书写 WHATWG 推荐的 section 标题样式，在 Less 中可以方便地书写为：

```css
article, aside, nav, section {
  h1 {
    margin-top: 0.83em; margin-bottom: 0.83em; font-size: 1.50em;
  }
  & & h1 {
    margin-top: 1.00em; margin-bottom: 1.00em; font-size: 1.17em;
  }
  & & & h1 {
    margin-top: 1.33em; margin-bottom: 1.33em; font-size: 1.00em;
  }
  & & & & h1 {
    margin-top: 1.67em; margin-bottom: 1.67em; font-size: 0.83em;
  }
  & & & & & h1 {
    margin-top: 2.33em; margin-bottom: 2.33em; font-size: 0.67em;
  }
}
```

当然，这个推荐样式十分脑残，编译出来的结果会有 47KB 之巨，根本不可用，这里只是借来演示一下。

除了 `&`，Sass 和 Stylus 更进一步，分别用 `@at-root` 和 `/` 符号作为嵌套时「根」规则集的选择器引用。这有什么用呢？举个例子，假设 HTML 结构是这样的：

```html
<article class="post">
  <h1>我是一篇文章</h1>
  <section>
    <h1 class="section-title"><a href="#s1" class="section-link">#</a>我是章节标题</h1>
    <p>我只是一个<em>例子</em>。</p>
  </section>
</article>
```

如果我这么写 Sass 代码，是完全符合业务的嵌套关系的：

```css
.post {
  section {
    .section-title {
      color: #333;
      .section-link {
        color: #999;
      }
    }
    /* other section styles */
  }
  /* other post styles */
}
```

但是这样生成出来的选择器会有 .post section .section-title .section-link，很多时候我们觉得写成 .post .section-link 就够了。

于是我们在 Stylus 中可以这么写：

```css
.post
  section
    .section-title
      color #333
      /.post .section-link
        color #999
    /* other section styles */

  /* other post styles */
```

这样输出的 CSS 就会是：

```css
.post section .section-title {
  color: #333;
}
.post .section-link {
  color: #999;
}
```

这就是我们想要的样子了。当然也可以这样写：

```css
.post
  section
    .section-title
      color #333
    /* other section styles */

  .section-link
    color #999
  /* other post styles */
```

我个人是推荐这种写法（不使用 root 引用）的，因为当你确定 .section-link 的样式不依赖于它位于 section 或 .section-title 下时，就不应该嵌套于此。否则如果为了一点点性能上的考虑（还不一定会是优化），使得设计意图变得更不准确，我觉得得不偿失。

# 变量

变量无疑为 CSS 增加了一种有效的复用方式，减少了原来在 CSS 中无法避免的重复「硬编码」。

Less：

```css
@red: #c00;

strong {
  color: @red;
}
```

Sass：

```css
$red: #c00;

strong {
  color: $red;
}
```

```css
Stylus：

red = #c00

strong
  color: red
```

Less 的选择有一个问题：@ 规则在 CSS 中可以算是一种「原生」的扩展方式，变量名用 @ 开头很可能会和以后的新 @ 规则冲突。（当然理论上只要 CSS 规范不引入 @a: b 这样的规则，问题也不大。而且规范制定的时候也会参考很多现有的实现。）

相比之下 Sass 的选择中规中矩，而 Stylus 就不同了，不需要额外的标志符。这意味着：在 Stylus 中，我们可以覆写 CSS 原生的属性值！Stylus 的设计让人有一种「你以为你在写 CSS，但其实你不是」的感觉，后面会有更多这样的例子。

顺便说一下，CSS 规范也有关于变量实现的草案，目前的方案是这个样子的：

```css
/* global scope */
:root {
  --red: #c00;
}

strong {
  color: var(--red);
}
```

不管语法槽点如何，原生 CSS 变量可以通过 DOM 结构来继承，也就是说是代码真正「运行」时（runtime）决定的。元素引用一个变量时会按 DOM 向上查找定义在上层元素上的同名变量。这一点是任何预处理语言都无法做到的。可以用 Firefox 31+ 看一下这个 [demo](http://jsbin.com/webuju/1/edit)。至于这种机制是不是好用，暂时还没研究过。不过从开发的思维惯性来看，还很难一下子适应这种方式。

# 变量作用域

三种预处理器的变量作用域都是按嵌套的规则集划分，并且在当前规则集下找不到对应变量时会逐级向上查找，注意这个和原生 CSS 的逻辑是完全不同的。

如果我们在代码中重写某个已经定义的变量的值，Less 的处理逻辑和其他两者有非常关键的区别。在 Less 中，这个行为被称为「懒加载（Lazy Loading）」。所有 Less 变量的计算，都是以这个变量最后一次被定义的值为准。举一个例子更容易说清楚：

Less：

```css
@size: 10px;
.box {
    width: @size;
}

@size: 20px;
.ball {
    width: @size;
}
```

输出：

```css
.box {
  width: 20px;
}
.ball {
  width: 20px;
}
```

而在 Stylus 中：

```css
size = 10px
.box
  width: size

size = 20px
.ball
  width: size
```

输出：

```css
.box {
  width: 10px;
}
.ball {
  width: 20px;
}
```

Sass 的处理方式和 Stylus 相同，变量值输出时根据之前最近的一次定义计算。这其实代表了两种理念：Less 更倾向接近 CSS 的声明式，计算过程弱化调用时机；而 Sass 和 Stylus 更倾向于指令式。这两种方式会导致怎样的结果呢？

举个例子来说，对于 Less，如果项目中引入了这样一个文件：

```css
@error-color: #c00;
@success-color: #0c0;
.error {
  color: @error-color;
  background-color: lighten(@error-color, 40%);
}
.success {
  color: @success-color;
  background-color: lighten(@success-color, 40%);
}
```

在业务代码中，在不修改外部引入文件的情况下，如果我想重写这两种状态的配色，只需要重新配置 `@error-color` 和 `@success-color` 这两个变量，就能改变 `.error` 和 `.success` 的样式。

而在 Stylus 中，如果引入的第三方样式库中有这样的代码：

```css
error-color = #c00
success-color = #0c0

.error
  color: error-color
  background-color: lighten(error-color, 40%)

.success
  color: success-color
  background-color: lighten(success-color, 40%)
```

这种情况下后面的代码就无法通过重写变量值来覆盖样式了。Sass 也是如此。优点是 Stylus 和 Sass 这样的处理会不容易受多个第三方库变量名冲突的影响，因为一个变量不能影响在定义它以前的输出样式。

由于 Sass 和 Stylus 变量在「运行」过程中使用完可以修改后再使用输出不同的值，所以这两者还提供了「仅当变量不存在时才赋值」的功能：

Sass：

```css
$x: 1;
$x: 5 !default;
$y: 3 !default;

// $x = 1, $y = 3
```

Stylus：

```css
x = 1
x := 5 // or x ?= 5
y = 3

// x = 1, y = 3
```

因为变量只能在输出前修改才能生效，所以如果要定制第三方库的样式，用户代码理论上得插入第三方库的配置与样式之间才能生效。而有了 !default，第三方库在提供默认配置时可以将开发给用户修改的变量设置为 !default，这样只要用户提前引入配置进行覆盖，就可以按需重写默认配置了：

```css
// lib.scss
$alert-color: red !default;
.alert {
  color: $alert-color;
}
```

```css
// var.scss
$alert-color: #c00;
```

```css
// page.scss
@import var
@import lib
```

这样最终页面输出的效果就是被用户重定义过的内容了。

```css
/* page.css */
.alert {
  color: #c00;
}
```

由于 Less 处理变量的方式，如果我们要引入多个外部样式库或在多个团队进行合作开发时，如果不能确保开发过程可控，那为变量添加模块前缀就变得很有必要。

此外，Sass 中提供一个 !global 的语法来让局部变量变成全局变量，也就是说 Sass 代码可以在内层覆盖全局变量的值。输出一段局部的样式可能使得后续所有样式都受到全局变量变化的影响。（这其实是 Sass 开始时默认的逻辑，Sass 3.3 以前所有变量都是全局的，之后改成了和 Less 和 Stylus 一样有嵌套作用域，全局变量要显式指定 !global。）

# 插值

预处理器都有定义变量的功能，除了在最常见的属性值中使用，其他还有哪些地方能用变量来增强对样式的抽象、复用呢？

#### 变量名插值

Less 中支持 @@foo 的形式引用变量，即该变量的名字是由 @foo 的值决定的。比如我们可以利用它简化更清晰地调用 mixin：

```css
// some icon font lib

// variables with prefix to prevent conflicts
@content-apple: "A";
@content-google: "G";

// clearer argument values
.icon-content(@icon) {
  @var: ~"content-@{icon}";
  &::before {
    content: @@var;
  }
}

.icon-apple {
  .icon-content(apple); // "A"
}

.icon-google {
  .icon-content(google); // "G"
}
```

#### 选择器插值

选择器是样式表和 DOM 的纽带，是我们实际暴露给 HTML 的接口。支持插值显然可以让接口更不容易和其他内容冲突。假设我们在开发一个 UI 库，生成的组件类名希望有一个可配置的前缀，这时选择器插值就变得相当重要。初看下来，三者用法类似：

Less：

```css
@prefix: ui;
.@{prefix}-button {
  color: #333;
}
```

Sass：

```css
$prefix: ui
.#{$prefix}-button
  color: #333;
```

Stylus：

```css
prefix = ui
.{prefix}-button
  color #333
```

但是在 Less 中，有一个很严重的问题：通过选择器插值生成的规则无法被继承（Extend dynamically generated selectors）！当然，如果有类似 Placeholder 的机制，这都不是事儿了。问题是 Less 没有！未来的方案看来可能是通过 :extend(.mixin()) 的方式实现类似功能（:extend mixins）。虽然用 :extend 本身的语法说不过去，但是在现有机制上来看还算可以接受。关于样式的继承复用，后面会详细讲到。

#### @import 语句插值

Sass 中只能在使用 url() 表达式引入时进行变量插值：

```css
$device: mobile;
@import url(styles.#{$device}.css);
```

Less 中可以在字符串中进行插值：

```css
@device: mobile;
@import "styles.@{device}.css";
```

Stylus 中在这里插值不管用，但是可以利用其字符串拼接的功能实现：

```css
device = "mobile"
@import "styles." + device + ".css"
```

注意由于 Less 的 Lazy Load 特性，即使是 @import 也是可以在后面的文件内容中进行覆盖的，修改掉变量就可以在前面引入不同的外部文件。而 Sass 与 Stylus 一旦输出语句，就无法通过变量改变了。

#### 属性名插值

三个预处理器的目前版本都支持属性名插值，用法也类似。这里仅以 Stylus 为例：

```css
red-border(sides)
  for side in sides
    border-{side}-color: red // property name interpolation

.x
  red-border(top right)
```

输出：

```css
.x {
  border-top-color: #f00;
  border-right-color: #f00;
}
```

#### 其他 @ 规则插值

三种预处理器均支持在 @media、@keyframes、@counter-style 等规则中进行插值。@media 插值主要用来做响应式的配置，而 @keyframes 这样带名称名称的 @ 规则则可以通过插值来避免命名冲突。

Less：

```css
@m: screen;
@orient: landscape;
@media @m and (orientation: @orient) {
  body {
    width: 960px;
  }
}

@prefix: ui;
@keyframes ~"@{prefix}-fade-in" {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
```

Sass：

```css
$m: screen;
$orient: landscape;
@media #{$m} and (orientation: $orient) {
  body {
    width: 1000px;
  }
}

$prefix: ui;
@keyframes #{$prefix}-fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
```

Stylus：

```css
m = screen
orient = landscape
mq = m + " and (orientation: " + orient + ")"
@media mq
  body
    width: 960px

vendors = official
prefix = ui;
@keyframes {prefix}-fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
```

三者均会输出如下 CSS：

```css
@media screen and (orientation: landscape) {
  body {
    width: 960px;
  }
}
@keyframes ui-fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
```

Stylus 中似乎有 and 时由于表达式计算的逻辑不能直接像 Less 与 Sass 那样写插值，所以这里采用了字符串拼接的方式。

# @import

@import 对于模块化开发来说非常有帮助，但就这个功能来说，三种预处理器的行为各不相同。

先说 Less，Less 扩展了语法，为 @import 增加了多种选项：

1. @import (less) somefile.ext
会将无论什么扩展名的文件都作为 Less 文件引入、一起编译；

2. @import (css) somefile.ext
直接编译生成 @import somefile.ext，当做原生 @import；

3. @import (inline) somefile.ext
直接将外部文件拷贝进输出文件的这个位置，但不会参与编译；

4. @import (reference) somefile.ext
外部文件参与编译，但不输出内容，仅用来被本文件中的样式继承；

5. @import (optional) somefile.ext
引入文件但在文件不存在时不报错，静默失败。

上面的选项是可以联合使用的，比如可以这样写：

```css
@import (less, optional) somefile.ext;
```

除此之外还有 once 和 multiple 选项分别用来表示去重和不去重的引入方式，默认为 once。在不写任何选项时，Less 会根据扩展名进行推断来决定引入逻辑。

Sass 没有扩展语法，而是自己推断引入的方式。.css 后缀、绝对路径、url() 表达式和带有 media query 的 @import 会直接用原生 @import，其他都会作为 Sass 代码参与编译。相比之下 Less 更灵活也更复杂。Sass 有个特有的功能叫做「partial」，因为 Sass 默认的编译工具可以编译整个目录下的文件，所以当一些文件不需要编译时，可以在文件名前加上 _ 表明这是一个被别的模块引入本身不需要编译的代码片段。Less 的 lessc 由于本来就只处理一个文件，所以这件事就交给用户自己去写编译脚本了。Sass 中有一个比较棘手的问题是，@import 不会被去重，多次引入会导致一个样式文件被多次输出到编译结果中。

# 混入

混入（mixin）应该说是预处理器最精髓的功能之一了。它提供了 CSS 缺失的最关键的东西：样式层面的抽象。从语法上来说，三种预处理器的差异也比较大，这甚至会直接影响到我们的开发方式。

Less 的混入有两种方式：

1. 直接在目标位置混入另一个类样式（输出已经确定，无法使用参数）；
2. 定义一个不输出的样式片段（可以输入参数），在目标位置输出。（注：后面如无特殊说明，mixin 均用来指代此类混入。）

举例来说：

```css
.alert {
  font-weight: 700;
}

.highlight(@color: red) {
  font-size: 1.2em;
  color: @color;
}

.heads-up {
  .alert;
  .highlight(red);
}
```

最后输出：

```css
.alert {
  font-weight: 700;
}
.heads-up {
  font-weight: 700;
  font-size: 1.2em;
  color: red;
}
```

可以混入已有类样式这一点很值得商榷。在上面的例子中，.alert 样式在被混入时甚至可以是 .alert();；.highlight() 混入时也可以写成 .highlight;。那么我们遇到这样的代码时根本不知道 alert 会不会是一个 HTML class。但由于这一点是在 Less 还不支持 extend 时就有的，所以也能够理解作者可能就是将这作为 extend 来用了。所以目前比较好的实践是：用代码规范规约开发者不得使用直接混入已有类样式的方式，而是先定义 mixin 然后在输出的类样式中进行调用，调用时必须显式加上 () 来表明这不是一个 class（事实上百度 EFE 已有的 Less 编码规范就是这么定义的）。继承则应该直接通过 Less 的 :extend 来实现。

另外需要注意的是，Less 在进行混入时，会找到所有符合调用参数的「mixin 签名」的样式一起输出。比如：

```css
.mixin(dark; @color) {
  color: darken(@color, 10%);
}
.mixin(light; @color) {
  color: lighten(@color, 10%);
}
.mixin(@_; @color) {
  display: block;
}

@switch: light;
.class {
  .mixin(@switch; #888);
}
```

这个例子中，第二个和第三个 mixin 都匹配了调用时的参数，于是它们的规则都会被输出：

```css
.class {
  color: #a2a2a2;
  display: block;
}
```

也就是说同名的 mixin 不是后面覆盖前面，而是会累加输出。只要参数符合定义，就会将 mixin 内部的样式规则、甚至变量全部拷贝到目标作用域下。

这一点同样会带来一个问题：如果存在和 mixin 同名的 class 样式，如果 mixin 没有参数则在调用时会把对应的 class 样式一起输出，这显然是不符合预期的。

假设有个叫 .clearfix 的 mixin，有两个 class 样式调用了它（其中一个也叫 clearfix）：

```css
.clearfix() {
  *zoom: 1;
  &:before,
  &:after {
    display: table;
    content: "";
  }
}

.clearfix {
  .clearfix();
}

.list {
  .clearfix();
}
```

得到的输出是：

```css
.clearfix {
  *zoom: 1;
}
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}
.list {
  *zoom: 1;
}
.list:before,
.list:after {
  display: table;
  content: "";
}
.list:after {
  clear: both;
}
.list:before,
.list:after {
  display: table;
  content: "";
}
.list:after {
  clear: both;
}
```

.list 的样式调用了两次！这一点在开发中一定要注意，不要给和非输出型 mixin 同名的类定义样式。

对于 Sass，语义非常明确：

```css
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  color: #ff0000;
}

.page-title {
  @include large-text;
  padding: 4px;
  margin-top: 10px;
}
```

Sass 用 @mixin 和 @include 两个指令清楚地描述了语义，不存在混入类样式的情况，但是书写时略显繁琐一些。当然，用 Sass 语法 而非 SCSS 语法的话可以简单地用 = 定义 mixin，用 + 引入 mixin：

```css
=large-text
  font:
    family: Arial
    size: 20px
    weight: bold
  color: #ff0000

.page-title
  +large-text
  padding: 4px
  margin-top: 10px
```

和 Less 不同，同名的 mixin 可以覆盖之前的定义，作用机制类似变量。

Stylus 和 Sass 类似，但不用什么特殊的标记来引入：

```css
border-radius(n)
  -webkit-border-radius: n
  -moz-border-radius: n
  border-radius: n

.circle
  border-radius(50%)
```

Stylus 中还有一个「透明 mixin」的功能，也就是说引入 mixin 完全可以和引入普通属性一样！例如上面的这个 mixin，也可以这样引入：

```css
.circle
  border-radius: 50%
```

这意味着可以把兼容性上的处理隐藏在 mixin 中，直接用标准属性同名的 mixin 按普通属性的方式输出。当不需要兼容老浏览器时，直接把 mixin 定义删除仍然能够正常输出。不过这种写法虽然感觉非常「爽快」，但要求开发者必须能很好地区分原生属性和某个样式库中提供的 mixin 功能（对于有经验的开发者问题不大），而且透明意味着看到一个普通属性开发者不能判断是否已经在某处用 mixin 进行了重写，无法明确知道这里的代码最后输出会不会发生变化。在可控条件下，这个功能应该说是非常诱人的。

# 继承

混入很好用，可也有问题：如果多个地方都混入同样的代码，会造成输出代码的多次重复。比如在 Stylus 下：

```css
message()
  padding: 10px
  border: 1px solid #eee

.message
  message()

.warning
  message()
  color: #e2e21e
```

会输出：

```css
.message {
  padding: 10px;
  border: 1px solid #eee;
}
.warning {
  padding: 10px;
  border: 1px solid #eee;
  color: #e2e21e;
}
```

而我们可能期望的输出是：

```css
.message,
.warning {
  padding: 10px;
  border: 1px solid #eee;
}
.warning {
  color: #e2e21e;
}
```

也许大家会说可以这么写：

```
message()
  padding: 10px
  border: 1px solid #eee

.message,
.warning
  message()

.warning
  color: #e2e21e
```

这样就可以按需要输出了。但其实预处理器的一个好处就是可以方便我们进行模块化开发。上面的例子中，.message 和 .warning 的样式如果是分布在两个模块中的，我合并过的选择器组样式写在哪里呢？情况更复杂的时候就更棘手了。

这个时候就该继承出场了：

```css
.message
  padding: 10px
  border: 1px solid #eee

.warning
  @extend .message
  color: #e2e21e
```

这样就可以按模块进行开发（不管是分文件还是在同一文件中按业务功能安排样式的顺序），同时兼顾输出的效率了。

Stylus 的继承方式来自 Sass，两者如出一辙。 而 Less 则又「独树一帜」地用伪类来描述继承关系：

```css
.message {
  padding: 10px;
  border: 1px solid #eee;
}

.warning {
  &:extend(.message);
  color: #e2e21e;
}
/* Or:
.warning:extend(.message) {
  color: #e2e21e;
}
*/
```

同时，Less 默认只继承父类本身的样式，如果要同时继承嵌套定义在父类作用域下的样式，得使用关键字 all，比如 &:extend(.message all);。

关于使用伪类描述继承关系，Hax 在 Less 的另一个 issue 下曾经言辞激烈地提出了批评，同时也遭到了 Less 项目组毫不客气的回应。我个人完全赞同 Hax 的看法，因为选择器是用来在树结构中找到元素的，和样式本身完全无关。但 Less 社区在当时却对这个语法表示了一致的赞同，不禁让人对其感到担忧。

不管语法如何，继承功能还有一个潜在的问题：继承会影响输出的顺序。假设有如下的 Sass 代码：

```css
.active {
   color: red;
}
button.primary {
   color: green;
}
button.active {
   @extend .active;
}
```

而对应的 HTML 代码是：

```html
<button class="primary active">Submit</button>
```

很容易误以为效果是红色的。而其实生成的 CSS 顺序如下：

```css
.active, button.active {
  color: red;
}

button.primary {
  color: green;
}
```

由于合并选择器的关系 .active 被移到了 .primary 之前，所以依赖顺序而非选择器 specificity 时可能会遇到陷阱。

# 总结

我个人认为，Less 从语言特性的设计到功能的健壮程度和另外两者相比都有一些缺陷，但因为 Bootstrap 引入了 Less，导致 Less 在今天还是有很多用户。用 Less 可以满足大多数场景的需求，但相比另外两者，基于 Less 开发类库会复杂得多，实现的代码会比较脏，能实现的功能也会受到 DSL 的制约。比 Stylus 语义更清晰、比 Sass 更接近 CSS 语法，使得刚刚转用 CSS 预编译的开发者能够更平滑地进行切换。当初 Sass 并不支持 SCSS 语法，使得转投 Sass 成本较高，所以 Alexis Sellier 才萌生开发一个更「CSS」的预处理器的念头。大获成功以后反过来影响到了 Sass，迫使其也支持类似 CSS 语法的 SCSS。另外，Less 支持浏览器端编译，这无疑降低了开发门槛，使得很多非专业的开发者能够更快地上手（对于一些个人项目来说，能让项目跑起来就行，对前端的性能并没有专业工程师那么高的要求）。

Sass 在三者之中历史最久，也吸收了其他两者的一些优点。从功能上来说 Sass 大而全，语义明晰但是代码很容易显得累赘。主项目基于 Ruby 可能也是一部分人不选择它的理由（Less 开始也是基于 Ruby 开发，后来逐渐转到 less.js 项目中）。 Sass 有一个「事实标准」库——Compass，于是对于很多开发者而言省去了选择类库的烦恼，对于提升开发效率也有不小的帮助。

Stylus 的语法非常灵活，很多语义都是根据上下文隐含的。基于 Stylus 可以写出非常简洁的代码，但对使用团队的开发素养要求也更高，更需要有良好的开发规范或约定。Stylus 是前 Node.js 圈第一大神 TJ Holowaychuk 的作品，虽然他已经弃坑了，但是仍然有不小的号召力。和 Sass 有 Compass 类似，Stylus 有一个官方开发的样式库 nib，同样提供了不少好用的 mixin。对于比较有经验的开发者，用 Stylus 可能更会有一种畅快的感觉。总的来说用一个词形容 Stylus 的话，我会用「sexy」。

总的来说，三种预处理器百分之七八十的功能是类似的。Less 适合帮助团队更快地上手预处理代码的开发，而 Sass 和 Stylus 的差异更在于口味。比如有的人喜欢 jQuery 用一个 $ 做大部分的事，而另一些人觉得不一样的功能就该有明确的语义上的差别。在这里我不会做具体的推荐。当然，再次声明一下由于我个人接触 Less 开发比较多，所以可能遇到的坑也多一些，文中没有列出 Sass 和 Stylus 的问题并不代表他们没有。

最后打个广告：百度 EFE 目前有一个基于 Less 的样式库 est，以及一个基于 Stylus 的针对移动端的样式库 rider，欢迎大家关注、提交 issue 和 pull request。