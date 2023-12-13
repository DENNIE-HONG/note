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









### 深入Router
不过我们一般不会直接使用Router，更多会使用已经封装好路由导航对象的BrowserRouter（react-router-dom包引入）、HashRouter（react-router-dom包引入）和MemoryRouter（react-router包引入）。

#### 两个**Context**
用于存储全局的路由导航对象以及导航位置的上下文。

```ts
import React from 'react'
import type {
  History,
  Location,
} from "history";
import {
  Action as NavigationType,
} from "history";

// 只包含，go、push、replace、createHref 四个方法的 History 对象，用于在 react-router 中进行路由跳转
export type Navigator = Pick<History, "go" | "push" | "replace" | "createHref">;

interface NavigationContextObject {
  basename: string;
  navigator: Navigator;
  static: boolean;
}

/**
 * 内部含有 navigator 对象的全局上下文，官方不推荐在外直接使用
 */
const NavigationContext = React.createContext<NavigationContextObject>(null!);


interface LocationContextObject {
  location: Location;
  navigationType: NavigationType;
}
/**
 * 内部含有当前 location 与 action 的 type，一般用于在内部获取当前 location，官方不推荐在外直接使用
 */
const LocationContext = React.createContext<LocationContextObject>(null!);

// 这是官方对于上面两个 context 的导出，可以看到都是被定义为不安全的，并且可能会有着重大更改，强烈不建议使用
/** @internal */
export {
  NavigationContext as UNSAFE_NavigationContext,
  LocationContext as UNSAFE_LocationContext,
};

```

#### 定义Router

```tsx
// 接上面，这里额外还从 history 中引入了 parsePath 方法
import {
  parsePath
} from "history";

export interface RouterProps {
  // 路由前缀
  basename?: string;
  children?: React.ReactNode;
  // 必传，当前 location
  /*
      interface Location {
            pathname: string;
            search: string;
            hash: string;
            state: any;
            key: string;
      }
  */
  location: Partial<Location> | string;
  // 当前路由跳转的类型，有 POP，PUSH 与 REPLACE 三种
  navigationType?: NavigationType;
  // 必传，history 中的导航对象，我们可以在这里传入统一外部的 history
  navigator: Navigator;
  // 是否为静态路由（ssr）
  static?: boolean;
}

/**
 * 提供渲染 Route 的上下文，但是一般不直接使用这个组件，会包装在 BrowserRouter 等二次封装的路由中
 * 整个应用程序应该只有一个 Router
 * Router 的作用就是格式化传入的原始 location 并渲染全局上下文 NavigationContext、LocationContext
 */
export function Router({
  basename: basenameProp = "/",
  children = null,
  location: locationProp,
  navigationType = NavigationType.Pop,
  navigator,
  static: staticProp = false
}: RouterProps): React.ReactElement | null {
  // 断言，Router 不能在其余 Router 内部，否则抛出错误
  invariant(
    !useInRouterContext(),
    `You cannot render a <Router> inside another <Router>.` +
      ` You should never have more than one in your app.`
  );
  // 格式化 basename，去掉 url 中多余的 /，比如 /a//b 改为 /a/b
  let basename = normalizePathname(basenameProp);
  // 全局的导航上下文信息，包括路由前缀，导航对象等
  let navigationContext = React.useMemo(
    () => ({ basename, navigator, static: staticProp }),
    [basename, navigator, staticProp]
  );

  // 转换 location，传入 string 将转换为对象
  if (typeof locationProp === "string") {
    // parsePath 用于将 locationProp 转换为 Path 对象，都是 history 库引入的
    /*
        interface Path {
              pathname: string;
              search: string;
              hash: string;
        }
    */
    locationProp = parsePath(locationProp);
  }

  let {
    pathname = "/",
    search = "",
    hash = "",
    state = null,
    key = "default"
  } = locationProp;

  // 经过抽离 base 后的真正的 location，如果抽离 base 失败返回 null
  let location = React.useMemo(() => {
    // stripBasename 用于去除 pathname 前面 basename 部分
    let trailingPathname = stripBasename(pathname, basename);

    if (trailingPathname == null) {
      return null;
    }

    return {
      pathname: trailingPathname,
      search,
      hash,
      state,
      key
    };
  }, [basename, pathname, search, hash, state, key]);

  if (location == null) {
    return null;
  }

  return (
    // 唯一传入 location 的地方
    <NavigationContext.Provider value={navigationContext}>
      <LocationContext.Provider
        children={children}
        value={{ location, navigationType }}
      />
    </NavigationContext.Provider>
  );
}

```
只是提供Context与格式化外部传入的location对象（而实际上这个location对象一般也不用我们传入）


