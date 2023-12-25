# Koa
启动前：
* Request：操作node原生请求对象的方法，如获取query、url等；
* Response：包含用于设置状态码，主体数据、header等一些用于操作响应的方法；
* Context: Koa的上下文，部分是自身属性，一部分是Request和Response委托操作方法。

## 启动server
```js

const koa = require('koa');
const app = koa();
app.listen(1995);
```
初始化：
```js
const response = require('./response');
const context = require('./context');
const request = require('./request');
// 简单伪代码
function Application() {
  if (!(this instanceof Application)) {
    return new Application;
  }
  // 初始化空的中间件集合
  this.middleware = [];
  this.context = Object.create(context);
  this.request = Object.create(request);
  this.response = Object.create(response);
}
// request.js
module.exports = {
  // 暴露了一些getter方法等
  get header() {...},
  get url() {...},
  get origin() {...},
  get href() {...}
}
```
## listen端口：
伪代码：
```js
app.listen = function() {
  const server = http.createServer(this. this.callback());
  return server.listen.apply(server, arguments);
};
app.callback = function() {
  // 是否支持es7,初始化中间件
  const fn = this.experimental ? compose._es7(this.middleware): co.wrap(comose(this.middleware));
  const self = this;
  return function(req, res) {
    res.statusCode = 404;
    const ctx = self.createContext(req, res);
    // 监听response，有错误执行ctx.onerror
    onFinished(res, ct.onerror);
    fn.call(ctx).then(function() {
      respond.call(ctx);
    }).catch(ctx.onerror);
  };
}
```

## koa-compose v2.5.1
把一个个不相干的中间件串联在一起
使用：
```js
this.middlewares = [function *m1(){}, function * m2() {}, function * m3(){}];

function * noop() {}

function compose(middleware) {
  return function * (next) {
    if (!next) {
      next = noop(); // 空函数
    }
    let i = middleware.length;
    while(i--) {
      next = middleware[i].call(this, next);
    }
    return yield * next;
  }
}

const middleware = compose(this.middlewares);

// 转换后得到
function *() {
  yield * m1(m2(m3(noop())));
}
```
1. 先把中间件从后往前依次执行，把每个中间件执行后得到generator对象赋值给next；
2. 第一次执行最后一个中间件，next为空的generator函数；第二次执行倒2个中间件，next为最后一个返回generator...
3. 执行compose与执行第一个中间件效果一致；

故fn = co.wrap(function *() {
  yield * m1(m2(m3(noop)))
});



### koa-compose v4.2.0版本

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  middleware = flatten(middleware)
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1;
    function dispatch (i) {
        if (i <= index) return Promise.reject(new Error('next() called multiple times'))
        index = i;
        let fn = middleware[i];
        if (i === middleware.length) fn = next
        if (!fn) return Promise.resolve();
        try {
            return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
        } catch (err) {
            return Promise.reject(err);
        }
    }
    return dispatch(0)
  }
}
```

也就是说koa-compose返回的是一个Promise，Promise中取出第一个函数（app.use添加的中间件），传入context和第一个next函数来执行。
第一个next函数里也是返回的是一个Promise，Promise中取出第二个函数（app.use添加的中间件），传入context和第二个next函数来执行。
第二个next函数里也是返回的是一个Promise，Promise中取出第三个函数（app.use添加的中间件），传入context和第三个next函数来执行。
第三个...
以此类推。最后一个中间件中有调用next函数，则返回Promise.resolve。如果没有，则不执行next函数。 这样就把所有中间件串联起来了。这也就是我们常说的洋葱模型。






## responsed
```js
fn.call(ctx).then(function() {
  responsed.call(ctx);
}).catch(ctx.onerror);
```
koa的response操作都是this.body = xxx;
this.status = xxx;
```js
function responsed() {
  if (false === this.responsed) {
    return;
  }
  const res = this.res;
  if (res.headerSent || !this.writeable) {
    return;
  }
  let body = this.body;
  const code = this.status;
  if (statuses.empty[code]) {
    this.body = null;
    return res.end();
  }
  // 读取body中的值，根据不同类型来决定以
  // 什么类型响应请求
  if ('HEAD' === this.method) {
    if (isJSON(body)) {
      this.length = Buffer.byteLength(JSON.stringify(body));
    }
    return res.end();
  }
  if (null === body) {
    this.type = 'text';
    body = this.message || String(code);
    this.length = Buffer.byteLength(body);
    return res.end(body);
  }
  if (Buffer.isBuffer(body)) {
    return res.end(body);
  }
  if ('string' === typeof body) {
    return res.end(body);
  }
  if (body instanceof Stream) {
    return body.pipe(res);
  }
  body = JSON.stringify(body);
  this.length = Buffer.byteLength(body);
  res.end(body);
}
```

```js
co.wrap = function (fn) {
  createPromise._generatorFunction_ = fn;
  function createPromise() {
    return co.call(this, fn.apply(this, arguments));
  }
  return createPromise;
}
```





## co

> https://github.com/berwin/Blog/issues/8
简单来说co，就是把generator自动执行，再返回一个promise。 generator函数这玩意它不自动执行呀，还要一步步调用next()，也就是叫它走一步才走一步。
```js
// 多个yeild，传参情况
function* generatorFunc(suffix = ''){
  const res = yield request();
  console.log(res, 'generatorFunc-res' + suffix);

  const res2 = yield request();
  console.log(res2, 'generatorFunc-res-2' + suffix);
}
```
简单版：
```js
function coSimple(gen){
  const ctx = this;
  const args = Array.prototype.slice.call(arguments, 1);
  gen = gen.apply(ctx, args);
  console.log(gen, 'gen');

  const ret = gen.next();
  const promise = ret.value;
  promise.then(res => {
    const ret = gen.next(res);
    const promise = ret.value;
      promise.then(res => {
        gen.next(res);
      });
  });
}

