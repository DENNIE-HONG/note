# react-router


## v6版本


举个栗子：

```js
import { Routes , Route , Outlet  } from 'react-router';
import { BrowserRouter } from 'react-router-dom';
const index = () => {
  return <div className="page" >
    <div className="content" >
       <BrowserRouter >
           <Routes>
              <Route element={<Home />}
                  path="/home"
              ></Route>
              <Route element={<Layout/>}
                  path="/children"
              >
                  <Route element={<Child1/>}
                      path="/children/child1"
                  ></Route>
                  <Route element={<Child2/>}
                      path="/children/child2"
                  ></Route>
              </Route>
           </Routes>
       </BrowserRouter>
    </div>
  </div>
}

//嵌套路由layout里就是一个outlet组件。

function Layout(){
  return (
    <div>
      <div> <Outlet/></div>
    </div>
  );
}


```


1. v6版本的 router 没有 Switch 组件，取而代之的是 Routes ，但是在功能上 Routes 是核心的，起到了不可或缺的作用。老版本的 route 可以独立使用，新版本的 route 必须配合 Routes 使用。


2. v6版本路由引入 Outlet 占位功能，可以更方便的配置路由结构，不需要像老版本路由那样，子路由配置在具体的业务组件中，这样更加清晰，灵活。



## history篇
v5.2.0 管理会话历史
内部封装了一些系列操作浏览器历史栈的功能，并提供了三种不同性质的history导航创建方法

* createBrowserHistory基于浏览器history对象最新 api。
* createHashHistory：基于浏览器 url 的 hash 参数。
* createMemoryHistory：基于内存栈，不依赖任何平台。 

### 通用基础概念

**路由切换时的 Action**
action被分为了三大类：

* pop
* push
* replace

```js
export enum Action {
  Pop = 'POP',
  Push = 'PUSH',
  Replace = 'REPLACE'
}
```
**Path 与 Location对象**

path：

```ts
// 下面三个分别是对 url 的 path，query 与 hash 部分的类型别名
export type Pathname = string;
export type Search = string;
export type Hash = string;

// 一次跳转 url 对应的对象
export interface Path {
  pathname: Pathname;
  search: Search;
  hash: Hash;
}

```

产生完整的url：

```ts
/**
 *  pathname + search + hash 创建完整 url
 */
export function createPath({
  pathname = '/',
  search = '',
  hash = ''
}: Partial<Path>) {
  if (search && search !== '?')
    pathname += search.charAt(0) === '?' ? search : '?' + search;
  if (hash && hash !== '#')
    pathname += hash.charAt(0) === '#' ? hash : '#' + hash;
  return pathname;
}

/**
 * 解析 url，将其转换为 Path 对象
 */
export function parsePath(path: string): Partial<Path> {
  let parsedPath: Partial<Path> = {};

  if (path) {
    let hashIndex = path.indexOf('#');
    if (hashIndex >= 0) {
      parsedPath.hash = path.substr(hashIndex);
      path = path.substr(0, hashIndex);
    }

    let searchIndex = path.indexOf('?');
    if (searchIndex >= 0) {
      parsedPath.search = path.substr(searchIndex);
      path = path.substr(0, searchIndex);
    }

    if (path) {
      parsedPath.pathname = path;
    }
  }

  return parsedPath;
}

```

**Location对象**

```ts
// 唯一字符串，与每次跳转的 location 匹配
export type Key = string;

// 路由跳转抽象化的导航对象
export interface Location extends Path {
  // 与当前 location 关联的 state 值，可以是任意手动传入的值
  state: unknown;
  // 当前 location 的唯一 key，一般都是自动生成
  key: Key;
}

```

### 深入history
在之前我们提到了history内部的三种导航方法会创建出三个有着不同性质的History对象，但其实这些History对象向外暴露的 api 大体都是一致的。