#### 使用Router

以MemoryRouter 源码解析为例（BrowserRouter与HashRouter跟它原理类似）

```tsx
import type { InitialEntry, MemoryHistory } from 'history';
import { createMemoryHistory } from 'history';

export interface MemoryRouterProps {
  // 路由前缀
  basename?: string;
  children?: React.ReactNode;
  // 与 createMemoryHistory 返回的 history 对象参数相对应，代表的是自定义的页面栈与索引
  initialEntries?: InitialEntry[];
  initialIndex?: number;
}

/**
 * react-router 里面只有 MemoryRouter，其余的 router 在 react-router-dom 里
 */
export function MemoryRouter({
    basename,
    children,
    initialEntries,
    initialIndex
}: MemoryRouterProps): React.ReactElement {
    // history 对象的引用
    let historyRef = React.useRef<MemoryHistory>();
    if (historyRef.current == null) {
        // 创建 memoryHistory
        historyRef.current = createMemoryHistory({ initialEntries, initialIndex });
    }

    let history = historyRef.current;
    let [state, setState] = React.useState({
        action: history.action,
        location: history.location
    });

    // 监听 history 改变，改变后重新 setState
    React.useLayoutEffect(() => history.listen(setState), [history]);

    // 简单的初始化并将相应状态与 React 绑定
    return (
        <Router
            basename={basename}
            children={children}
            location={state.location}
            navigationType={state.action}
            navigator={history}
        />
    );
}


```
高阶路由其实就是将history库与我们声明的Router组件绑定起来，当history.listen监听到路由改变后重新设置当前的location与action。
一般不会直接使用Router组件，而是使用react-router内部提供的高阶Router组件，而这些高阶组件实际上就是将history库中提供的导航对象与Router组件连接起来，进而控制应用的导航状态。


#### 开始配置 Route

```tsx
import { render } from "react-dom";
import {
  BrowserRouter,
  Routes,
  Route
} from "react-router-dom";
// 这几个页面不用管它
import App from "./App";
import Expenses from "./routes/expenses";
import Invoices from "./routes/invoices";

const rootElement = document.getElementById("root");
render(
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<App />} />
      <Route path="/expenses" element={<Expenses />} />
      <Route path="/invoices" element={<Invoices />} />
    </Routes>
  </BrowserRouter>,
  rootElement
);

```
APP内部：

```tsx
import { Outlet } from 'react-router'
export function App() {
    return (
        <>
          App
          <Outlet/>
        </>
    )
}

```
Outlet，该组件用于在父路由元素中呈现它们的子路由元素。

也就是说，后续子路由匹配到的内容都会放到Outlet组件中，当父路由元素在内部渲染它时，就会展示匹配到的子路由元素。



#### Route源码解析

```tsx
// Route 有三种 props 类型，这里先了解内部参数的含义，下面会细讲
export interface PathRouteProps {
  caseSensitive?: boolean;
  // children 代表子路由
  children?: React.ReactNode;
  element?: React.ReactNode | null;
  index?: false;
  path: string;
}

export interface LayoutRouteProps {
  children?: React.ReactNode;
  element?: React.ReactNode | null;
}

export interface IndexRouteProps {
  element?: React.ReactNode | null;
  index: true;
}

/**
 * Route 组件内部没有进行任何操作，仅仅只是定义 props，而我们就是为了使用它的 props
 */
export function Route(
    _props: PathRouteProps | LayoutRouteProps | IndexRouteProps
): React.ReactElement | null {
    // 这里可以看出 Route 不能够被渲染出来，渲染会直接抛出错误，证明 Router 拿到 Route 后也不会在内部操作
    invariant(
        false,
        `A <Route> is only ever to be used as the child of <Routes> element, ` +
        `never rendered directly. Please wrap your <Route> in a <Routes>.`
    );
}

```
它在react-router内仅仅只是个**传递参数的工具人**（后续讲Routes会细说），对用户的唯一作用就是提供命令式的路由配置方式。


#### Routes源码解析

```ts
export interface RoutesProps {
    children?: React.ReactNode;
    // 用户传入的 location 对象，一般不传，默认用当前浏览器的 location
    location?: Partial<Location> | string;
}

/**
 * 所有的 Route 都需要 Routes 包裹，用于渲染 Route（拿到 Route 的 props 的值，不渲染真实的 DOM 节点）
 */
export function Routes({
    children,
    location
}: RoutesProps): React.ReactElement | null {
    return useRoutes(createRoutesFromChildren(children), location);
}

```
调用**useRoutes**这个hook，并且使用了createRoutesFromChildren这个方法将children转换为了useRoutes的配置参数，从而得到最后的路由元素。


