# 面试题锦集



## 对闭包的看法，为什么要用闭包？说一下闭包原理以及应用场景

公司：滴滴、携程、喜马拉雅、微医、蘑菇街、酷家乐、腾讯应用宝、安居客

### 什么是闭包

有权访问另一个函数作用域中的变量的函数。


### 闭包原理
函数执行分成两个阶段(预编译阶段和执行阶段)。

* 在预编译阶段，如果发现内部函数使用了外部函数的变量，则会在内存中创建一个“闭包”对象并保存对应变量值，如果已存在“闭包”，则只需要增加对应属性值即可。
* 执行完后，函数执行上下文会被销毁，函数对“闭包”对象的引用也会被销毁，但其内部函数还持有该“闭包”的引用，所以内部函数可以继续使用“外部函数”中的变量

利用了**函数作用域链**的特性，一个函数内部定义的函数会将包含外部函数的**活动对象**添加到它的**作用域链**中，函数执行完毕，其执行作用域链销毁，但因内部函数的作用域链仍然在引用这个活动对象，所以其活动对象不会被销毁，直到内部函数被销毁后才被销毁。

### 优点
1. 可以从内部函数访问外部函数的作用域中的变量，且访问到的变量长期驻扎在内存中，可供之后使用
2. 避免变量污染全局
3. 把变量存到独立的作用域，作为私有成员存在


### 缺点
1. 对内存消耗有负面影响。因内部函数保存了对外部变量的引用，导致无法被垃圾回收，增大内存使用量，所以使用不当会导致内存泄漏
2. 对处理速度具有负面影响。闭包的层级决定了引用的外部变量在查找时经过的作用域链长度
3. 可能获取到意外的值(captured value)

### 应用
应用场景一： 典型应用是模块封装，在各模块规范出现之前，都是用这样的方式防止变量污染全局。

应用场景二： 在循环中创建闭包，防止取到意外的值。






## 介绍防抖节流原理、区别以及应用，并用JavaScript进行实现

公司：滴滴、虎扑、挖财、58、头条




## 介绍下 promise 的特性、优缺点，内部是如何实现的，动手实现 Promise

公司：滴滴、头条、喜马拉雅、兑吧、寺库、百分点、58、安居客


## 实现Promise.all


## 实现一个多并发请求

```js
let urls = ['http://dcdapp.com', …];
/*
	*实现一个方法，比如每次并发的执行三个请求，如果超时（timeout）就输入null，直到全部请求完
	*batchGet(urls, batchnum=3, timeout=3000);
	*urls是一个请求的数组，每一项是一个url
	*最后按照输入的顺序返回结果数组[]
*/

function delayPromise(timeout) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, timeout);
    });
}

function timeoutPromise(promise, ms) {
    const timeout = delayPromise(ms).then(() => {
        return null;
    });
    return Promise.race[timeout, promise];
}

function fetch(url) {
    return new Promise(async (resolve, reject) => {
        try {
            const res = await fetch(url);
            resolve(res);
        } catch (err) {
            reject(err);
        }
    });
}

function batchGet(urls,batchnum=3, timeout=3000 ) {
    const result = [];
    const chunkLen = Math.ceil(urls.length / batchnum);
    for (let i = 0; i < chunkLen; i++) {
        const curUrls = urls.slice(batchnum * i, batchnum * (i + 1));
        const fetchs = curUrls.map((url) => fetch(url));
        const pAll = Promise.all(fetchs);
        const curRes = timeoutPromise(pAll, timeout);
        if (!curRes) {
            return null;
        }
        result.push(...curRes);
    }
    return result;
}

```






## 字符串repeact实现?
1. 'ni'.repeact(3);
2.
```js
String.prototype.repeact = function (n) {
  return Array(n+1).join(this);
}

```
3.
```js
String.prototype.repeact = function(n) {
  return Array(n).fill(this).join('');
}
```

