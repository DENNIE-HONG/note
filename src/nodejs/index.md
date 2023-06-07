# NodeJs
## 1. 工作模式
单线程的问题：
1. cpu的利用率
2. 进程的健壮性

解决：  多进程：一个进程一个CPU，child-process  
Master-Workder模式（主从）：主进程负责调度与管理，工作进程负责业务处理。

## 2. 通信
进程间通信：管道技术；
主进程：创建IPC通道并监听；
子进程：文件描述符 -> 连接 -> IPC
## 3. 负载均衡
操作系统的抢占式策略：端口共同监听，TCP服务端socket套接字的文件描述符相同。但是只有一个进程能够抢到连接。

## 4. Node中的事件循环
6个阶段
```
   ┌───────────────────────┐
┌─>│        timers         │setTimeout、setInterval预定的callback
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │某些系统操作的回调
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │执行 setImmediate() 设定的callbacks;
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │比如 socket.on(‘close’, callback) 的callback会在这个阶段执行
   └───────────────────────┘
```
event loop的每一次循环都需要经过6个阶段。每个阶段有自己的callback队列，每当进入某个阶段，都会从所属队列中取出callback执行。当队列为空或者执行callback数量达系统最大数量时，进入下一个阶段。这6个阶段都执行完称为一轮循环。  
各个阶段切换的中间执行微任务。当执行栈里的同步代码执行完毕切换到node的event loop时，也属于阶段切换，也先去清空微任务。



例如：setImmediate与setTimeout
* setTimeout：poll阶段空闲 && 没定时间到达，在timers阶段执行；  
* setImmediate: poll阶段完成，check阶段；


```js
const fs = require('fs');
fs.readFile(_filename, () => {
  setTimeout(()=> {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
// 输出immediate timeout  
```
1. 在异步I/O callback之外调用，执行先后顺序不定！！！
2. 在异步I/O callback内部调用，先执行setImmediate后setTimeout

readFile回调在poll中进行，发现有setimmedate, 跳到check阶段执行回调，再去timer阶段执行setTimeout

栗子：
```js
console.log(1);
console.log(2);
setTimeout(function() {
  console.log('setTimeout1');
  Promise.resolve().then(function() {
    console.log('promise');
  });
});
setTimeout(() => {
  console.log('setTimeout2');
});
// 1
// 2
// setTimeout1
// setTimeout2
// promise
```
1. 先执行完栈里的代码。
2. 从栈进入到event loop的timer阶段，由于nodejs的event loop是每个阶段的callback执行完毕才进入下一个阶段，所以会打出timers的2个settimeout回调；
3. 微任务不在event loop任何阶段，而是在各个阶段切换的中间执行。即从一个阶段切换到下一个阶段前执行。timers阶段callback执行完，执行微任务。


再举个栗子：
```js

```









## 5. 模块机制
* 模块引用：require()
* 模块定义：exports是module的属性，导出方法or变量
* 模块标识：传给require的参数，小驼峰命名or路径

### Nodejs模块类型
* 核心模块：包含在Node.js源码中，可执行二进制文件js模块，native模块，如https,fs...;
* c/c++模块：built-in模块；
* native模块：如buffer、fs、os，底层都有调用built-in模块；
* 第三方模块：如express等；

nodejs会缓存引入模块，且是编译和执行之后的对象。  
文件模块：
```js
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
    parent.children.push(this);
  }
  this.filename = null;
  this.loaded = false;
  this.children = [];
}
```
编译成功的模块：文件路径缓存在Module._cache

### require？
require定义在Module的原型链上。
```js
Module.prototype.require = function(path) {
  ...
  return Module._load(path, this, false);
}
```
Module._load(request, parent, isMain):
1. 根据文件名，调用Module._resolveFilename解析文件路径；
2. 查看缓存Module._cache中是否有该模块，如果有，直接返回；
3. 通过NativeModule.noInternalExists判断是否为核心模块，是 ->NativeModule.require;
4. 不是核心，新对象Module -> tryModuleLoad()

#### 核心模块加载原理
NativeModule.require:
1. cache中是否有，有返回；
2. 新建nativeModule对象，缓存加载编译compile
```js
NativeModule.wrap = function (script) {
  return NativeModule.wrapper[0] + script + NativeModule.wrapper[1];
};
NativeModule.wrapper = [
  '(function (exports, require, module, _filename, _dirname) {',
  '\n});'
];

NativeModule.prototype.compile = function() {
  var source = NativeModule.getSource(this.id);
  source = NativeModule.wrap(source);
  this.loading = true;
  try {
    const fn = runInThisContext(source, {
      filename: this.filename,
      lineOffset: 0,
      displayError: true
    });
    fn(this.exports, NativeModule.require, this, this.filename);
    this.loaded = true;
  } finally  {
    this.loading = false;
  }
}

```

#### 第三方模块加载原理
tryModuleLoad() -> Module.prototype.load();  
检测filename扩展名，针对不同扩展名调用不同Module。  
_extensions方法来加载、编译模块。  
Module._extensions: 
* js: fs读取 -> module._compile
* json: 读取 -> JSON.parse
* node: process.dlopen




## 6. 异步I/O
优势：第一个资源请求不会阻塞第2个。消耗时间为Max(m,n, ...), 特别是分布式服务器，而不是sum(m, n, ...).  
调用非阻塞I/O的过程：
1. 打开文件描述符
2. 获取数据立即返回（不带数据）
3. 重复调用I/O确认是否完成（轮询）即判断状态

内部完成I/O任务的另有线程池。





## 7. Koa
启动前：
* Request：操作node原生请求对象的方法，如获取query、url等；
* Response：包含用于设置状态码，主体数据、header等一些用于操作响应的方法；
* Context: Koa的上下文，部分是自身属性，一部分是Request和Response委托操作方法。

### 启动server
```js

const koa = require('koa');
const app = koa();
app.listen(1995);
```
初始化：
```js
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
### listen端口：
```js
app.listen = function() {
  const server = http.createServer(this. callback());
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

### koa-compose
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


### responsed
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
### co
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
    if (gen || typeof gen.next !== 'function') {
      return resolve(gen);
    }
    onFulfilled();

    function next(ret) {
      // 中间件结束
      if (ret.done) {
        return resolve(ret.value);
      }
      // 确保返回值是promise对象。
      var value = toPromise.call(ctx, ret.value);
      if (value && isPromise(value)) {
        return value.then(onFulfilled, onReject);
      }
      return onReject(new TypeError(''));
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
  });
}
```

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