**createRoutesFromChildren**:
```tsx
// 路由配置对象
export interface RouteObject {
    // 路由 path 是否匹配大小写
    caseSensitive?: boolean;
    // 子路由
    children?: RouteObject[];
    // 要渲染的组件
    element?: React.ReactNode;
    // 是否是索引路由
    index?: boolean;
    path?: string;
}

/**
 * 将 Route 组件转换为 route 对象，提供给 useRoutes 使用
 */
export function createRoutesFromChildren(
    children: React.ReactNode
): RouteObject[] {
    let routes: RouteObject[] = [];

    // 内部逻辑很简单，就是递归遍历 children，获取 <Route /> props 上的所有信息，然后格式化后推入 routes 数组中
    React.Children.forEach(children, element => {
        if (!React.isValidElement(element)) {
            // Ignore non-elements. This allows people to more easily inline
            // conditionals in their route config.
            return;
        }

        // 空节点，忽略掉继续往下遍历
        if (element.type === React.Fragment) {
            // Transparently support React.Fragment and its children.
            routes.push.apply(
                routes,
                createRoutesFromChildren(element.props.children)
            );
            return;
        }

        // 不要传入其它组件，只能传 Route
        invariant(
            element.type === Route,
            `[${
                typeof element.type === "string" ? element.type : element.type.name
            }] is not a <Route> component. All component children of <Routes> must be a <Route> or <React.Fragment>`
        );

        let route: RouteObject = {
            caseSensitive: element.props.caseSensitive,
            element: element.props.element,
            index: element.props.index,
            path: element.props.path
        };

        // 递归
        if (element.props.children) {
            route.children = createRoutesFromChildren(element.props.children);
        }

        routes.push(route);
    });

    return routes;
}

```
结论：

* react-router在路由定义时同时提供了两种方式：命令式与声明式，而这两者本质上都是调用的同一种路由生成的方法。
* Route可以被看做一个挂载用户传入参数的对象，它不会在页面中渲染，而是会被Routes接受并解析，我们也不能单独使用它。
* Routes与Route强绑定，有Routes则必定要传入且只能传入Route。

####  useRoutes（核心概念）

生命式写法举例

```tsx
import { useRoutes } from "react-router-dom";

// 此时 App 返回的就是已经渲染好的路由元素了
function App() {
    let element = useRoutes([
        {
            path: "/",
            element: <Dashboard />,
            children: [
                {
                    path: "/messages",
                    element: <DashboardMessages />
                },
                { path: "/tasks", element: <DashboardTasks /> }
            ]
        },
        { path: "/team", element: <AboutPage /> }
    ]);

    return element;
}

```


#### useRoutes源码解析
在这里，我们又需要引入一个新的Context - RouteContext，它存储了两个属性：outlet与matches。

```tsx
/**
 * 动态参数的定义
 */
export type Params<Key extends string = string> = {
    readonly [key in Key]: string | undefined;
};

export interface RouteMatch<ParamKey extends string = string> {
    // params 参数，比如 :id 等
    params: Params<ParamKey>;
    // 匹配到的 pathname
    pathname: string;
    /**
     * 子路由匹配之前的路径 url，这里可以把它看做是只要以 /* 结尾路径（这是父路由的路径）中 /* 之前的部分
     */
    pathnameBase: string;
    // 定义的路由对象
    route: RouteObject;
}

interface RouteContextObject {
    // 一个 ReactElement，内部包含有所有子路由组成的聚合组件，其实 Outlet 组件内部就是它
    outlet: React.ReactElement | null;
    // 一个成功匹配到的路由数组，索引从小到大层级依次变深
    matches: RouteMatch[];
}
/**
 * 包含全部匹配到的路由，官方不推荐在外直接使用
 */
const RouteContext = React.createContext<RouteContextObject>({
    outlet: null,
    matches: []
});

/** @internal */
export {
    RouteContext as UNSAFE_RouteContext
};

```


#### 拆分useRoutes
内部逻辑十分复杂，我们先来看看最外层的代码，将其逻辑拆分出来：



