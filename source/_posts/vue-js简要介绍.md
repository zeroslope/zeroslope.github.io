---
title: Vue.js简要介绍
date: 2018-04-10 23:22:16
tags: 前端
---

## 为什么要使用前端框架 ？

> 在过去的十年中，网页变得更加动态化和强大了。多亏了JavaScript，我们把很多服务端代码放到了浏览器中，这样就产生了成千上万的JavaScript代码，它们连接了各式各样的HTML和CSS文件，但缺乏正规的组织形式，这也是为什么越来越多的开发者使用JavaScript框架。

在了解了HTML、CSS基础知识后，理论上就可以实现你想要的任何静态网页；再学习了JavaScript的知识后，就可以为网页添加丰富的交互。当网页的功能越多时，代码就越复杂，再加上没有固定的标准，代码变得越来越难以维护。

- 框架的使用可以节省开发时间，提高代码重用性，让开发变得更简单，当然框架本身的学习成本也需要考虑在内；
- 框架使用可以提升网页渲染的效率，提升交互体验；
- 框架的使用也是对开发者的一种约束，按照一定的标准开发，更容易debug，同时也为多人协作提供了基本的保证

## Why Vue.js ？

Vue是一款友好的、多用途且高性能的JavaScript框架，能够帮我们创建可维护性和可测试性更强的代码库。

- Approachable
- Versatile
- Performant

Vue是渐进式的JavaScript框架，也就是说，如果你已经有一个现成的服务端应用，你可以将Vue作为该应用的一部分嵌入其中，带来更加丰富的交互体验。或者你希望将更多业务逻辑放到前端来实现，那么Vue的核心库及其生态系统也可以满足你的各种需求。

像其他前端框架一样，Vue允许你将一个网页分割成可复用的组件，每个组件都包含属于自己的HTML、CSS、JavaScript以用来渲染网页中相应的地方。

