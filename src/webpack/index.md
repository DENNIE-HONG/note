# webpack

## tree-shaking（摇树）
是无用代码移除（DCE, dead code elimination）的一个方法。找出需要的代码，灌入最终的结果。都是依赖ES6 modules的静态特性才得以实现。
静态特性：
* 模块顶层，不能出现在if/function里；
* import的模块名只能是字符串常量；
* 模块初始化时候所有import必须导入完成；
* import binding 是immutable的，类似const

## modules
打包后文件内容"框架"：
```js
(function(modules) {
  var installedModules = {};
  // 模块加载方法, 做了缓存优化
  function _webpack_require_(moduleId) {
    if (installedMudules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      i:moduleId,
      l: false,
      exports: {}
    };
    // 通过模块名找对应模块函数并执行
    modules[moduleId].call(module.exports, module, module.exports, _webpack_require_);
    module.l = true;
    return module.exports;
  }
  return _webpack_require_("./src/xx.js");
})({
  "./src/xx.js": (function(module, exports) {
      eval(...)
  })
})
```

## 原理
本质上是一种事件机制，工作流程即将各个插件串联起来，通过Tapable。
* 负责编译：Complier,包含了webpack环境所有配置信息，全局唯一。整个webpack从启动到关闭的生命周期。
* 负责创建bundles:Compilation，包含了当前模块资源、编译生成资源、变化的文件等。开发模式时，检测到一个文件变化，一个新的compliation被创建。



## loader与plugin？
webpack是一个模块打包器，可以用loader和plugin扩展，基于tapable的插件架构。


### loader
loader是一个node模块，导出一个函数，获得处理前的原内容，对原内容执行处理后，返回处理后的内容。
多个loader串联使用，执行顺序是最后一个到第一个：
* 最后loader首先调用，参数是source；
* 第一个loader最后调用，返回js代码；
* 中间的loader链式调用；

例如html-minify-loader:

```js
var loaderUtils = require('loader-utils');
// 可获取loader的options
var Minimize = require('minimize');
module.exports = function(source) {
    var options = loaderUtils.getOptions(this) || {};
    var minimize = new Minimize(options);
    return minimize.parse(source);
};
```





### plugin
在webpack运行的生命周期中会广播出许多事件，plugin可以监听这些事，在合适的时机通过webpack提供的API改变输出结果。

```js
class BasicPlugin {
  constructor(options) {
    ...
  }
  // webpack会调用BasicPlugin实例的apply方法
  // 给插件实例传入complier对象
  apply(complier) {
    complier.plugin('compilation', function(complilation) {
      ...
    });
  }
}
module.exports = BasicPlugin;
```
1. 执行new BasicPlugin(options)初始化一个BasicPlugin获得实例；
2. 调用apply给插件实例传入complier对象；
3. 通过complier.plugin(事件名，回调)监听webpack广播事件，通过complier对象操作webpack；
* complier: 包含了options、loaders、plugins、全局唯一，webpack实例；
* complilation:包含当前模块资源、编译生成资源、变化的文件等，开发环境时，一个文件变化 => 一次新的complilation被创建。


webpack通过Tapable的事件流机制，插件只需监听它所关心的事件。
```js
// 广播出事件
complier.apply('event-name', params);
// 监听名为event-name的事件，事件发生触发回调
complier.plugin('event-name', function(params) {});
```

在开发插件是经常需要知道是哪个文件发生了变化导致了新的complilation。
```js
compiler.plugin('watch-run', (watching, callback) => {
  // 获取发生变化的文件列表
  const changedFiles = watching.Complier.watchFilesSystem.watcher.mtimes;
  if (changedFiles[filePaht] !== undefined) {
    // filePath对应的文件发生了变化
  }
  callback();
});
```


## Tapable机制

webpack本质是一种事件流机制，核心是Tapable。
Tapable：
* complier 编译
* compilation 创建bundles

Tapbale 1.0:
* Sync
    * SyncHook
    * SyncBailHook
    * SyncWaterfailHook
    * SyncLoopHook
* Async
    * AsyncParaller
    * AsyncSeries

