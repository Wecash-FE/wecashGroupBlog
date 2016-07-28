title: Koa源码阅读笔记(1) -- co
author: wecash FE team
date: 2016-07-26 22:37:42
tags:
---
> From 刘子健

## 起因
在7月23号时，我参加了北京的NodeParty。其中第一场演讲就是深入讲解Koa。
由于演讲只有一个小时，讲不完Koa的原理。于是在听的时候觉得并不是很满足，遂开始自己翻看源代码。
而Koa1是基于ES6的`generator`的。其在Koa1中的运行依赖于co。
正好自己之前也想看co的源代码，所以趁着这个机会，一口气将其读完。
## co
关于co,其作者的介绍很是简单。
> The ultimate generator based flow-control goodness for nodejs (supports thunks, promises, etc)

而co的意义，则在于使用`generator`函数，**解决了JavaScript的回调地狱问题**。

## 源码解读
co的源代码十分简洁，一共才两百余行。而且里面注释到位，所以阅读起来的难度还是不大的。
co的核心代码如下（已加上自己的注释）：

```javascript
/**
 * Execute the generator function or a generator
 * and return a promise.
 *
 * @param {Function} fn
 * @return {Promise}
 * @api public
 */

function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1)

  // we wrap everything in a promise to avoid promise chaining,
  // which leads to memory leak errors.
  // see https://github.com/tj/co/issues/180
  return new Promise(function(resolve, reject) {
    // 启动generator函数。
    if (typeof gen === 'function') gen = gen.apply(ctx, args);
    // 如果gen不存在或者gen.next不是函数（非generator函数）则返回空值
    if (!gen || typeof gen.next !== 'function') return resolve(gen);

    onFulfilled();

    /**
     * @param {Mixed} res
     * @return {Promise}
     * @api private
     */

    function onFulfilled(res) {
      var ret;
      try {
        // ret = gen.next return的对象
        // gen.next(res),则是向generator函数传参数，作为yield的返回值
        /**
         * yield句本身没有返回值，或者说总是返回undefined。
         * next方法可以带一个参数，该参数就会被当作上一个yield语句的返回值。
         * [next方法的参数](http://es6.ruanyifeng.com/#docs/generator#next方法的参数)
         */
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      // 在这儿，每完成一次yield，便交给next()处理
      next(ret);
    }

    /**
     * @param {Error} err
     * @return {Promise}
     * @api private
     */

    function onRejected(err) {
      var ret;
      try {
        ret = gen.throw(err);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }

    /**
     * Get the next value in the generator,
     * return a promise.
     *
     * @param {Object} ret
     * @return {Promise}
     * @api private
     */

    function next(ret) {
      // 如果这个generator函数完成了，返回最终的值
      // 在所有yield完成后，调用next()会返回{value: undefined, done: true}
      // 所以需要手动return一个值。这样最后的value才不是undefined
      if (ret.done) return resolve(ret.value);
      // 未完成则统一交给toPromise函数去处理
      // 这里的ret.value实际是 yield 后面的那个(对象|函数|值) 比如 yield 'hello', 此时的value则是 'hello'
      var value = toPromise.call(ctx, ret.value);
      // 这里value.then(onFulfilled, onRejected)，实际上已经调用并传入了 onFulfilled, onRejected 两个参数。
      // 因为非这些对象，无法调用then方法。也就无法使用onFulfilled
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"'));
    }
  });
}

/**
 * Convert a `yield`ed value into a promise.
 *
 * @param {Mixed} obj
 * @return {Promise}
 * @api private
 */

function toPromise(obj) {
  if (!obj) return obj;
  if (isPromise(obj)) return obj;
  if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);
  if ('function' == typeof obj) return thunkToPromise.call(this, obj);
  if (Array.isArray(obj)) return arrayToPromise.call(this, obj);
  if (isObject(obj)) return objectToPromise.call(this, obj);
  return obj;
}
```

## co的运行机制
看完了源代码，对`generator`函数有更深的理解，也理解了co的运行机制。
### 自动执行generator
首先解决的问题则是自动执行`generator`函数是如何实现的。
这儿的核心部分则在于: 

```javascript
function co(gen) {
  if (typeof gen === 'function') gen = gen.apply(ctx, args);
  onFulfilled();

  function onFulfilled(res) {
    var ret;
    try {
      ret = gen.next(res);
    } catch (e) {
      return reject(e);
    }
    next(ret);
  }
  function next(ret) {
    if (ret.done) return resolve(ret.value);
    var value = toPromise.call(ctx, ret.value);
    if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
    return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
      + 'but the following object was passed: "' + String(ret.value) + '"'));
  }
}
```

这儿，在给co传入一个`generator`函数后，co会将其自动启动。然后调用`onFulfilled`函数。
在`onFulfilled`函数内部，首先则是获取next的返回值。交由`next`函数处理。
而`next`函数则首先判断是否完成，如果这个generator函数完成了，返回最终的值。
否则则将`yield`后的值，转换为`Promise`。
最后，通过`Promise`的then，并将`onFulfilled`函数作为参数传入。

```javascript
if (value && isPromise(value)) {
  return value.then(onFulfilled, onRejected);
}
```

**而在`generator`中，`yield`句本身没有返回值，或者说总是返回`undefined`。**
**而next方法可以带一个参数，该参数就会被当作上一个`yield`语句的返回值。**
同时通过`onFulfilled`函数，则可以实现自动调用。
这也就能解释为什么co基于`Promise`。且能自动执行了。

## 结语
co的源代码读取来不难，但其处理方式却令人赞叹。
而且`generator`函数的使用，对ES7中的`Async/Await`的产生，起了关键作用。
正如其作者TJ在co的说明文档中所说的那样：
> Co is a stepping stone towards ES7 async/await. 

虽然说我没用过co，只使用过`Async/Await`。
但如今的`Async/Await`，使用babel，启用[transform-async-to-generator](http://babeljs.io/docs/plugins/transform-async-to-generator/)插件，转译后，也是编译为`generator`函数。
所以了解一下，还是有好处的。而且阅读co的源代码，是阅读koa1源码的必经之路。

---
前端路漫漫，且行且歌。