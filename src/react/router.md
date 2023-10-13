# react-router

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
