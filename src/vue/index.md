# Vue2.0（早期的）
## 1 Vue的双向绑定
发布订阅模式 + Object.defineProperty  

简单伪代码：
1. 数据监听器Observer，对数据对象所有属性监听，有变化通知订阅者
2. Watcher订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图。 
```js
  let uid = 0；
  // 消息管理员
  class Dep {
    constructor () {
      this.id = uid ++; // 区分新的watcher和只改变值产生的watcher
      this.subs = [];
    }
    // 触发target上的watcher中的addDep方法
    depend() {
      Dep.target.addDep(this);
    }
    // 添加订阅者
    addSub(sub) {
      this.subs.push(sub);
    }
    // 通知所有订阅者，触发订阅者相应逻辑处理
    notify() {
      this.subs.forEach((sub) => {
        sub.update();
      })
    }
  }
  Dep.target = null;

  function observe(value) {
    if (!value || typeof value !== 'object') {
      return;
    }
    return new Observe(value);
  }

  function defineReative(obj, key, val) {
    const dep = new Dep();
    // 给当前属性添加监听
    let childObj = observe(val);
    Object.defineProperty(obj, key, {
      emumerable: true,
      configurable: true,
      get: () => {
        // 如果有target，将其添加到dep实例的subs中
        // watcher实例化会读data属性，触发get方法
        if (Dep.target) {
          dep.depend();
        }
        return val;
      }
      set: (newVal) => {
        if (val === newVal) {
          return;
        }
        val = newVal;
        // 对新值监听
        childObj = observe(newVal);
        dep.notify();
      }
    })
  }
  // 监听者
  class Observer {
    constructor(value) {
      this.value = value;
      this.walk(value);
    }
    // 遍历属性并监听
    walk(value) {
      Object.keys(value).forEach((key) => {
        this.convert(key, value[key]);
      });
    }
    // 执行监听的具体方法
    convert(key, val) {
      defineReative(this.value, key, val);
    }
  }
  // 订阅者
  class Watcher {
    constructor(vm, expOrFn, cb) {
      this.depIds = {}; // 存订阅者id
      this.vm = vm;
      this.cb = cb;
      this.expOrFn = expOrFn; // 被订阅数据
      this.val = this.get(); // 维护更新数据
    }

    addDep(dep) {
      // 避免同id的watcher被多次存储
      if (!this.depIds.hasOwnProperty(dep.id)) {
        dep.addSub(this);
        this.depIds[dep.id] = dep;
      }
    }

    update () {
      const oldValue = this.val;
      this.val = this.get();
      this.cb.call(this.vm, this.val, oldValue);
    }

    get () {
      // 当前watcher被订阅数据更新后值，通知dep
      // 收集当前订阅者
      Dep.target = this;
      const val = this.vm._data[this.expOrFn];
      // 置空，用于下一个watcher调用
      Dep.target = null;
      return val;
    }
  }
```

### Array怎么进行侦测？
原理：继承原型，对继承后对象使用Object.defineProperty做拦截操作。
简要代码：
```js
class Observe {
  constructor(value) {
    ...
    if (Array.isArray(value)) {
      // 不想直接修改Array.prototype，只希望data的Array生效
      value._proto_ = arrayMethods;
      this.observeArray(value);
      
    } else {
      ...
    }
    ...
    const arrayProto = Array.prototype;
    const arrayMethods = Object.create(arrayProto);
    ['psuh', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(function(method) {
      const original = arrayProto[method];
      def(arrayMethods, method, function mutator(...args) {
        const result = original.apply(this, args);
        const ob = this.__ob__;
        let inserted;
        // 3种会新增数组
        switch(method) {
          case 'push':
          case 'unshift':
            inserted = args;
            break;
          case 'splice':
            inserted = args.slice(2);
            break;
        }
        
        // 将新增元素也转换成被侦测数据
        if (inserted) {
          ob.observeArray(inserted);
        }
        // 调用实例发送通知
        ob.dep.notify();
        return result;
      });
    });
  }
}
```



## 2 Diff
主要策略：  
1. 按tree层级diff  
新旧节点树之间按层级进行diff得到差异，而非传统按照深度遍历搜索；复杂度o(n^2) => o(n)
2. 按类型进行diff  
只对相同类型的同一个节点diff  
类型改变？直接建新虚拟dom，替代原来
3. 列表diff  
更新：插入、移动、删除，设置key，调整diff更新排序，没有key只能按顺序进行对比。当key唯一稳定，diff的效率提高。

