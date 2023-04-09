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

## js引擎对js处理过程
1. 读取代码，进行词法分析，代码分解成词元（token）
2. 对词元进行语法分析，整理成语法树
3. 使用翻译器，将代码转为字节码
4. 使用字节码解释器，将字节码转成机器码



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

# 3. DNS解析过程？
1. 浏览器缓存
2. 寻找本地localhost，看有没有该域名对应的ip
3. 路由器缓存
4. ISP（互联网服务提供商）DNS缓存，比如用的电信网络，则进入电信的DNS缓存服务器找。
5. 根域名服务器：全球只有13台
6. 顶级域名服务器：若无则将主域名服务器ip告诉DNS
7. 主域名服务器：如果没有则进入下一级域名服务器，重复直到找到。
8. 保存结果至缓存：本地域名服务器把返回保存。将结果返回给客户端，与web服务器建立连接。

# 4. get/post
除了http层区别，在tcp/ip层面也有区别  
get请求：
```
headers + data  --->   服务器     
tcp数据包       <--- 200
```

post请求：  
```js
headers  ----->    服务器
tcp包   <------
        100 continue

data     ----->    服务器
tcp包    <----
          200
```

# 5. Event Loop
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