```tsx

/**
 * 1.该 hooks 不是只调用一次，每次重新匹配到路由时就会重新调用渲染新的 element
 * 2.当多次调用 useRoutes 时需要解决内置的 route 上下文问题，继承外层的匹配结果
 * 3.内部通过计算所有的 routes 与当前的 location 关系，经过路径权重计算，得到 matches 数组，然后将 matches 数组重新渲染为嵌套结构的组件
 */
export function useRoutes(
    routes: RouteObject[],
    locationArg?: Partial<Location> | string
): React.ReactElement | null {
    // useRoutes 必须最外层有 Router 包裹，不然报错
    invariant(
        useInRouterContext(),
        // TODO: This error is probably because they somehow have 2 versions of the
        // router loaded. We can help them understand how to avoid that.
        `useRoutes() may be used only in the context of a <Router> component.`
    );

    // 1.当此 useRoutes 为第一层级的路由定义时，matches 为空数组（默认值）
    // 2.当该 hooks 在一个已经调用了 useRoutes 的渲染环境中渲染时，matches 含有值（也就是有 Routes 的上下文环境嵌套）
    let { matches: parentMatches } = React.useContext(RouteContext);
    // 最后 match 到的 route（深度最深），该 route 将作为父 route，我们后续的 routes 都是其子级
    let routeMatch = parentMatches[parentMatches.length - 1];
    // 下面是父级 route 的参数，我们会基于以下参数操作，如果项目中只在一个地方调用了 useRoutes，一般都会是默认值
    let parentParams = routeMatch ? routeMatch.params : {};
    // 父路由的完整 pathname，比如路由设置为 /foo/*，当前导航是 /foo/1，那么 parentPathname 就是 /foo/1
    let parentPathname = routeMatch ? routeMatch.pathname : "/";
    // 同上面的 parentPathname，不过是 /* 前的部分，也就是 /foo
    let parentPathnameBase = routeMatch ? routeMatch.pathnameBase : "/";
    let parentRoute = routeMatch && routeMatch.route;
    // 获取上下文环境中的 location
    let locationFromContext = useLocation();

    // 判断是否手动传入了 location，否则用默认上下文的 location
    let location;
    if (locationArg) {
        // 格式化为 Path 对象
        let parsedLocationArg =
        typeof locationArg === "string" ? parsePath(locationArg) : locationArg;
        // 如果传入了 location，判断是否与父级路由匹配（作为子路由存在）
        invariant(
            parentPathnameBase === "/" ||
                parsedLocationArg.pathname?.startsWith(parentPathnameBase),
            `When overriding the location using \`<Routes location>\` or \`useRoutes(routes, location)\`, ` +
                `the location pathname must begin with the portion of the URL pathname that was ` +
                `matched by all parent routes. The current pathname base is "${parentPathnameBase}" ` +
                `but pathname "${parsedLocationArg.pathname}" was given in the \`location\` prop.`
        );

        location = parsedLocationArg;
    } else {
        location = locationFromContext;
    }

    let pathname = location.pathname || "/";
    // 剩余的 pathname，整体 pathname 减掉父级已经匹配的 pathname，才是本次 routes 要匹配的 pathname（适用于 parentMatches 匹配不为空的情况）
    let remainingPathname =
        parentPathnameBase === "/"
        ? pathname
        : pathname.slice(parentPathnameBase.length) || "/";
    // 匹配当前路径，注意是移除了 parentPathname 的相关路径后的匹配
    
    // 通过传入的 routes 配置项与当前的路径，匹配对应渲染的路由
    let matches = matchRoutes(routes, { pathname: remainingPathname });

    // 参数为当前匹配到的 matches 路由数组和外层 useRoutes 的 matches 路由数组
    // 返回的是 React.Element，渲染所有的 matches 对象
    return _renderMatches(
        // 没有 matches 会返回 null
        matches &&
        matches.map(match =>
            // 合并外层调用 useRoutes 得到的参数，内部的 Route 会有外层 Route（其实这也叫父 Route） 的所有匹配属性。
            Object.assign({}, match, {
            params: Object.assign({}, parentParams, match.params),
            // joinPaths 函数用于合并字符串
            pathname: joinPaths([parentPathnameBase, match.pathname]),
            pathnameBase:
                match.pathnameBase === "/"
                ? parentPathnameBase
                : joinPaths([parentPathnameBase, match.pathnameBase])
            })
        ),
        // 外层 parentMatches 部分，最后会一起加入最终 matches 参数中
        parentMatches
    );
}

/**
 * 将多个 path 合并为一个
 * @param paths path 数组
 * @returns
 */
const joinPaths = (paths: string[]): string =>
  paths.join("/").replace(/\/\/+/g, "/");


```


这里整体概括一下useRoutes做的事情：

1. 获取上下文中调用useRoutes后的信息，如果有信息证明此次调用时作为子路由使用的，需要合并父路由的匹配信息。
2. 移除父路由已经匹配完毕的pathname前缀后，调用matchRoutes与当前传入的routes配置相匹配，返回匹配到的matches数组。
3. 调用_renderMatches方法，渲染上一步得到的matches数组。


整个流程对应三个阶段：路由上下文解析阶段，路由匹配阶段，路由渲染阶段。


#### 路由匹配阶段
路由匹配阶段其实就是调用matchRoutes方法的过程，我们来看看这个方法：

```tsx
/**
 * 通过 routes 与 location 得到 matches 数组
 */
