# NodeJs
Node.js是一个开源且跨平台的JavaScript运行时环境，它在浏览器外运行V8 JavaScript引擎（Google Chrome的内核），利用事件驱动、非阻塞和异步输入输出模型等技术来提高性能，使得JavaScript可以用于服务器端开发。Node.js采用非阻塞型I/O机制，在处理I/O操作时不会阻塞进程，而是通过事件通知的方式处理操作结果。这种机制使得Node.js在处理高并发场景时表现优秀。

**优点**
* 处理高并发场景性能更佳：Node.js适用于处理大量并发请求的场景，由于其非阻塞I/O和事件驱动模型，能够高效地处理并发请求。
* 适合I/O密集型应用：Node.js擅长处理I/O密集型应用，其执行效率仍然较高，即使在运行极限时，CPU占用率仍然较低。
* 轻量快速：Node.js采用V8引擎作为底层，使得其执行速度快，且具有较小的内存占用。

**缺点**
* 不适合CPU密集型应用：由于Node.js是单线程的，如果应用程序中有大量计算密集型任务，会导致阻塞，影响整个应用的性能。
* 单线程：Node.js采用单线程事件循环，虽然能够高效处理I/O密集型应用，但是无法充分利用多核CPU，不能充分发挥多核处理器的优势。
* 可靠性低：由于Node.js是单线程的，一旦某个环节崩溃，整个系统都会崩溃，可靠性较低。
## 1. 工作模式
单线程的问题：
1. cpu的利用率
2. 进程的健壮性

解决：  多进程：一个进程一个CPU，child-process
Master-Workder模式（主从）：主进程负责调度与管理，工作进程负责业务处理。

node进程中并非只有一个线程，包含：
1. 1个js执行主线程；
2. 1个watching监控线程用于处理调试信息；
3. 1个V8 task线程用于调度任务优先级；
4. 4个v8线程：代码调优与GC后台、异步I/O;

单线程指的是js执行主线程只有一个。

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

#### 查找规则
1. 核心模块： 首先，Node.js会尝试查找是否为内置模块;
2. 文件模块：Node.js将尝试查找文件模块;
3. 目录作为模块: 
如果使用了目录作为模块名，并且目录中包含一个package.json文件，则Node.js会查找该文件中指定的main入口文件。如果package.json不存在或者未指定main，则Node.js会尝试加载目录下的index.js或index.node文件作为入口；
4. 非原生模块： 
如果以上查找都没有找到模块，Node.js会将模块名解析为绝对路径，并按照一定的路径顺序在文件系统中查找node_modules目录。Node.js会从当前模块的目录开始查找，然后逐级向上查找父目录的node_modules，直到根目录。如果找到了node_modules目录，则进入其中查找对应模块。


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
3. 通过NativeModule.noInternalExists判断是否为**核心模块**，是 ->NativeModule.require;
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


```mermaid
---
title: require查抄策略
---
flowchart TB
    isCache{moduele._cache缓存有吗} --有--> 返回
    isCache--没有--> built{核心模块吗}
    built--是--> NativeModule.require --> isBC{有缓存吗} --有--> A[返回]
    built--不是--> tryModule--> 检测扩展名 --> 不同扩展名调用不同moudle --> 加载,编译模块Module._extensions
    
    isBC --没有--> 缓存加载编译
    
```




## 6. 异步I/O
优势：第一个资源请求不会阻塞第2个。消耗时间为Max(m,n, ...), 特别是分布式服务器，而不是sum(m, n, ...).
调用非阻塞I/O的过程：
1. 打开文件描述符
2. 获取数据立即返回（不带数据）
3. 重复调用I/O确认是否完成（轮询）即判断状态

内部完成I/O任务的另有线程池。








## 7. EventEmitter
在Node.js中，EventEmitter是事件驱动的基础，几乎所有模块都继承自它。它实现了观察者模式，其中被观察者维护一组观察者，并在更新时通知观察者。


### 基本用法

```js
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

function callback() {
    console.log('触发了event事件！');
}

myEmitter.on('event', callback);
myEmitter.emit('event');
myEmitter.removeListener('event', callback);

```






### 如何实现Event库(发布订阅模式)？

公司：滴滴、头条

```js
class EventEmeitter {
    constructor() {
        this._events = this._events || new Map();
    }

    emit (type, ...args) {
        let handlers = this._events.get(type);
        if (!handers) {
            return false;
        }
        for (let i = 0; i < handlers.length; i++) {
            if (args.length > 0) {
                handlers[i].apply(this, args);
            } else {
                handlers[i].call(this);
            }
        }
        return true;
  }

    addListener (type, fn) {
        const handlers = this._events.get(type);
        if (!handllers) {
            this._events.set(type, [fn]);
        } else {
            handlers.push(fn);
            this._events.set(type, handlers);
        }
    }

    removeListener(type, fn) {
        const handlers = this._events.get(type);
        if (!handlers) {
            return;
        }
        let position = -1;
        for (let i = 0; i < handler.length; i++) {
            if (handers[i] === fn) {
                position = i;
                break;
            }
        }
        if (position !== -1) {
            handlers.splice(position, 1);
        }
    }
}
```
