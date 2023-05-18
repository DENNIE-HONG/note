# React

## 1. React V16
新特性：
1. react hooks；
2. 允许在render函数返回节点数组和字符串
```jsx
return [
  <li>1</li>
  <li>2</li>
  ...
];
```
3. 更好的错误处理；
4. 支持自定义DOM属性；
5. 重写，改为Fiber异步渲染架构；

## 2. Fiber
Fiber是新的reconciler
* reconciliation: effect list
* commit: 同步
```js
     ○               <=>      ○        ->      ○ (fiber)
一个ReactElement            current          alternate fiber
更新后： ○ alternate成为新的current fiber
```

```js
fiber {
  tag: fiber类型,
  alternate: 更新时克隆出的fiber,
  'return': 指向fiber树中的父节点,
  child: 第一个节点,
  sibling: 兄弟节点,
  effectTag: side effect类型,
  pendingWorkPriority: 标记子树上更新任务优先级,
  nextEffect: 单链表结构，方便遍历fiber树上有副作用节点,
}
```
### reconciliation
fiber phase 1: render reconciliation, 调用的生命周期方法：
* componentWillMount
* componnetWillReceiveProps
* shouldComponentUpdate
* componentWillUpdate


### commiet
fiber phase 2: commit
* componentDidMount（commit 2阶段）
* componentDidUpdate(commit 2阶段)
* componentWillUnmount(commit 1阶段)

如果effectTag是Deletion, 调用commitDeletion做处理。  
commitDeletion会递归地将子节点从fiber树上移除，对于节点上存在的ref做detach,调用componentWillUnmount生命周期钩子。最后调用renderer传入平台相关方法removeChild和removeChildFromContainer更新UI.

## 3. 生命周期

```js

      _____   
     |start|
      —————            ___________
        ↓             |           |
 ________________     |   ________|_________
|getDefaultProps |    |  |ComponentDidUpdate|
 ————————————————     |   —————————————————— 
        ↓             |           ↑
 _______________      |        ______
|getInitialState|     |       |render| 
 ———————————————      |        ——————
        ↓             |           ↑
 __________________   |   ___________________
|ComponentWillMount|  |  |ComponentWillUpdate|
 ——————————————————   |   ——————————————————— 
        ↓             |           ↑
     ______           |   _____________________
    |render|          |  |ComponentShouldUpdate|<-----
     ——————           |   —————————————————————       |
        ↓             |           ↑                   |
 _________________    |        _______       _________|_______________
|ComponentDidMount|——————————>| 运行中 |————>|ComponentWillReceiveProps|
 —————————————————             ———————       —————————————————————————
```

* 获取父传props -> 初始化state -> （将会挂载、组件渲染、挂载）
* props改变 -> ComponentWillReceiveProps ——> ComponentShouldUpdate判定是否需要更新(默认true)
* state改变 -> ComponentShouldUpdate ? true: （组件将更新、渲染、更新完毕）

## 4. setState
调用this.updater.enqueneSetState  ——> addUpdate(向队列中推入需要更新fiber) ——> scheduleUpdate（触发调度器一次新更新）

**scheduleUpdate**:
从当前触发节点向上搜索。父节点不是hostRoot(ReactDOM.render()的根节点)，且更新父节点的peddingWorkPriority,标记这个节点上等待更新事务的优先级。  
父节点是hostRoot, 调用scheduleRoot,根据优先级决定是否立即执行update。  
有nextScheduleRoot指针指向下一个待更新HostRoot,构成链表结构。


## 5. 任务调度
利用requestIdleCallback实现task scheduling
```js
                                                      |
                      Frame#1                         |        Frame#2
  ______    __________   ___________________________  |   ______     ______ 
 |  run |  |  Update  | |       idle period         | |  |  run |   |  run |
 | task |  |Rendering | |  __________   __________  | |  | task |   | task |  ...
  ——————    ——————————  | |  idle    | |   idle   | | |   ——————     —————— 
                        | | callback | | callback | | |
                        |  ——————————   ——————————  | |
                         ———————————————————————————  |
                                                      | 
                                                      |
```
requestIdleCallback回调执行时间：
一帧开始，JS执行完，浏览器渲染后，到这帧结束之前。

## 6. Flux模式
数据和逻辑永远是单向流动

```js
    __________________________________
   \|/                                |
 __ |__     __________     _____     _|__
|Action|——>|dispatcher|——>|store|——>|view|
 ——————     ——————————     —————     ————

```
* dispatcher: 分发事件；
* store: 保存数据，响应事件更新数据；
* view: 订阅store中数据，渲染页面；
