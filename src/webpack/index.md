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
  // 模块加载方法
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