即暴露出很多钩子
hook伪代码：
```js
class Hook {
    construtor(args) {
        if (!Array.isArray(args)) args = [];
        // 实例钩子的string变数组
        this._args = args;
        // 消费者
        this.taps = [];
        this.interceptors = [];
        // 以sync类型方式来调用钩子
        this.call = this._call = this._createCompileDelegate("call", "sync");
        this.promise = this._promise = this._createCompileDelegate("promies", "promise");
        this.callAsync = this._callAsync = this._createCompileDelegate("callAsync", "async");
        this._x = undefined;
    }
    _createCall(type) {
        return this.compile({
            taps: this.taps,
            interceptors: this.interceptors,
            args: this._args,
            type
        });
    }
    _createCompileDelegate(name, type) {
        const lazyCompileHook = (...args) => {
            this[name] = this._createCall(type);
            return this[name](...args);
        };
        return lazyCompileHook;
    }
    // 调用tap类型注册
    tap (options, fn) {
        //...
        options = Object.assign({
            type: 'sync',
            fn
        }, options);
        // ...
        this._insert(options);
    }
    // 注册async类型的钩子
    tapAsync(options, fn) {
        //...
        options = Object.assign({
            type: "async",
            fn
        }, options);
        // ...
        this._insert(options);
    }
    // 注册promise类型钩子
    tapPromise(options, fn) {
        //...
        options = Object.assign({
            type: 'promise',
            fn
        }, options);
        this._insert(options);
    }
}
```

每次都是调用tap、tapSync、tapPromise注册不同类型的插件钩子，通过调用call、callAsync、promise方式调用。
按照一定的执行策略执行，调用compile方法快读编译出一个方法来执行插件。


### 1. sync类型的钩子：
插件只按顺序执行；只注册tap;

伪代码：
```js
const fatory = new Sync*CodeFactory();
class Sync* extends Hook {
    tapAsync() {
        throw new Error('topAsync is not supported on a Sync *');
    }
    tapPromise() {
        throw new Error("tapPromise is not supported on a Sync *");
    }
    // 编译按一定策略执行Plugin
    compile(options) {
        factory.setup(this, options);
        return factory.create(options);
    }
}
```

### 2. Async* 类型钩子支持tap、tapPromise、tapAsync注册
伪代码；
```js
class AsyncParallerHook extends Hook {
    construtor(args) {
        super(args);
        this.call = this._call = undefined;
    }
    compile(options) {
        factory.setup(this, options);
        return factory.create(options);
    }
}
```

