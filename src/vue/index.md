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
