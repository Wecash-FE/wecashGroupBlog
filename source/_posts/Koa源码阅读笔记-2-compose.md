title: Koa源码阅读笔记(2) -- compose
author: wecash FE team
date: 2016-07-28 06:59:56
tags:
---
> From 刘子健

![2016-07-27_19:11:39.jpg](http://7xoxxe.com1.z0.glb.clouddn.com/2016-07-27_19:11:39.jpg)
## 起因
自从写了个Koa的脚手架[koa2-easy](https://github.com/Lxxyx/koa2-easy)，愈发觉得Koa的精妙。
于是抱着知其然也要知其所以然的想法，开始阅读Koa的源代码。

## 问题
读Koa源代码时，自然是带着诸多问题的。无论是上一篇所写的`generator`函数如何自动执行，还是对于Koa中间件如何加载，next参数如何来的。都充满了好奇。
今天写文章，并不是介绍整个`koa-compose`如何如何（涉及太宽，准备放在下面几篇统一介绍）。而是从自身需求出发，找到问题的答案。
而问题就是**Koa中间件的加载，和next参数的来源**。

## 源码解读
### 初始化与中间件加载
首先的是Koa加载初始化时的函数（删除部分）:

```javascript
// Koa类
function Application() {
  this.middleware = [];
}
// Koa原型
var app = Application.prototype;

// Koa中间件加载函数
app.use = function(fn){
  if (!this.experimental) {
    // es7 async functions are not allowed,
    // so we have to make sure that `fn` is a generator function
    assert(fn && 'GeneratorFunction' == fn.constructor.name, 'app.use() requires a generator function');
  }
  this.middleware.push(fn);
  return this;
};
```

在这儿不难看出，Koa对象内部有个中间件的数组，其中所有中间件都会存在其中。
而在服务器启动时，则会调用并处理该数组。
源代码如下：

```javascript
var co = require('co');
var compose = require('koa-compose');

var fn = co.wrap(compose(this.middleware))
```

在fn被处理完后，每当有新请求，便会调用fn，去处理请求。
而在这里，co.wrap的作用是返回一个`Promise`函数，用于后续自动执行`generator`函数。

### koa-compose
于是不难看出，中间件这儿的重点，是compose函数。
而compose函数的源代码虽然很简洁，但是也很烧脑。（对我而言）

```javascript
/**
 * Compose `middleware` returning
 * a fully valid middleware comprised
 * of all those which are passed.
 *
 * @param {Array} middleware
 * @return {Function}
 * @api public
 */

// 传入中间件作为参数
function compose(middleware){
  return function *(next){
    // next不存在时，调用一个空的generator函数
    if (!next) next = noop();
    
    var i = middleware.length;
    // 倒序处理中间件，给每个中间件传入next参数
    // 而next则是下一个中间件
    while (i--) {
      next = middleware[i].call(this, next);
    }

    return yield *next;
  }
}

function *noop(){}
```

在这里，得提一提`Koa`中间件的调用方式。

```javascript
app.use(function * (next) {
  this.set('Koa', 'Example');
  yield next;
})
app.use(function * (next) {
  this.body = 'Hello World'
})
```

在中间件中的next，则是在`koa-compose`中传入的。
而这儿， `yield next` 和 `yield *next`也是有区别的。
`yield next`, next 会作为next()的value返回。
而`yield *next`则是在`generator`函数内执行这个`generator`函数。

## 结语
这两天一直在读Koa的源代码，细细看来不是很难，但是被作者的奇思妙想给打动了。
接下来会继续写一些阅读笔记，因为看Koa的源代码确实是获益匪浅。

---
前端路漫漫，且行且歌