```ts
// listen 回调的参数，包含有更新的行为 Action 和 Location 对象
export interface Update {
  action: Action; // 上面提到的 Action
  location: Location; // 上面提到的 Location
}

// 监听函数 listener 的定义
export interface Listener {
  (update: Update): void;
}


// block 回调的参数，除了包含有 listen 回调参数的所有值外还有一个 retry 方法
// 如果阻止了页面跳转（blocker 监听），可以使用 retry 重新进入页面
export interface Transition extends Update {
  /**
   * 重新进入被 block 的页面
   */
  retry(): void;
}

/**
 * 页面跳转失败后拿到的 Transition 对象
 */
export interface Blocker {
  (tx: Transition): void;
}

// 跳转链接，可以是完整的 url，也可以是 Path 对象
export type To = string | Partial<Path>;

export interface History {
  // 最后一次浏览器跳转的行为，可变
  readonly action: Action;

  // 挂载有当前的 location 可变
  readonly location: Location;

  // 工具方法，把 to 对象转化为 url 字符串，其实内部就是对之前提到的 createPath 函数的封装
  createHref(to: To): string;

  // 推入一个新的路由到路由栈中
  push(to: To, state?: any): void;
  // 替换当前路由
  replace(to: To, state?: any): void;
  // 将当前路由指向路由栈中第 delta 个位置的路由
  go(delta: number): void;
  // 将当前路由指向当前路由的前一个路由
  back(): void;
  // 将当前路由指向当前路由的后一个路由
  forward(): void;

  // 页面跳转后触发，相当于后置钩子
  listen(listener: Listener): () => void;

  // 也是监听器，但是会阻止页面跳转，相当于前置钩子，注意只能拦截当前 history 对象的钩子，也就是说如果 history 对象不同，是不能够拦截到的
  block(blocker: Blocker): () => void;
}

```

#### 创建

```js
export interface BrowserHistory extends History {}
export interface HashHistory extends History {}
export interface MemoryHistory extends History {
  readonly index: number;
}

```

**createBrowserHistory**
用于给用户提供的创建基于浏览器 history API 的History对象，适用于绝大多数现代浏览器（除了少部分不支持 HTML5 新加入的 history API 的浏览器，也就是浏览器的history对象需要具有pushState、replaceState和state等属性和方法），同时在生产环境需要服务端的重定向配置才能正常使用。


```ts
// 可以传入指定的 window 对象作为参数，默认为当前 window 对象
export type BrowserHistoryOptions = { window?: Window };


export function createBrowserHistory(
  options: BrowserHistoryOptions = {}
): BrowserHistory {
  let { window = document.defaultView! } = options;
  // 拿到浏览器的 history 对象，后续会基于此对象封装方法
  let globalHistory = window.history;
  // 初始化 action 与 location
  let action = Action.Pop;
  // 获取当前路由的 index 和 location
  let [index, location] = getIndexAndLocation(); 

  // 省略其余代码
  
  let history: BrowserHistory = {
    ...
  };

  return history;
}

```

**createHashHistory**

用于给用户提供基于浏览器 url hash 值的History对象，一般来说使用这种方式可以兼容几乎所有的浏览器，但是考虑到目前浏览器的发展，在5.x版本内部其实同createBrowserHistory，也是使用最新的 history API 实现路由跳转的（如果你确实需要兼容旧版本浏览器，应该选择使用4.x版本），同时由于浏览器不会将 url 的 hash 值发送到服务端，前端发送的路由 url 都是一致的，就不用服务端做额外配置了。


```ts
// 这里同 BrowserRouter
export type HashHistoryOptions = { window?: Window };



export function createHashHistory(
  options: HashHistoryOptions = {}
): HashHistory {
  let { window = document.defaultView! } = options;
  // 浏览器本身就有 history 对象，只是 HTML5 新加入了几个有关 state 的 api
  let globalHistory = window.history;
  let action = Action.Pop;
  let [index, location] = getIndexAndLocation();

  // 省略其余代码
  
  let history: HashHistory = {
    ...
  }

  return history;
}

```

**createMemoryHistory**

用于给用户提供基于内存系统的History对象，适用于所有可以运行 JavaScript 的环境（包括 Node），内部的路由系统完全由用户支配。

```ts
// 这里与 BrowserRouter 和 HashRouter 相比有略微不同，因为没有浏览器的参与，所以我们需要模拟历史栈
// 用户提供的描述历史栈的对象
export type InitialEntry = string | Partial<Location>;// 上面提到的 Location 

// 因为不是真实的路由，所以不需要 window 对象，取而代之的是
export type MemoryHistoryOptions = {
  // 初始化的历史栈
  initialEntries?: InitialEntry[];
  // 初始化的 index
  initialIndex?: number;
};


// 判断上下限值
function clamp(n: number, lowerBound: number, upperBound: number) {
  return Math.min(Math.max(n, lowerBound), upperBound);
}

export function createMemoryHistory(
  options: MemoryHistoryOptions = {}
): MemoryHistory {
  let { initialEntries = ['/'], initialIndex } = options;
  // 将用户传入的 initialEntries 转换为包含 Location 对象数组，会在之后用到
  let entries: Location[] = initialEntries.map((entry) => {
    // readOnly 就是调用 Object.freeze 冻结对象，这里做了个开发模式的封装，遇到都可以直接跳过
    let location = readOnly<Location>({
      pathname: '/',
      search: '',
      hash: '',
      state: null,
      key: createKey(),
      ...(typeof entry === 'string' ? parsePath(entry) : entry)
    });
    return location;
  });
  // 这里的 location 与 index 的获取方式不同了，是直接从初始化的 entries 中取的
  let action = Action.Pop;
  let location = entries[index];
  // clamp 函数用于取上下限值，如果 没有传 initialIndex 默认索引为最后一个 location
  // 这这里调用是为了规范初始化的 initialIndex 的值
  let index = clamp(
    initialIndex == null ? entries.length - 1 : initialIndex,
    0,
    entries.length - 1
  );

  // 省略其余代码
  let history: MemoryHistory = {
    ...
  };

  return history;
}

```