## 介绍模块化？
例如有3个模块，a引用了b、c，a模块打印了‘aaa’，c模块打印了‘cccc’
### 1. commonjs
用module.exports 定义当前模块对外输出接口，用require加载模块，**同步方式**加载模块。
执行:
```js
加载了a；
输出“aaa”
加载了b；
加载了c；
输出“ccc”
```
### 2. AMD和require.js （依赖前置）
用define()定义模块，require()加载模块，有依赖模块放在[]中作为第一参数
```js
define(['underscore'], function(_) {

});
require(['jquery', 'math'], function($, math) {

});

```
采用**异步加载**，模块加载不影响后续语句，等加载完才运行回调函数。
执行：
```js
加载了a、加载了b、加载模块c
才开始输出“aaa”
```
### 3. CMD和sea.js
推崇依赖就近、延迟执行
```js
defind(function(require, exports, module) {
  var a = require('a');
  a.doSomething();
})
```
require时才去加载模块，加载完再接着执行。
执行：和commonjs一样
```js
加载了a模块
输出“aaa”
加载了模块b
加载了模块c
输出“ccc”
```
### 4. es6 Module：export和import
import编译时就引入模块代码，而不是运行时加载，无法实现条件加载。
执行：和AMD一样
```js
加载了a模块；
加载了b模块；
加载了c模块；
输出“aaa”
输出“ccc”
```
### 差异
1. commonjs输出是拷贝，es6输出的是绑定
2. commonjs模块是运行时加载，es6是编译时输出接口；

## 实现一个深度克隆？
注意点：
* 复制函数
* 复制正则对象
* 抛弃对象的constructor,所有构造函数指向Object
* 栈溢出

```js
// 工具函数，对象类型判断
function isType (obj, type) {
  if (typeof obj !== 'Object') {
    return false;
  }
  const typeString = Object.prototype.toString().call(obj);
  let flag;
  switch (type) {
    case 'Array':
      flag = typeString === '[object Array]';
      break;
    case 'Date':
      flag = typeString === '[object Date]';
      break;
    case 'RegExp':
      flag = typeString === '[object RegExp]';
      break;
    default:
      flag = false;
  }
  return flag;
}

const getRegExp = (re) => {
  let flags = '';
  if (re.global) {
    flags += 'g';
  }
  if (re.ignoreCase) {
    flags += 'i';
  }
  if (re.multiline) {
    flags += 'm';
  }
  return flags;
}


function clone (parent) {
  const parents = [];
  const children = [];
  const _clone = (parent) => {
    if (parent === null) {
      return null;
    }
    if (typeof parent !== 'object') {
      return parent;
    }
    let child;
    let proto;
    if (isType(parent, 'Array')) {
      child = [];
    } else if (isType(parent, 'RegExp')) {
      child = new RegExp(parent.source, getRegExp(parent));
      if (parent.lastIndex) {
        child.lastIndex = parent.lastIndex;
      }
    } else if (isType(parent, 'Date')) {
      child = new Date(parent.getTime());
    } else {
      // 处理原型对象
      proto = Object.getPrototypeOf(parent);
      child = Object.create(proto);

    }
    // 处理循环引用
    const index = parents.indexOf(parent);
    if (index !== -1) {
      return children[index];
    }
    parents.push(parent);
    children.push(child);
    for (let i in parent) {
      child[i] = _clone(parent[i]);
    }
    return child;
  }

  return _clone(parent);

}
```


## 浅拷贝与深拷贝（只针对对象）
浅拷贝：只拷贝一层；
深拷贝：无限层级拷贝。
例如：
```js
var a1 = {b: {c: 1}};
var a2 = shadowClone(a1);
a2.b.c === a1.b.c; // true

var a3 = clone(a1);
a3.b.c === a1.b.c; // false

```
### 深拷贝问题

公司：顺丰、新东方、高德、虎扑、微医、百分点、酷狗

在进行深拷贝时，会拷贝所有的属性，并且如果这些属性是对象，也会对这些对象进行深拷贝，直到最底层的基本数据类型为止。这意味着，对于深拷贝后的对象，即使原对象的属性值发生了变化，深拷贝后的对象的属性值也不会受到影响。

* 栈溢出
* 循环引用
* clone层级很深，栈溢出，递归方式会有这个问题；

