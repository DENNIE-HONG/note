# 1. v8引擎
## 垃圾回收机制分代式垃圾回收机制。 
* 新生代： --max-new-space-size 32M/16M
* 老生代： --max-old-space-size 1.4G/0.7G
### 新生代：scavenge算法
1. Form空间分配对象
2. 回收时，存活对象-> To空间，非存活对象释放
3. Form空间 <=> To空间，存活对象呼唤 

缺点:只能使用堆内存的一半；  
优点：只复制存活对象，效率高，而新生代对象生命周期短，故而适合。
在新生代多次复制后仍存活，*晋升*为老生代

### 老生代：标记清除和标记整理相结合
标记清除：标记存活的对象，没被标记的清除。  
标记整理：标记后，将存活对象移动，清理边界外内存。  
缺点： 内存碎片  
v8主要用标记清除，空间不足时才使用标记整理。

### js字符串拼接性能很差？
因为字符串是不可变的，只能被另外一个字符串代替。字符串拼接新建一个临时对象存储计算结果，然后再用临时对象替换变量。若有海量字符串拼接，js引擎GC需要大量工作清理临时变量，从而影响性能。

## js引擎对js处理过程
1. 读取代码，进行词法分析，代码分解成词元（token）
2. 对词元进行语法分析，整理成语法树
3. 使用翻译器，将代码转为字节码
4. 使用字节码解释器，将字节码转成机器码

## 结构
```js
call stack （空间小） memory heep（大）
 _____             __________
| 栈  |            |  堆     | 
|     |  boolean   |         | 引用数据类型
|     |  number    |         |
|_____|  ...       |_________|
           
```
闭包函数里的变量存储？
* captured variables：即闭包中访问的外部变量，基本类型变量在闭包中也是存在堆中的；
* local variables：栈中；

# 2. 缓存
## localStorage
基于LocalStorage设计一个1M的缓存系统，实现缓存淘汰机制？  
思路：
1. 过期时间  
2. 所占空间大小  
3. 是否过期

```js
class Store {
  constructor () {
    const store = localStorage.getItem('cache');
    if (!store) {
      store = {
        maxSize: 1024 * 1024,
        size: 0
      }
    }
    this.store = store;
  }
  set(key, value, expire) {
    this.store[key] = {
      date: Date.now(),
      expire
    };

    const size = sizeof(JSON.stringify(value));
    if (this.store.maxSize < size + this.store.size) {
      console.log('超了');
      let keys = Object.keys(this.store);
      // 时间排序
      keys = keys.sort((a, b) => {
        const item1 = this.store[a];
        const item2 = this.store[b];
        return item2.date - item1.date;
      });
      while(size + this.store.size > this.store.maxSize) {
        delete this.store[keys[keys.length -1]];
      }
    }
    this.store.size += size;
    this.store[key] = value;
    localStorage.setItem('cache', this.store);
  }

  get(key) {
    let d = this.store[key];
    if (!d) {
      console.log('not Finded');
      return;
    }
    if (d.expire > Date.now) {
      console.log('过期删除');
      delete this.store[key];
      localStorage.setItem('cache', this.store);
    } else {
      return d;
    }
  }

  sizeof (str, charset) {
    let total = 0;
    let charCode;
    let i;
    let len;
    charset = charset ? charset.toLowerCase(): '';
    //
  }
}
```

## cookie
cookie优化：  
客户端在域名A下有cookie,页面依赖很多静态资源，静态资源会默认带上cookie，造成浪费。
解决：  
多域名拆分，将静态资源分组，放到不同域名下

## sessionStorage/localStorage/cookie区别？
1. 都会在浏览器保存，有大小限制，同源限制；
2. cookie在请求时发送到服务器，作为会话标识，服务器可修改cookie，webstorage不会发送到服务器；
3. cookie有path概念，子路径可访问父路径cookie，父路径不能访问子；
4. 有效期：
* cookie在设置有效期内有效，默认为浏览器关闭；
* sessionStorage在窗口关闭前有效；
* localStorage长期有效，直到用户删除；
5. 共享：
  * sessionStorage不能共享；
  * localStorage在同源文档中间共享；
  * ciikie同源且符合path规则的文档之间共享；
6. localStorage的修改会促发其他文档窗口的update事件；
7. cookie有sercure属性要求HTTPS传输；
8. 浏览器不能保存超过300个cookie，单个服务器不超过20个，每个cookie不超4k，web storage支持达到5M;




# 3. DNS解析过程？
1. 浏览器缓存
2. 寻找本地localhost，看有没有该域名对应的ip
3. 路由器缓存
4. ISP（互联网服务提供商）DNS缓存，比如用的电信网络，则进入电信的DNS缓存服务器找。
5. 根域名服务器：全球只有13台
6. 顶级域名服务器：若无则将主域名服务器ip告诉DNS
7. 主域名服务器：如果没有则进入下一级域名服务器，重复直到找到。
8. 保存结果至缓存：本地域名服务器把返回保存。将结果返回给客户端，与web服务器建立连接。


# 4. Event Loop
js在执行中产生执行环境，会被顺序加入到执行栈中。遇到异步代码，会被挂起并加入到Task(有多种task)。队列中，一旦执行栈为空，Event Loop从Task队列中拿出需要执行的代码并放入执行栈中执行。  
任务源：  
* 微任务  
* 宏任务  

不同任务源会被分配到不同的task队列中。  
微任务：  
* process.nextTick  
* promise  
* Object.observe
* MutationObserver  

宏任务：  
* script
* setTimeout 
* setInterval
* setImmediate
* I/O
* UI rendering