具体的与其他框架的详细比较可以官网的[对比其他框架](https://cn.vuejs.org/v2/guide/comparison.html)


## 简单体验一下Vue.js

我们从网页中需要展示的数据开始，假设我们有一家水果店，要显示每种商品的数量。

我们的示例将从这里开始：

{% jsfiddle 50bd3roo %}

首先，用一个简单的`<h2>`标签来包裹我们要表示的数据，product是我们要显示的商品。

``` html
<div id="app">
    <h2> {{ product }} 在架上。</h2>
</div>
```

当我们打开这个网页时会发现显示的就是`{{ product }} 在架上。`，并没有什么特殊的地方，`id`将会在后面用到，现在忽略他。

让我们来引入Vue,在`<head>`中加上这个标签：

``` html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

然后，在`<body>`中加入下面的代码：

```html
<script>
    const app = new Vue({
        el: '#app',
        data: {
            product: "苹果"
        }
    })
</script>
```

重新刷新浏览器，你会发现现在显示的是`香蕉在架上。` 。Vue成功的把我们数据和HTML联系在了一起。但是vue的魔力在数据变更时才会出现。

如果我们打开控制台，更改`product`的值，在`product`的值发生改变的同时，vue自动更新了我们的HTML。这是因为VUE是响应式的`Reactive`，也就是说当我们的数据变更时，Vue会更新所有网页中用到它的地方。除了字符串，Vue对其它类型的数据也是如此。

所以我们把简单的商品换成一个商品数组试试看:

``` javascript
product: ["苹果", "草莓", "香蕉"]
```

然后我们将`<h2>`改成一个无序列表，再为数组中的每个商品创建一个列表项：使用Vue的`v-for`指令让每个商品都有各自的列表项。

``` html
<div id="app">
    <ul>
        <li v-for="product in products">
            {{ product }}
        </li>
    </ul>
</div>
```

不过如果这还不够有说服力，现在我们先从空列表开始，然后从一个实际的API获取我们的商品信息，当应用被创建时，我们会从这个API获取最新的商品信息。获取到商品列表后，Vue会自动更新网页中的信息，各列表项展示了该API返回的对象。

```json
{ "id": 1, "quantity": 1, "name": "苹果" }
{ "id": 2, "quantity": 0, "name": "草莓" }
{ "id": 3, "quantity": 5, "name": "香蕉" }
{ "id": 4, "quantity": 2, "name": "芒果" }
```

这还不是常人能够读懂的样式，所以我们改变一下他的展示方式。

``` vue
 <li v-for="product in products">
     {{ product.name }}： {{ product.quantity }}
</li>
```

假设我们需要特别注意数量为零的商品，所以我们添加一个`<span>`，写入文字`已售完`，这里用到了`v-if`指令。因为我们的JACKET数量为零，所以显示已经售完。

如果我们想要显示商品的总数怎么办呢？

在HTML中加入下面的代码，用来显示商品总数：

```html
<h3>商品总数: {{ totalProducts }}</h3>
```

 然后在Vue的实例中添加下面的计算变量：

``` javascript
computed: {
    totalProducts () {
        var sum = 0;
        for(var i = 0;i < this.products.length; ++ i) sum += this.products[i].quantity;
        return sum;
    }
}
```

刷新页面，在我们的浏览器中，已经把商品总数准确的计算出来了。

现在我们再来体会一下Vue的响应式效果，在浏览器的控制台中输入`app.products.pop()`，这个函数的作用是删除`products`数组中的最后一个元素。运行这条命令之后，我们发现芒果那一行已经不见了，而商品总数也随之发生了改变。

接下来我们在页面中添加一些交互行为，通过按钮来完成。每点击一次按钮，对应的商品数量就增加1。当我们添加一个项目时，不只是更新整个库存，同时增加草莓的数量时，`已售完`的提示也不见了。

如果想要增加很多个商品，重复点击可能会很麻烦，所以我们来实现可以直接改变商品数量的功能。我们只需要创建一个文本框，并通过`v-model`指令，绑定我们的商品数量字段并指明之一定是一个数字，而不是其他的数据类型。现在我们可以直接输入每个项目的总数，并且数据会被立刻更新。当我们把数量设置为0时，又看到了`已售完`的提示；而且现在增加按钮仍然能够正确的运行。

最终的demo如下:

{% jsfiddle 9he08jo6 %}

### Vue Devtools

一个非常有用的Vue调试工具，安装地址和使用方法[这里](https://github.com/vuejs/vue-devtools)

在上面的网页中打开这个调试工具，大概长这样：

![Ck6yLT.png](https://s1.ax1x.com/2018/04/11/Ck6yLT.png)

可以通过这个调试工具方便的查看数据，修改数据。



## Vue.js基础介绍

> **注：**
以下绝大多数内容来自[vue.js官方文档](https://cn.vuejs.org/v2/guide/)

> Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与[现代化的工具链](https://cn.vuejs.org/v2/guide/single-file-components.html)以及各种[支持类库](https://github.com/vuejs/awesome-vue#libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

### 声明式渲染
Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统：

``` html
<div id="app">
  {{ message }}
</div>
```

``` javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
实现的效果如下:

{% raw %}
<div id = "app" class="demo">
  {{ message }}
</div>
<style>
.demo {
  border: 1px solid #eee;
  border-radius: 2px;
  padding: 10px 35px;
  margin-top: 1em;
  margin-bottom: 20px;
  font-size: 1.2em;
  line-height: 1.5em;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  overflow-x: auto;
}
</style>

<script>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
</script>
{% endraw %}
我们已经成功创建了第一个 Vue 应用！看起来这跟渲染一个字符串模板非常类似，但是 Vue 在背后做了大量工作。现在数据和 DOM 已经被建立了关联，所有东西都是**响应式的**。

我们要怎么确认呢？

打开你的浏览器的 JavaScript 控制台 (就在这个页面打开)，并修改 `app.message` 的值，你将看到上例相应地更新。

除了文本插值，我们还可以像这样来绑定元素特性：

``` html
<div id="app-2">
  <span v-bind:title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>
```

``` javascript
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date().toLocaleString()
  }
})
```

{% raw %}
<div id="app-2" class="demo">
  <span v-bind:title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>

<script>
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date().toLocaleString()
  }
})
</script>
{% endraw %}

这里我们遇到了一点新东西。你看到的 `v-bind` 特性被称为**指令**。指令带有前缀 `v-`，以表示它们是 Vue 提供的特殊特性。它们会在渲染的 DOM 上应用特殊的响应式行为。在这里，该指令的意思是：“将这个元素节点的 `title` 特性和 Vue 实例的 `message` 属性保持一致”。