例如：
```js
var a = {
    a1: 1,
    a2: {
        b1: 1,
        b2: {
            c1: 1
        }
    }
};
```
借助栈，栈里存下一个拷贝节点，遍历当前节点下的子元素，if是对象放在栈里，or直接拷贝。

```js
function cloneLoop(x) {
    const root = {};
    // 栈
    const loopList = [{
        parent: root,
        key: undefined,
        data: x
    }];
    while(loopList.length) {
        // 深度优先
        const node = loopList.pop();
        const {parent, key, data} = node;
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }
        Object.keys(data).forEach((k) => {
            if (typeof data[k] === 'object') {
                // 下一次循环
                loopList.push({
                    parent: res,
                    key: k,
                    data: data[k]
                });
            } else {
                res[k] = data[k];
            }
        });
    }
    return root;
}
```
问题：循环引用，也是引用丢失造成的。
解决：拷贝对象前，判断是否拷贝过了，拷贝过就用原来的。

```js

function find(arr, item) {
    for (let i = 0; i < arr.length; i++) {
        if (arr[i].soruce === item) {
            return arr[i];
        }
    }
    return null;
}
function cloneForce(x) {
    const uniqueList = []; // 用来去重
    let root = {};
    const loopList = [{
        parent: root,
        key: undefined,
        data: x
    }];
    while(loopList.length) {
        const node = loopList.pop();
        const {parent} = node;
        const {key, data} = node;
        let res = parent;
        // 非首次
        if (typeof key !== undefined) {
            res = parent[key] = {};
        }
        // 数据已经存在
        let uniqueData = find(uniqueList, data);
        if (uniqueData) {
            parent[key] = uniqueData.target;
            continue; // 中断本次循环
        }
        // 数据不存在
        uniqueList.push({
            source: data,
            target: res
        });
        
        Object.keys(data).forEach((k) => {
            if (typeof data[k] === 'object') {
                // 下一次循环
                loopList.push({
                    parent: res,
                    key: k,
                    data: data[k]
                });
            } else {
                res[k] = data[k];
            }
        });
    }
    return root;
}

// 验证
var a = {};
a.a = a;
cloneForce(a); // 没报错，破解循环引用

```





## 前端注意哪些SEO?
1. 合理的title、description、keywords：搜索对这三项权重逐个减少
2. 语义化的HTML代码
3. 少用iframe：搜索引擎不会抓取iframe中的内容
4. 非装饰性图片必须加alt
5. 提高网站速度：网站速度是搜索引擎排序重要指标
6. 重要的内容不要用js输出


## 图片懒加载？
利用最新API: IntersectionObserver
new IntersectionObserver(callback, options)

```js
// <img data-src="xxx">
function query(tag) {
  return Array.from(document.getElementsByTagName(tag));
}

const observer = new IntersectionObserver((changes) => {
  changes.forEach((change) => {
    // 可见比例
    if (change.intersectionRatio >0) {
      const img = change.target;
      img.src = img.dataset.src;
      // 不再监听
      observer.unobserve(img);
    }
  });
});
query('img').forEach((item) => observer.observe(item));
```


## 实现大文件上传 ？
核心利用Blob.prototype.slice，每个切片顺序，并发传。
服务端收到再合并切片。
伪代码

```html
<input type="file" @change="handleFileChange" />
```