export function matchRoutes(
    // 用户传入的 routes 对象
    routes: RouteObject[],
    // 当前匹配到的 location，注意这在 useRoutes 内部是先有过处理的
    locationArg: Partial<Location> | string,
    // 这个参数在 useRoutes 内部是没有用到的，但是该方法是对外暴露的，用户可以使用这个参数来添加统一的路径前缀
    basename = "/"
): RouteMatch[] | null {
    // 先格式化为 Path 对象
    let location =
        typeof locationArg === "string" ? parsePath(locationArg) : locationArg;

    // 之前提到过，抽离 basename，获取纯粹的 pathname
    let pathname = stripBasename(location.pathname || "/", basename);
  
    // basename 匹配失败，返回 null
    if (pathname == null) {
        return null;
    }

    // 1.扁平化 routes，将树状的 routes 对象根据 path 扁平为一维数组，同时包含当前路由的权重值
    let branches = flattenRoutes(routes);
    // 2.传入扁平化后的数组，根据内部匹配到的权重排序
    rankRouteBranches(branches);

    let matches = null;
    // 3.这里就是权重比较完成后的解析顺序，权重高的在前面，先进行匹配，然后是权重低的匹配
    // branches 中有一个匹配到了就终止循环，或者全都没有匹配到
    for (let i = 0; matches == null && i < branches.length; ++i)   {
        // 遍历扁平化的 routes，查看每个 branch 的路径匹配规则是否能匹配到 pathname
        matches = matchRouteBranch(branches[i], pathname);
    }

    return matches;
}

```
内部将路由的匹配分为了三个阶段：路由扁平化、路由权值计算与排序、路由匹配与合并。




#### 路由渲染阶段
useRoutes在内部是调用_renderMatches方法来实现的，这里先看源码：

```tsx
/**
 * 其实就是渲染 RouteContext.Provider 组件（包括多个嵌套的 Provider）
 */
function _renderMatches(
    matches: RouteMatch[] | null,
    // 如果在已有 match 的 route 内部调用，会合并父 context 的 match
    parentMatches: RouteMatch[] = []
): React.ReactElement | null {
    if (matches == null) return null;

    // 生成 outlet 组件，注意这里是从后往前 reduce，所以索引在前的 match 是最外层，也就是父路由对应的 match 是最外层
    /**
     *  可以看到 outlet 是通过不断递归生成的组件，最外层的 outlet 递归层数最多，包含有所有的内层组件，
     *  所以我们在外层使用的 <Outlet /> 是包含有所有子组件的聚合组件
     * */
    return matches.reduceRight((outlet, match, index) => {
        return (
            <RouteContext.Provider
                // 如果有 element 就渲染 element，如果没有填写 element，则默认是 <Outlet />，继续渲染内嵌的 <Route />
                children={
                match.route.element !== undefined ? match.route.element : <Outlet />
                }
                // 代表当前 RouteContext 匹配到的值，matches 并不是全局状态一致的，会根据层级不同展示不同的值，最后一个层级是完全的 matches，这也是之前提到过的不要在外部使用 RouteContext 的原因
                value={{
                    outlet,
                    matches: parentMatches.concat(matches.slice(0, index + 1))
                }}
            />
        );
        // 最内层的 outlet 为 null，也就是最后的子路由
    }, null as React.ReactElement | null);
}

```




## 问题
### 1. react-router 里的 <Link> 标签和 <a> 标签有什么区别?

A: 先看Link点击事件handleClick部分源码

```js
    if (_this.props.onClick) _this.props.onClick(event);

    if (!event.defaultPrevented && // onClick prevented default
        event.button === 0 && // ignore everything but left clicks
        !_this.props.target && // let browser handle "target=_blank" etc.
        !isModifiedEvent(event) // ignore clicks with modifier keys
    ) {
        event.preventDefault();

        var history = _this.context.router.history;
        var _this$props = _this.props,
            replace = _this$props.replace,
            to = _this$props.to;


        if (replace) {
            history.replace(to);
        } else {
            history.push(to);
        }
    }
```
Link做了3件事情：

有onclick那就执行onclick
click的时候阻止a标签默认事件（这样子点击<a href="/abc">123</a>就不会跳转和刷新页面）
再取得跳转href（即是to），用history（前端路由两种方式之一，history & hash）跳转，此时只是链接变了，并没有刷新页面