如果你再次打开浏览器的 JavaScript 控制台，输入 `app2.message = '新消息'`，就会再一次看到这个绑定了 `title` 特性的 HTML 已经进行了更新。

### 条件与循环

```html
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>
```

``` javascript
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```

{% raw %}
<div id="app-3" class="demo">
  <p v-if="seen">现在你看到我了</p>
</div>

<script>
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
</script>
{% endraw %}

继续在控制台输入 `app3.seen = false`，你会发现之前显示的消息消失了。

这个例子演示了我们不仅可以把数据绑定到 DOM 文本或特性，还可以绑定到 DOM **结构**。此外，Vue 也提供一个强大的过渡效果系统，可以在 Vue 插入/更新/移除元素时自动应用[过渡效果](https://cn.vuejs.org/v2/guide/transitions.html)。

还有其它很多指令，每个都有特殊的功能。例如，`v-for` 指令可以绑定数组的数据来渲染一个项目列表：

``` html
<div id="app-4">
    <ul>
        <li v-for="product in products">
            {{ product }}
        </li>
    </ul>
</div>
```

``` javascript
var app4 = new Vue({
  el: '#app-4',
  data: {
      products: [
          "苹果", 
          "草莓", 
          "香蕉"
      ]
  }
})
```
{% raw %}
<div id="app-4" class="demo">
    <ul>
        <li v-for="product in products">
            {{ product }}
        </li>
    </ul>
</div>

<script>
var app4 = new Vue({
  el: '#app-4',
  data: {
      products: [
          "苹果", 
          "草莓", 
          "香蕉"
      ]
  }
})
</script>
{% endraw %}

在控制台里，输入 `app4.products.pop()`，你会发现列表最后少了一种水果；输入 `app4.products.push('芒果')`，你会发现芒果出现在了列表中。

### 处理用户输入

为了让用户和你的应用进行交互，我们可以用 `v-on` 指令添加一个事件监听器，通过它调用在 Vue 实例中定义的方法：

``` html
<div id="app-5">
  <p> {{ nums }} <p>
  <button v-on:click="popNums">删除最后一个元素</button>
</div>
```

``` javascript
var app5 = new Vue({
  el: '#app-5',
  data: {
      nums: [1, 2, 3, 4, 5, 6]
  },
  methods: {
    popNums () {
      if(this.nums.length > 0)  this.nums.pop();
    }
  }
})
```
{% raw %}
<div id="app-5" class="demo">
  <p> {{ nums }} <p>
  <button v-on:click="popNums">删除最后一个元素</button>
</div>

<script>
var app5 = new Vue({
  el: '#app-5',
  data: {
      nums: [1, 2, 3, 4, 5, 6]
  },
  methods: {
    popNums () {
      if(this.nums.length > 0)  this.nums.pop();
    }
  }
})
</script>
{% endraw %}

注意在 `popNums` 方法中，我们更新了应用的状态，但没有触碰 DOM——所有的 DOM 操作都由 Vue 来处理，我们编写的代码只需要关注逻辑层面即可。

Vue 还提供了 `v-model` 指令，它能轻松实现表单输入和应用状态之间的双向绑定。

``` html
<div id="app-6">
  <p>Hello, {{ name }}!</p>
  <input v-model="name">
</div>
```

``` javascript
var app6 = new Vue({
  el: '#app-6',
  data: {
    name: 'FDDN'
  }
})
```
{% raw %}
<div id="app-6" class="demo">
  <p>Hello, {{ name }}!</p>
  <input v-model="name">
</div>

<style>
h3 > a:before {
  content: "#";
  color: #42b983;
  position: absolute;
  left: -0.7em;
  margin-top: -0.05em;
  padding-right: 0.5em;
  font-size: 1.2em;
  line-height: 1;
  font-weight: bold;
}
</style>

<script>
var app6 = new Vue({
  el: '#app-6',
  data: {
    name: 'FDDN'
  }
})
</script>
{% endraw %}

注意：

之前的命令都是单向的数据绑定，只有变量改变时，DOM中的元素才发生改变，反之则不行；而`v-model`是双向的数据绑定，一方改变，另一方也会改变。

在控制台中输入`app6.name='abcd'`,会发现输入框也变成了`abcd`；将输入框中改为`1234`,在控制台中输入`app6.name`,发现`app6.name`的值变成了`1234`。

### 模板语法

Vue.js 使用了基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。所有 Vue.js 的模板都是合法的 HTML ，所以能被遵循规范的浏览器和 HTML 解析器解析。

在底层的实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，Vue 能够智能地计算出最少需要重新渲染多少组件，并把 DOM 操作次数减到最少。

#### 插值
##### 文本
数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

``` html
<span>Message: {{ msg }}</span>
```

Mustache 标签将会被替代为对应数据对象上 `msg` 属性的值。无论何时，绑定的数据对象上 `msg` 属性发生了改变，插值处的内容都会更新。

通过使用 [v-once 指令](https://cn.vuejs.org/v2/api/#v-once)，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定：

``` html
<span v-once>这个将不会改变: {{ msg }}</span>
```

##### 原始HTML

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 `v-html` 指令：

``` html
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

这个 `span` 的内容将会被替换成为属性值 `rawHtml`，直接作为 HTML——会忽略解析属性值中的数据绑定。注意，你不能使用 `v-html` 来复合局部模板，因为 Vue 不是基于字符串的模板引擎。反之，对于用户界面 (UI)，组件更适合作为可重用和可组合的基本单位。

##### 特性

Mustache 语法不能作用在 HTML 特性上，遇到这种情况应该使用 [v-bind 指令](https://cn.vuejs.org/v2/api/#v-bind)：

``` html
<div v-bind:id="dynamicId"></div>
```

在布尔特性的情况下，它们的存在即暗示为 `true`，`v-bind` 工作起来略有不同，在这个例子中：

``` html
<button v-bind:disabled="isButtonDisabled">Button</button>
```

如果 `isButtonDisabled` 的值是 `null`、`undefined` 或 `false`，则 `disabled` 特性甚至不会被包含在渲染出来的 `<button>` 元素中。

##### 使用 JavaScript 表达式

迄今为止，在我们的模板中，我们一直都只绑定简单的属性键值。但实际上，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。

``` html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

这些表达式会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。有个限制就是，每个绑定都只能包含**单个表达式**，所以下面的例子都**不会**生效。

``` html
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

#### 指令

指令 (Directives) 是带有 `v-` 前缀的特殊属性。指令属性的值预期是**单个 JavaScript 表达式**(`v-for` 是例外情况，稍后我们再讨论)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。回顾我们在介绍中看到的例子：

``` html
<p v-if="seen">现在你看到我了</p>
```

这里，`v-if` 指令将根据表达式 `seen` 的值的真假来插入/移除 `<p>` 元素。

##### 参数

一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，`v-bind` 指令可以用于响应式地更新 HTML 属性：

``` html
<a v-bind:href="url">...</a>
```

在这里 `href` 是参数，告知 `v-bind` 指令将该元素的 `href` 属性与表达式 `url` 的值绑定。

另一个例子是 `v-on` 指令，它用于监听 DOM 事件：

``` html
<a v-on:click="doSomething">...</a>
```

在这里参数是监听的事件名。

##### 修饰符

#### 缩写

`v-` 前缀作为一种视觉提示，用来识别模板中 Vue 特定的特性。当你在使用 Vue.js 为现有标签添加动态行为 (dynamic behavior) 时，`v-` 前缀很有帮助，然而，对于一些频繁用到的指令来说，就会感到使用繁琐。同时，在构建由 Vue.js 管理所有模板的[单页面应用程序 (SPA - single page application)](https://en.wikipedia.org/wiki/Single-page_application) 时，`v-` 前缀也变得没那么重要了。因此，Vue.js 为 `v-bind` 和 `v-on` 这两个最常用的指令，提供了特定简写：

##### `v-bind`缩写

``` html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

##### `v-on`缩写

``` html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

它们看起来可能与普通的 HTML 略有不同，但 `:` 与 `@` 对于特性名来说都是合法字符，在所有支持 Vue.js 的浏览器都能被正确地解析。而且，它们不会出现在最终渲染的标记中。缩写语法是完全可选的，但随着你更深入地了解它们的作用，你会庆幸拥有它们。