#### action、location 与 index（MemoryRouter 独有）

**createBrowserHistory**
```ts
export function createBrowserHistory(
  options: BrowserHistoryOptions = {}
): BrowserHistory {
  let { window = document.defaultView! } = options;
  let globalHistory = window.history;

  /**
   * 拿到当前的 state 的 idx 和 location 对象
   */
  function getIndexAndLocation(): [number, Location] {
    let { pathname, search, hash } = window.location;
    // 获取当前浏览器的 state
    let state = globalHistory.state || {};
    // 可以看到下面很多属性都是保存到了 history api 的 state 中
    return [
      state.idx,
      readOnly<Location>({
        pathname,
        search,
        hash,
        state: state.usr || null,
        key: state.key || 'default'
      })
    ];
  }
  let action = Action.Pop;
  let [index, location] = getIndexAndLocation();
  
  // 初始化 index
  if (index == null) {
    index = 0;
    // 调用的是 history api 提供的 replaceState 方法传入 index，这里只是初始化浏览器中保存的 state，没有改变 url
    globalHistory.replaceState({ ...globalHistory.state, idx: index }, '');
  }

  //...

  let history: BrowserHistory = {
    get action() {
      return action;
    },
    get location() {
      return location;
    }
    // ...
  };

  return history;
}

```


**createHashHistory**
```ts
export function createHashHistory(
  options: HashHistoryOptions = {}
): HashHistory {
  let { window = document.defaultView! } = options;
  let globalHistory = window.history;

 function getIndexAndLocation(): [number, Location] {
    // 注意这里和 browserHistory 不同了，拿的是 hash，其余逻辑是一样的
    // parsePath 方法前面有讲到过，解析 url 为 Path 对象
    let {
      pathname = '/',
      search = '',
      hash = ''
    } = parsePath(window.location.hash.substr(1));
    let state = globalHistory.state || {};
    return [
      state.idx,
      readOnly<Location>({
        pathname,
        search,
        hash,
        state: state.usr || null,
        key: state.key || 'default'
      })
    ];
  }

  let action = Action.Pop;
  let [index, location] = getIndexAndLocation();

 if (index == null) {
    index = 0;
    globalHistory.replaceState({ ...globalHistory.state, idx: index }, '');
  }
  //...

  let history: HashHistory = {
    get action() {
      return action;
    },
    get location() {
      return location;
    }
    // ...
  };

  return history;
}

```

#### push 和 replace

需要封装将新的导航推入到历史栈的功能外，还需要同时改变当前的action与location，并且判断并调用相应的监听方法。

通用方法：
```ts
function getNextLocation(to: To, state: any = null): Location {
    // 简单的格式化
  return readOnly<Location>({
    // 这前面三个实际上是默认值
    pathname: location.pathname,
    hash: '',
    search: '',
    // 下面胡返回带有 pathname、hash、search 的对象
    ...(typeof to === 'string' ? parsePath(to) : to),
    state,
    key: createKey()
  });
}
/**
* 调用所有 blockers，没有 blocker 的监听时才会返回 true，否则都是返回 false
*/
function allowTx(action: Action, location: Location, retry: () => void) {
    return (
      !blockers.length || (blockers.call({ action, location, retry }), false)
    );
}

```