```js
const container = {
    file: null
};

const handleFileChange = (e) => {
    const [file] = e.target.files;
    if (!file) return;
    container.file = file;
};
const request = ({
    url,
    method = "post",
    data,
    headers = {}
}) => {
    return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
        xhr.open(method, url);
        Object.keys(headers).forEach(key => {
            xhr.setRequestHeader(key, headers[key]);
        });
        xhr.send(data);
        xhr.onload = e => {
            resolve({
                data: e.target.response
            });
        };
    });
}

const SIZE = 10 * 1024 * 1024; // 切片大小
// 生成文件切片
const createFileChunk = (file, size = SIZE) => {
    const fileChunkList = [];
    let cur = 0;
    while (cur < file.size) {
        fileChunkList.push({
            file: file.slice(cur, cur + size);
        });
        cur += size;
    }
    return fileChunkList;
}

const mergeRequest = async() => {
    await request({
        url: 'http://localhost:3000/merge',
        headers: {
            "content-type": "application/json"
        },
        data: JSON.stringify({
            fileName: container.file.name
        })
    });
};
// 上传切片
const uploadChunks = async (data) => {
    const requestList = data.map(({chunk, hash}) => {
        const formData = new FormData();
        formData.append("chunk", chunk);
        formData.append("hash", hash);
        formData.append("fileName", container.file.name);
        return {
            formData
        };
    })
    .map(async({formData}) =>
        request({
            url: 'http://localhost:3000',
            data: fromData
        });
    );
    await Promise.all(requestList);
    await mergeRquest();
};

// 提交表单
const handleUpload = async () => {
    if (!container.file) return;
    const fileChunkList = createFileChunk(container.file);
    const data = fileChunkList.map(({file}, index) => ({
        chunk: file,
        hash: container.file.name + '_' + index
    }));
    await uploadChunks(data);

};


```

服务端代码：
```js
const http = request("http");
const path = require("path");
const fse = require("fs-extra");
const server = http.createServer();
const UPLOAD_DIR = path.resolve(__dirname, "..", "target"); // 目录
const resolvePost = req =>
    new Promise(resolve => {
        let chunk = "";
        req.on("data", data => {
            chunk += data;
        });
        req.on("end", () => {
            resolve(JSON.parse(chunk));
        });
    });

const pipeStream = (path, writeStream) =>   new Promise(resolve => {
    const readStream = fse.createReadStream(path);
    // 结束后同步删除文件
    readStream.on("end", ()=> {
        fse.unlinkSync(path);
        resolve();
    });
    readStream.pipe(writeStream);
});
// 合并切片
const mergeFileChunk = async(filePath, filename, size) => {
    const chunkDir = path.resolve(UPLOAD_DIR, filename);
    const chunkPaths = await fse.readdir(chunkDir);
    // 根据切片排序
    chunksPaths.sort((a, b) => a.split("_")[1] - b.split("_")[1]);
    await Promise.all(
        chunkPaths.map((chunkPath, index) => pipeStream(
            path.resolve(chunkDir, chunkPath),
            // 指定位置创建可写流
            fse.createWirteStream(filePath, {
                start: index * size,
                end: (index + 1) * size
            });
        ))
    );
    // 同步删除目录
    fse.rmdirSync(chunkDir);
};
server.on("request", async(req, res) => {
    res.setHeader("Access-Control-Allow-Origin", "*");
    res.setHeader("Access-Control-Allow-Headers", "*");
    if (req.method === "OPTIONS") {
        res.status = 200;
        res.end();
        return;
    }
    if (req.url === '/merge') {
        const data = await resolvePost(req);
        const {filename, size} = data;
        const filePath = path.resolve(UPLOAD_DIR, `${filename}`);
        await mergeFileChunk(filePath, filename, size);
        res.end(JSON.stringify({
            code: 0,
            message: "file merged success"
        }));
    }
});

server.listen(3000, ()=> console.log("3000端口"));



```





## 实现add(1)(2)(3)


```js
const add = (a: number, b: number, c: number) => a + b + c;


//参数确定
const curry = (fn: Function) => {
  let args = [];

  return function temp(...newArgs) {
    args.push(...newArgs);
    if (args.length === fn.length) {
      const val = fn.apply(this, args);
      args = [];
      return val;
    } else {
      return temp;
    }
  };
};

//参数不确定
const currying = (fn: Function) => {
  let args = [];

  return function temp(...newArgs) {
    if (newArgs.length) {
      args.push(...newArgs);
      return temp;
    } else {
      const val = fn.apply(this, args);
      args = [];
      return val;
    }
  };
};

const curryAdd = curry(add);
console.log(curryAdd(1)(2)(3)); // 6
console.log(curryAdd(1, 2)(3)); // 6
console.log(curryAdd(1)(2, 3)); // 6


```
