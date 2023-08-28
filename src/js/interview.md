# 面试题锦集
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
        const parent = node.parent;
        const key = node.key;
        const data = node.data;
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }
        for (let k in data) {
            if (data.hasOwnProperty(k)) {
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
            }
        }
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
        for (let k in data) {
            if (data.hasOwnProperty(k)) {
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
            }
        }
    }
    return root;
}

// 验证
var a = {};
a.a = a;
cloneForce(a); // 没报错，破解循环引用

```




## 如何实现Event库(发布订阅模式)
```js
class EventEmeitter {
    constructor() {
        this._events = this._events || new Map();
        this._maxListeners = this._maxListeners || 10;
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