coSimple(generatorFunc, ' 哎呀，我真的是后缀')
```

```js
function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1);
  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') {
      gen = gen.apply(ctx, args);
    }
     // 如果不是函数 直接返回
    if (!gen || typeof gen.next !== 'function') {
      return resolve(gen);
    }
    onFulfilled();

    function next(ret) {
        // 中间件结束
        if (ret.done) {
            return resolve(ret.value);
        }
        // 确保返回值是promise对象。 用co在包一层toPromise.call(ctx, ret.value);
        var value = toPromise.call(ctx, ret.value);
        if (value && isPromise(value)) {
            return value.then(onFulfilled, onReject);
        }
        return onReject(new TypeError('You may only yield a function, promise, generator, array, or object, '
            + 'but the following object was passed: "' + String(ret.value) + '"')));
    }
    function onFulfilled(res) {
        var ret;
        try {
            // 包含下一个中间件generator对象
            ret = gen.next(res);
        } catch (e) {
            return reject(e);
        }
        next(ret);
    }

    function onReject(err) {
        var ret;
        try {
            ret = gen.throw(err);
        } catch (e) {
            return reject(e);
        }
        next(ret);
    }
  });
}
```

所以我们可以看到，如果第二个中间件里依然有yield next这样的语句，那么第三个中间件依然会被co包裹一层并运行.next方法，依次列推，这是一个递归的操作

所以我们可以肯定的是，每一个中间件都被promise包裹着，直到有一天中间件中的逻辑运行完成了，那么会调用promise的resolve来告诉程序这个中间件执行完了。

那么中间件执行完了之后，会触发onFulfilled，这个函数会执行.next方法。

所以有一个非常重要的一点需要注意，onFulfilled这个函数非常重要，重要在哪里？？？重要在它执行的时间上。

**onFulfilled这个函数只在两种情况下被调用，一种是调用co的时候执行，还有一种是当前promise中的所有逻辑都执行完毕后执行**

其实就这一句话就能说明koa的中间件为什么会回逆。

回逆其实是有一个去和一个回的操作



```js
 请求
    |
 ___|_______________
|   | 中间件        |
|  _|____________   |
| | | 中间件     |   |
| | ↓_______    |   |
| | | 中间件-|———|———|———> 响应
| | |_______|   |   |
| |_____________|   |
|___________________|
```

例如：3个中间件
1. 当请求时，调用co,调用onFulfilled来调用next往下执行。将返回得到的结果(第2个中间件generator对象)传到co中执行一遍，以此类推。
2. 运行到最后一个yield，等中间件最后一个执行结束，调用promise的resolve表示结束。（一个generator对象yield只能next一次，下次执行next时从上一次停顿的yield处继续执行）
3. 一旦第3个中间件触发成功，第2个立即调用onFulfilled来执行next，继续从第2个中间件上一次yield停顿处开始执行下面代码；
4. 第2个中间件逻辑执行完毕，执行resolve表示成功，而第1个中间件正好通过then方法监听了第2个中间件的promise。立即调用onFulfilled函数执行next方法。继续从第1个中间件上一次yield停顿处继续执行下面逻辑，以此类推。