```js
 ———————————————
|被订阅者sertter|
|______________|
      |
 ______________
|Dep.notify    |
 ——————————————
      | 
 _______________________
|patch(oldVnode, Vnode) |   
 ———————————————————————     
         __|___________
    ————| isSameVnode ?|————
    |    ———————————————    |    
 ___|____              _____|____
|replace |            |patchVnode|
 ————————              ——————————
 ____|________           |
| return Vnode|          |
 —————————————           |———oldVnode有子节点
                         |
                         |———oldVnode没有子节点
                         |
                         |———都只有文本节点
                         |
                         |———都有子节点

```
### 伪代码
```js

function sameVnode(a, b) {
  return a.key === b.key &&
    a.tag === b.tag &&
    a.isComment === b.isComment &&
    // 是否都绑定了data
    isDef(a.data) === isDef(b.data) &&
    sameInputType(a, b); // 当input，type要相同
}

function patchVnode(oldVnode, vnode) {
  const el = vnode.el = oldVnode.el;
  let i;
  let oldCh = oldVnode.children;
  let ch = vnode.children;
  // 都是文本节点
  if (oldVnode.text !== null && vnode.text !== null
  && oldVnode.text !== vnode.text) {
    api.setTextContent(el, vnode.text);
  } else {
    updateEle(el, vnode, oldVnode);
    // 都有子节点
    if (oldCh && ch && oldCh !== ch) {
      updateChildren(el, oldCh, ch);
      // 只有新节点
    } else if (ch) {
      createEle(vnode);
      // 只有老节点
    } else if (oldCh) {
      api.removeChild(el, oldCh);
    }
  }
}
function patch(oldVnode, vnode) {
  if (sameVnode(oldVnode, vnode)) {
    patchVnode(oldVnode, vnode);
  } else {
    const oEl = oldVnode.el;
    let parentEle = api.parentNode;
    // 根据vnode生成新元素
    createEle(vnode);
    if (parentEle !== null) {
      api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl));
      // 移除旧元素节点
      api.removeChild(parentEle, oldVnode.el);
    }
  }
  return vnode;
}
```
updateChildren思路；
1. 将vnode子节点ch和oldVnode子节点oldCh提取出来
2. oldCh和ch各有头尾变量startIdx和endIdx, 2个变量相比较，若都没匹配，如果设置了key，用key比较。一旦startIdx > endIdx,
表明至少有一个遍历完了，则结束。

## 3. nextTick的实现原理
1. vue用异步队列的方式来控制DOM更新和NnextTick回调先后执行；
2. 微任务因为优先级高特性，能确保队列中的微任务在一次事件循环前被执行完毕；
3. 因为兼容问题，vue不得不做了微任务向宏任务降级方案；

## 4. computed实现？
计算属性如何与属性建立依赖关系？  
属性发生变化又如何通知到计算属性？
### 伪代码
```js
// 这里开始转换data的getter、setter，原始值已存入_ob_属性中
Object.defineProperty(obj, key, {
  enumerable: true,
  configurable: true,
  get: function reativeGetter() {
    const value = getter ? getter.call(obj): val;
    // 判断是否处于依赖收集状态
    if (Dep.target) {
      // 建立关系
      dep.depend();
    }
    return value;
  },
  set: function reactiveSetter(newVal) {
    // 依赖发生变化，通知计算属性重新计算
    dep.notify();
  }
});

// 计算属性初始化
function initComputed(vm, computed) {
  ...
  // 遍历computed计算属性
  for (const key in computed) {
    ...
    // 创建watcher实例
    watchers[key] = new Watcher(vm, getter || noop, noop, computeWatchOptions);
    // 创建属性vm.计算属性, 并将提供的函数当作属性vm.计算属性的getter
    // 最终computed与data会一起混合到vm下，所以当computed与data存在同名属性时会抛出警告
    defineComputed(vm, key, userDef);
    ...
  }
}

export function defineComputed(target, key, userDef) {
  // 创建set get 方法
  sharedPropertyDefinition.get = createComputedGetter(key);
  sharedPropertyDefinition.get = noop;
  ...
  // 创建属性vm.计算属性，并初始化getter setter
  Object.defineProperty(target, key, sharedPropertyDefinition);
}

function createComputedGetter(key) {
  return function computedGetter() {
    const watcher = this._computedWatchers && this._computedWatchers[key];
    if (watcher) {
      if (watcher.dirty) {
        // watcher暴露evaluate方法用于取值操作
        watcher.evaluate();
      }
      // 同第一步，判断是否处于依赖收集状态
      if (Dep.target) {
        watcher.depend();
      }
      return watcher.value;
    }
  }
}
```
1. data属性初始化getter setter
2. computed计算属性初始化，提供的函数将用作属性vm.计算属性的getter
3. 当首次获取计算属性的值时，Dep开始依赖收集
4. 在执行某个属性 getter方法时，如果Dep处于依赖收集状态，则判定该属性为计算属性的依赖，并建立依赖关系
5. 当 某个属性发生变化时，根据依赖关系，触发计算属性的重新计算