**createBrowserHistory和createHashHistor**
```ts

type HistoryState = {
  usr: any;
  key?: string;
  idx: number;
};

function getHistoryStateAndUrl (
  nextLocation: Location,
  index: number
): [HistoryState, string] {
  return [
    {
      usr: nextLocation.state,
      key: nextLocation.key,
      idx: index
    },
    createHref(nextLocation)
  ];
}

function applyTx(nextAction: Action) {
    action = nextAction;
    // 在这里修改了 index 与 location，getIndexAndLocation 方法我们在前面将 location 属性的时候有提到过
    [index, location] = getIndexAndLocation();
    listeners.call({ action, location });
}


function push(to: To, state?: any) {
    let nextAction = Action.Push;
    let nextLocation = getNextLocation(to, state);

    /**
     * 重新执行 push 操作
     */
    function retry() {
      push(to, state);
    }

    // 当没有 block 监听时 allowTx 返回 true，否则都是返回 false，不会推送新的导航
    if (allowTx(nextAction, nextLocation, retry)) {
      let [historyState, url] = getHistoryStateAndUrl(nextLocation, index + 1);

      // try...catch 是因为 ios 限制最多调用 100 次 pushState 方法，否则会报错
      try {
        globalHistory.pushState(historyState, '', url);
      } catch (error) {
        // push 失败后就没有 state 了，直接使用 href 跳转
        window.location.assign(url);
      }

      applyTx(nextAction);
    }
}

function replace(to: To, state?: any) {
    let nextAction = Action.Replace;
    let nextLocation = getNextLocation(to, state);
    function retry() {
      replace(to, state);
    }

    // 同 push 函数，否则不会替换新的导航
    if (allowTx(nextAction, nextLocation, retry)) {
      let [historyState, url] = getHistoryStateAndUrl(nextLocation, index);

      globalHistory.replaceState(historyState, '', url);

      applyTx(nextAction);
    }
}


```

**createMemoryHistory**



```ts

function applyTx(nextAction: Action, nextLocation: Location) {
    // 没有在这里改变 index ，和其余 router 不同，将 index 改变操作具体到了 push 和 go 等函数中
    action = nextAction;
    location = nextLocation;
    listeners.call({ action, location });
}



function push(to: To, state?: any) {
  let nextAction = Action.Push;
  let nextLocation = getNextLocation(to, state);
  function retry() {
    push(to, state);
  }

  if (allowTx(nextAction, nextLocation, retry)) {
    // 修改 index 与 entries 历史栈数组
    index += 1;
    // 添加一个新的 location，删除原来 index 往后的栈堆
    entries.splice(index, entries.length, nextLocation);
    applyTx(nextAction, nextLocation);
  }
}

function replace(to: To, state?: any) {
  let nextAction = Action.Replace;
  let nextLocation = getNextLocation(to, state);
  function retry() {
    replace(to, state);
  }

  if (allowTx(nextAction, nextLocation, retry)) {
    // 覆盖掉原来的 location
    entries[index] = nextLocation;
    applyTx(nextAction, nextLocation);
  }
}

```



#### 浏览器 popstate 事件的监听
这里只有createBrowserHistory和createHashHistory才会有该事件的监听：

```ts
const HashChangeEventType = 'hashchange';
const PopStateEventType = 'popstate';

export function createBrowserHistory(
  options: BrowserHistoryOptions = {}
): BrowserHistory {
   //...

  let blockedPopTx: Transition | null = null;
  /**
   * 事件监听回调函数
   * 如果设置了 blocker 的监听器，该函数会执行两次，第一次是跳回到原来的页面，第二次是执行 blockers 的所有回调
   * 这个函数用于监听浏览器的前进后退，因为我们封装的 push 函数已经被我们拦截了
   */
  function handlePop() {
    if (blockedPopTx) {
      blockers.call(blockedPopTx);
      blockedPopTx = null;
    } else {
      let nextAction = Action.Pop;
      let [nextIndex, nextLocation] = getIndexAndLocation();

      // 如果有前置钩子
      if (blockers.length) {
        if (nextIndex != null) {
          // 计算跳转层数
          let delta = index - nextIndex;
          if (delta) {
            // Revert the POP
            blockedPopTx = {
              action: nextAction,
              location: nextLocation,
              // 恢复页面栈，也就是 nextIndex 的页面栈
              retry() {
                go(delta * -1);
              }
            };
            // 跳转回去（index 原本的页面栈）
            go(delta);
          }
        } else {
          // asset
          // nextIndex 如果为 null 会进入该分支打警告信息，这里就先不管它
        }
      } else {
        // 改变当前 action，调用所有的 listener
        applyTx(nextAction);
      }
    }
  }

  // 可以看到在创建 History 对象的时候就进行监听了
  window.addEventListener(PopStateEventType, handlePop);
  //...
}
```

**createHashHistory**
```ts

export function createHashHistory(
  options: HashHistoryOptions = {}
): HashHistory {
   //...

  // 下面和 createBrowserHistory 一样
  let blockedPopTx: Transition | null = null;
  function handlePop() {
    //...
  }
  
    
  // 下面额外监听了 hashchange 事件
  window.addEventListener(PopStateEventType, handlePop);
  // 低版本兼容，监听 hashchange 事件
  // https://developer.mozilla.org/de/docs/Web/API/Window/popstate_event
  window.addEventListener(HashChangeEventType, () => {
    let [, nextLocation] = getIndexAndLocation();

    // 如果支持 popstate 事件这里就会相等，因为会先执行 popstate 的回调
    if (createPath(nextLocation) !== createPath(location)) {
      handlePop();
    }
  });
  //...
}

```