Event Loop顺序:
1. 执行同步代码，这属于宏任务；
2. 执行栈为空，查询是否有微任务需要执行
3. 执行所有微任务
4. 必要的话渲染UI
5. 下一轮Event Loop, 执行宏任务中的异步代码。

# 5.执行时间线
1. 创建Document对象 -> 解析HTML元素 -> 添加Element对象和Text节点，
document.readystate: loading.
2. 遇到\<script\>执行行内or外部脚本，脚本同步执行，脚本下载时解析器暂停；
3. 遇到async的\<script\> -> 开始下载脚本并继续解析文档，脚本会下载完后尽快执行，但解析器不会停下等它下载；
4. 文档完成解析，document.readystate:interactive;
5. 所有defer的\<script\>按照文档出现顺序执行脚本；
6. 触发DOMContentLoaded事件 -> 异步事件驱动；
7. 等待图片载入  -> 异步脚本载入与执行 -> 触发load事件，document.readystate: complete;
8. 调用异步事件：用户输入等；

# 6. 浏览器渲染过程

```js
 _________________________________________________
| JavaScript | Style | Layout | Paint | Composite |
 —————————————————————————————————————————————————
```
* JavaScript: 执行js来触发视觉效果；
* Style: 计算元素匹配的css选择器，应用各规则计算元素最终样式；
* Layout: 根据样式，计算元素占据空间大小和位置；
* Paint: 填充可视部分，文本、图像、边框、阴影；
* Composite: 不同层按顺序绘制到屏幕上；


# 7. 跨域
跨域解决方案：
## 方案1： jsonp
```html
<script>
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = 'http://xxx?user=admin&callback=onBack';
  document.head.appendChild(script);

  // 回调执行
  function onBack(res) {
    alert(JSON.stringfy(res));
  }
</script>
```
缺点：只能实现get请求。

## 方案2：document.domain + iframe
父窗口：
```html
<iframe id="iframe" src="htp://child.domain.com/b.html"></iframe>
<script>
  document.domain = 'domain.com';
  var user = 'admin';
</script>
```
子窗口：
```html
<script>
  document.domain = 'domain.com';
  alert(window.parent.user);
</script>
```
缺点：仅限主域相同，子域不同的应用场景。

### 方案3：location.hash + iframe
a域与b域相互通信，通过中间页c来实现，不同域利用iframe的location.hash传值，相同域之间js访问。
1. a.html(domain1.com)
```html
<iframe id="iframe" src="http://domain2.com/b.html" style="display:none"></iframe>
<script>
  var iframe = document.getElementById("iframe");
  // 向b传hash值
  setTimeout(() => {
    iframe.src = iframe.src + '#user=admin';
  }, 1000);
  // 开放给同域c.html回调
  function onCallback(res) {
    alert(res);
  }
</script>
```

2. b.html(domain2.com)
```html
<iframe id="iframe" src="http://domain1.com/c.html" style="display:none;"></iframe>
<script>
  var iframe = document.getElementByiId("iframe");
  // 监听a.html传来的hash值，再传给c.html
  window.onhashchange = function() {
    iframe.src = iframe.src + location.hash;
  }
</script>
```

3. c.html(domain1.com)
```html
<script>
  // 监听b.html传来的hash值
  window.onhashchange = function() {
    // 同域a.html的js回调
    window.parent.parent.onCallback('hello' + location.hash.replace('#user=', ''));
  };
</script>
```

### 方案4：window.name + iframe
window.name值在不同页面加载后依旧存在，可支持（2M）的值。  
1. a.html(domain1.com/a.html)
```js
var proxy = function(url, callback) {
  var state = 0;
  var iframe = document.createElement('iframe');
  // 加载跨域页面
  iframe.src = url;
  // onload事件触发2次，第一次加载跨域页面，并留存数据window.name
  iframe.onload = function() {
    if (state === 1) {
      // 第二次onload（同域proxy）成功后，读取同域window.name中数据
      callback(iframe.contentWindow.name);
      destroyFrame();
    } else {
      // 第一次onload成功，切换到同域代理页
      iframe.conentWindow.location = 'http://domain1.com/proxy.html';
      state = 1;
    }
  }
  document.body.appendChild(iframe);
  // 获取数据后销毁
  function destroyFrame() {
    iframe.contentWindow.document.write('');
    iframe.contentWindow.close();
    document.body.removeChild(iframe);
  }
}

// 执行
proxy('http://domain2.com/b.html', function(data) {
  alert(data);
});
```

2. proxy.html
内容为空
3. b.html(domain2.com/b.html)
```html
<script>
  window.name = "This is domain2.com";
</script>
```

### 方案5：postMessage
1) 页面和其打开的新窗口的数据传递  
2) 多窗口之间消息传递
3) 页面与嵌套的iframe消息传递

1. a.html(domain1/a.html)
```html
<iframe id="iframe" src="http://domain2.com/b.html"></iframe>
<script>
  var iframe = document.getElementById("iframe");
  iframe.onload = function() {
    var data = {
      name: 'aym'
    };
    // 向domain2传数据
    iframe.contentWindow.postMessage(JSON.stringify(data), 'http://domain2.com');
  };
  // 接收domain2返回数据
  window.addEventListener('message', function(e) {
    console.log(e.data);
  }, false);
</script>
```

2. b.html(domain2.com/b.html)
```html
<script>
  // 接收域名1的数据
  window.addEventListener('message', function(e) {
    console.log(e.data);
     var data = JSON.parse(e.data);
     if (data) {
       data.number = 16;
       // 处理后发回域名
       window.parent.postMessage(JSON.stringify(data), 'http://domain1.com');
     }
  }, false);

</script>
```
