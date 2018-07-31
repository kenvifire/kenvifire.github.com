---
layout: post
title: "编写一个JavaScript框架 - 项目结构"
description: "编写一个JavaScript框架"
category: 
tags: [js-framework,javascript]
---
{% include JB/setup %}

## 编写一个JavaScript框架 - 项目结构
在过去的几个月里，RisingStack的JavaScript工程师Bertalan Miklos编写了一个下一代客户端框架，叫做NX。在这个《自己动手写JavaScript框架》系列里，Bertalan分享了他在这整过过程中所学习到的内容：

在这个章节里，我会讲解NX项目的结构以及我是如何解决扩展性，依赖注入以及私有变量这些难点的。

整个系列包含下列内容：
1. 项目结构（当前章节）
2. 执行时间
3. 沙盒代码执行
4. 数据绑定简介
5. 通过ES6代理进行数据绑定
6. 自定义元素
7. 客户端路由

### 项目结构
虽然一个可以适用于所有项目的结构，但是对于项目结构，我们还是有一些通用的规范的。感兴趣的话，可以参考我们《Node Hero》系列中的_Node.js项目结构指南_。

### NX JavaScript框架概览

NX的目标是成为由开源社区驱动的项目，它既容易扩展，也适用于不同规模的项目。

- 它拥有所有现代客户端框架的所有功能
- 除了polyfills，它没有任何外部依赖
- 它总共才3000行代码
- 没有一个模块超过300行代码
- 没有一个功能模块有超过3个依赖

NX最终的依赖图如下：
[http://blog-assets.risingstack.com/2016/Jul/javascript\_framework\_in\_2016\_the\_nx\_project\_structure-1467708098586.svg]
NX项目的这种结构对于框架项目相关的几个难点提供了很好地解决方案。
- 可扩展性
- 依赖注入
- 私有变量

### 达到扩展性
对于一个依赖社区驱动的项目而言，易于扩展是很重要的一个特性。要达到这点，项目本身必须拥很少的核心功能以及内置的依赖处理功能。前者保证项目本身易于被理解和接受，后者则保证项目能一直保护这点。

在这个章节里，我会重点关于维护较少的核心功能。

现代框架里很重要的一点就是提供自定义组件并且在DOM使用该组件的能力。NX仅有一个`component`函数作为它的核心功能。这个函数可以让用户自己配置和注册新的组件。
''  component(config)
''       .register('comp-name')
上面代码里注册的是一个空的组件类型，注册之后就可以直接在DOM使用了。
'' <comp-name></comp-name>

接下来就是需要保证组件能够可以扩展新的功能。但是为了保证简洁性和扩展性，这些新功能的代码不能够影响核心模块的代码。所以，我们需要引入依赖注入的功能。

### 通过中间层进行依赖注入
如果你对依赖注入不是很熟悉的话，我建议你去阅读我们关于这个话题的文章：_Node.js依赖注入.

> 依赖注入是一种设计模式，在这种模式里，一个或者多个依赖（也可以是服务）可以被注入或者过引用的方式，设置到依赖它们的对象里。

依赖注入解决了一个棘手的问题，但是又引入了一个新的问题。用户必须知道如何去配置和注入所有的依赖。绝大部分的客户端框架都有容器来负责进行依赖管理，而不需要用户去操心。

> 依赖注入容器是也是一个对象，它知道如何去初始化和配置它所管理的对象。

另外一种方式是中间层依赖注入模式，这个模式在服务端框架（Express，Koa）上很常见。这种方式的诀窍在于所有的可注入的依赖（也叫做中间层）都有着相同的接口并且可以通过相同的方式进入注入。在这种方式下，是不需要容器来进行依赖管理的。

为了保持简洁性，我选择了中间层注入这个方式。如果你用过Express的话，下面的代码应该就比较熟悉了。
'' component()
''   .use(paint) // 注入paint中间层
''   .use(resize) // 注入resize中间层
''   .register('comp-name')
'' 
'' function paint (elem, state, next) {
''   // elem 是具体的组件实例，可以在这里设置或者进行扩展
''   elem.style.color = 'red'
''   //然后可以继续执行下一个组件 (resize)
''   next()
'' }
'' 
'' function resize (elem, state, next) {
''   elem.style.width = '100 px'
''   next()
'' }

中间层会在新的组件实例被加载到DOM上的时候执行，特别是在给组件扩展新功能的时候比较有用。通过多个不同的库来给同一个对象进行扩展，就会带来命名冲突的问题。对外暴露私有变量可以掩盖这个问题，但是也可能会造成私有变量会意外地被其他地方使用到。

仅暴露最少的公用API并且隐藏其他实现部分是避免这些问题的最佳实践。

### 隐藏私有内容

隐私性在JavaScript里是通过函数域来控制的。当我们需要进行跨域访问一些私有变量时，人们往往会将这些私有变量暴露出来，并且加上前缀`_`来进行标记。这个能够提示使用者去避免不必要的使用，但是也还是解决不了命名冲突的问题。一个更好的解决方案是使用ES6的`Symbol`类型。

> Symbol是一个唯一的且不可变的数据类型，它可以被用来最为对象属性的唯一标识。

下面的代码就是一个symbol的实例。
'' const color = Symbol()
'' 
'' // a middleware
'' function colorize (elem, state, next) {
''   elem[color] = 'red'
''   next()
'' }

现在red只能通过拥有color（以及对应的element）的引用才能使用到。私有变量`red`的访问范围可以通过控制`color`来进行控制。当我们有比较多的私有变量时，通过统一的symbol来进行存储是个很棒的解决方案。

'' // symbols module
'' exports.private = {
''   color: Symbol('color from colorize')
'' }
'' exports.public = {}

添加下面这个`index.js`文件。
'' // main module
'' const symbols = require('./symbols')
'' exports.symbols = symbols.public

这个存储在当前project的所有模块里都是可以进行访问的，但是里面的私有变量并没有暴露给外面。对外暴露的公共部分可以提供给外部开发者访问一些底层的特性。这样就可以避免意外而非必要地访问私有部分，因为如果要使用私有变量的话，开发者是需要显示地去依赖这个symbol存储的。除此之外，symbol引用不会像字符串名字那样会产生冲突，因此也就不会产生命名冲突的问题。

1. ####  公共变量

正常进行使用就可以了。
'' function (elem, state, next) {
''   elem.publicText = 'Hello World!'
''   next()
'' }

2. ##### 私有变量
跨作用域的对于当前项目而言是私有的变量，需要通过key放到私有变量存储里。
'' // symbols module
'' exports.private = {
''   text: Symbol('private text')
'' }
'' exports.public = {}

并且在使用的时候进行依赖。
'' const private = require('symbols').private
'' 
'' function (elem, state, next) {
''   elem[private.text] = 'Hello World!'
''   next()
'' }

3. 半私有变量
	对于底层API使用的变量，应该把它放到一个公共的symbol存储里。

'' // symbols module
'' exports.private = {
''   text: Symbol('private text')
'' }
'' exports.public = {
''   text: Symbol('exposed text')
'' }

然后再使用的时候进行依赖。
'' const exposed = require('symbols').public
'' 
'' function (elem, state, next) {
''   elem[exposed.text] = 'Hello World!'
''   next()
'' }

### 结论
如果你对NX框架感兴趣的话，请访问我们的主页。
喜欢挑战的读者可以直接在github仓库里查看NX框架的代码。

希望你觉得这篇文章对你是有帮助的，下次我们会讨论执行时间这个主题。

如果你对当前主题有任何意见，请在评论里进行分享。




