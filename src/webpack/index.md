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