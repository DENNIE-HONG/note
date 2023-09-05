# webpack5

## 变动
### 1. 不再尾Nodejs模块自动引用polyfills
模块引用了如cypto, webpack <= 4 自动引入polyfills


### 2. 长期缓存
用了新算法，生产环境默认开启。 
optimization.chunkIds: "deterministic" (2字符)
optimization.moduleIds: "deterministic" (3-5数字)

真正的内容hash，例如：只改变了注释/变量名字，也不会变。

### 3. 开发环境支持
开发环境默认使用了新的named chunkId算法

### 4. 支持新的web平台特性
支持资源模块化

### 5. 开发体验
5.1 target，以前只有"web" or "node",
    现在：["web", "es2020"]等， 可以有组合
5.2 自动唯一命名
    多个webpack runtime在一个页面，自动读取package.name作为唯一的bundle命名，而不用output.uniqueName
5.3 自动添加public path.

### 6.构建优化
1. 更好的tree shaking
例如：

```js
// inner.js
export const a = "aa";
export const b = "bb";


// module.js
import * as inner from "./inner.js";
export {inner};

// index.js
import * as module from "./module";
console.log(module.inner.a);

// b识别为unused, 不再打包
```

2. inner模块tree-shaking
optimization.innerGraph(生产自动开发)

3. commonjs的tree-shaking

4. 生产、开发模式默认都开启了sideEffects优化，避免一些错误的tree-shaking只发生在生产模式。
例如：

```js
import "xx/main.css";
// 以前生产模式会抖掉
```
5. 改进的target配置
可用browserlist配置，可多种组合

6. splitChunks 和 moduleSizes

例如：
```js
optimization: {
    splitChunks: {
        miniSize: {
            javascript: 3000,
            webassembly: 50000
        }
    }
}
```

### 7. 性能优化
持久（长效）缓存。