## 5. vue模板编译原理
1. 将模板字符串转换成elements ASTs(解析器)；
2. 对AST进行静态节点标记，主要用来做虚拟dom的渲染优化（优化器）；
3. 使用elements ASTs生成render函数代码字符串（代码生成器）

例如：
```html
<div><p>{{name}}</p></div>
```

elements AST:
```js
{
  // 以"<"开头截取的这段字符串是标签or文本
  tag："div",
  type: 1,
  staticRoot: false,
  parent: undefined,
  children: [
    {
      tag: "p",
      type: 1,
      staticRoot: false,
      static: false,
      children: [{
        type: 2,
        text: "{{name}}",
        static: false,
        ...
      }]
    }
  ]
}
```
### 截取文本；

```js
// 比如div解析后剩余模板字符串<p>{{name}}</p></div>
let textEnd = html.indexOf('<');
let text,rest, next;
if (textEnd >= 0) {
  rest = html.slice(textEnd);
  // 剩余部分html不符合标签格式暂定是文本
  // 并且是以<开头的文本
  const ncname = '[a-zA-Z][\\w\\-\\.]*';
  const qnameCapture = `((?:${ncname}\\:)?${ncname})`;
  const startTagOpen = new RegExp(`^<${qnameCapture}`);
  const startTagClose = /^\s*(\/?)>/;
  while(!endTag.test(rest) && !startTagOpen.test(rest) && ...) {
    next = rest.indexOf('<', 1);
    if (next < 0) {
      break;
    }
    textEnd += next;
    rest = html.slice(textEnd);
  }
  text = html.substring(0, textEnd);
  html = html.suustring(0, textEnd);
}
```
解析文本：
* 纯文本：直接将文本节点的ast push到parent的children中；
* 带变量：多一个const expression = parseText(text, delimiters);

结束标签的处理：
用当前标签名在stack从后往前找，将找到的stack中的位置往后的所有标签删除。

标记静态节点好处：
1. 每次重新渲染不需要为静态节点创建节点；
2. 在虚拟dom中patching过程跳过；

### 代码生成器：

elements ASTs ——> render函数代码
例如开头AST：
```js
render: `with(this) {
  return _c('div', [_c('p', [_v(_s(name))])])
}`

// 即
with(this) {
  return _c(
    'div',
    [_c(
      'p',
      [_v(_s(name))]
    )]
  )
}
// _c: createElement(创建一个元素)
// _v: 创建文本节点
// _s: 返回参数中字符串

function genData(el, state) {
  let data = '{';
  if (el.key) {
    data += `key:${el.key},`;
  }
  if (el.ref) {
    data += `ref:${el.ref},`;
  }
  if (el.refInFor) {
    data += `refInFor:true,`;
  }
  if (el.pre) {
    data += `pre:true,`;
  }
  // 类似...
  data = data.replace(/,$/, '') + '}';
  return data;
}

function genChildren(el, state) {
  const children = el.children;
  if (children.length) {
    return `[${children.map(c => genNode(c, state)).join(',')}]`;
  }
}

function genNode(node, state) {
  if (node.type === 1) {
    return genElement(node, state);
  }
  if (node.type === 3 && node.isComment) {
    return genComment(node);
  } else {
    return genText(node); // 带变量文本
  }
}

function genElement(el, state) {
  const data = el.plain ? undefined : genData(el, state);
  const children = el.inlineTemplate ? null ? genChildren(el, state, true);
  let node = `_c['${el.tag}'${data ? `,${data}`: ''} ${
    children ? `,${children}`: ''
  }]`;
  return node;
}
```