### 3. 工厂函数Sync*CodeFactory
伪代码：
```js
class Sync*CodeFactory extends HookCodeFactory {
    construtor({onError, onResult, onDone, throwIfProssible}) {
        return this.callTapsSeries({
            onError: (i, err) => onError(err);
            onDone,
            throwIfPossible
        });
    }
}
class HookCodeFactory {
    construtor(ccofig) {
        this.config = config;
        this.options = undefined;
    }
    create(options) {
        this.init(options);
        switch(this.options.type) {
            // 结果直接返回
            case "sync":
                return new Function(this.args(), "\"use strict\"; \n" + this.header() + this.content({
                    //...
                    onResult: result => `return ${result};\n`,
                    // ...
                }));
            // async类型，异步执行，将调用插件执行结果来调用callback
            case "async":
                return new Function(this.args({
                    after: "_callback"
                }), "\"use strict\", \n" + this.header() + this.content({
                    //...
                    onResult: result => `_callback(null, ${result});\n`,
                    onDone: () => "_callbck();\n"
                }));
            // 返回promise类型，将结果放在resolve中
            case "promise":
                //...
                code += "return new Promise((_resolve, _reject) => {\n";
                code += "var _sync = true;\n";
                code += this.header();
                code += this.content({
                    //...
                    onResult: result => `_resolve(${result}), \n`,
                    onDone: () => "_resolve();\n"
                });
                // ...
                return new Function(this.args(), code);
        }
    }
    // callTap执行插件，并将结果返回
    callTap(tapIndex, {onError, onResult, onDone, rethrowIfPossible}) {
        let code = "";
        let hasTapcached = false;
        // ...
        code += `var _fn${tapIndex}=${this.getTapFn(tapIndex)};\n`;
        const tap = this.options.taps[tapIndex];
        switch(tap.type) {
            case "sync":
                // ...
                if (onResult) {
                    code += `var _result${tapIndex}=_fn${tapIndex}(${this.args({
                        before: tap.context ? "_context" : undefined
                    })}); \n`;
                } else {
                    code += `_fn${tapIndex}(${this.args({
                        before: tap.context ? "_context": undefined
                    })});\n`;
                }
                if (onResult) {
                    code += onResult(`_result${tapIndex}`);
                }
                // 通知插件执行完毕，可执行下一个插件
                if (onDone) {
                    code += onDone();
                }
                break;
            // 异步执行，插件运行完再将结果通过执行callback透传
            case "async":
                let cbCode = "";
                if (onResult) {
                    cbCode += `(_err${tapIndex}, _result${tapIndex}) => {\n`;
                } else {
                    cbCode += `_err${tapIndex} =>{\n`;
                }
                cbCode += `if(_err${tapIndex}) { \n`;
                cbCode += onError(`_err${tapIndex}`);
                cbCode += "} else {\n";
                if (onResult) {
                    cbCode += onResult(`_result${tapIndex}`);
                }
                cbCode += "}\n";
                cbCode += "}";
                code += `_fn${tapIndex}(${this.args({
                    before: tap.context ? "_context": undefined,
                    after: cbCodde // cbCode将结果透传
                })});\n`;
                break;
            case "promise":
                ...
        }
        return code;
    }

    // 按插件注册顺序，递归调用执行插件
    callTapSeries({onError, onResult, onDone, rethrowIfPossible}) {
        // ...
        const firstAsync = this.options.taps.findIndex(t => t.type !== "sync");
        const next = i => {
            // ...
            const done = () => next(i+1);
            // ...
            return this.callTap(i, {
                // ...
                onResult: onResult && ((result) => {
                    return onResult(i, result, done, doneBreak);
                });
                // ...

            });
        };
        return next(0);
    }
    // 并行调用插件执行
    callTapParalllel({onError, onResult, onDone, rethrowIfPossible, onTap=(i, run) => run()}) {
        // 遍历注册所有插件，并调用
        for (let i = 0; i < this.options.taps.length; i++) {
            // ...
            code += "if (_counter <= 0) break;\n";
            code += onTap(i, 1) => this.callTap(i, {
                // ...
                onResult: onResult && ((result) => {
                    let code = "";
                    code += "if (_counter > 0) { \n";
                    code += onResult(i, result, done, doneBreak);
                    code += "}\n";
                    return code;
                }),
                // ...
            }), done, doneBreak);
        }
        // ...
        return code;
    }
}
```

### syncBailHook
syncBailHook中一旦某个返回结果不为undefined,结束执行列表中插件

```js
class SyncBailHookCodeFactory extends HookCodeFactory {
    content({onError, onResult, onDone, rethrowIfPossible}) {
        return this.callTapsSeries({
            // ...
            onResult: (i, result, next) => `if (${result} !== undefined {\n
                ${onResult(result)};\n} else {\n
                    ${next(i)}}\n`,
            // ...
        });
    }
}
```

### 调用
```js
const {
    SyncHook,
    SyncBailHook,
    SyncWaterfailHook,
    ...
    AsyncParallelHook,
    AsyncSeriesHook,
    ...
} = requiere("tapable");
class Order {
    construtor() {
        this.hooks = {
            goods: new SyncHook(["goodsId", 'number']),
            consumer: new AsyncParallelHook(['userId', 'orderId'])
        }
    }

    queryGoods(goodsId, number) {
        this.hooks.goods.call(goodsId, number);
    }
    consumerInfoPromise(userId, orderId) {
        this.hooks.consumer.promise(userId, orderId)
            .then(() => {
                // todo
            });
    }
    consumerInfoAsync(userId, ordereId) {
        this.hooks.consumer.callAsync(userId, orderId, (err, data) => {
            // todo
        });
    }
}
// 调用tap方法注册一个consumer
order.hooks.goods.tap('QueryPlugin', (goodsId, number) => {
    return fetchGoods(goodsId, number);
});
order.hooks.goods.tap('LoggerPlugin', (goodsId, number) => {
    logger(goodsId, number);
});

// 调用
order.queryGoods('10000', 1);
```

通过tap添加消费者，通过call来触发钩子的顺序执行。
```js
order.hooks.onsumer.tapAsync('LoginCheckPlugin', (userId, orderId, callback) => {
    LoginCheck(usesrId, callback);
});
```
Tapable用法：
* 插件注册数量
* 插件注册类型(sync, async, promise)
* 调用方式(sync, async, promise)
