# Vue2.0（早期的）
## Vue的双向绑定
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

## Diff
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

## nextTick的实现原理
1. vue用异步队列的方式来控制DOM更新和NnextTick回调先后执行；
2. 微任务因为优先级高特性，能确保队列中的微任务在一次事件循环前被执行完毕；
3. 因为兼容问题，vue不得不做了微任务向宏任务降级方案；