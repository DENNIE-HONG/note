# redux

## 1. Flux模式
MVC架构的问题，随着项目越来越复杂，数据流动很混乱，所以有了Flux架构。
数据和逻辑永远是单向流动

```js
    _______________calls______________
    |                                 |
 __ ↓__     __________     _____     _|__
|Action|——>|dispatcher|——>|store|——>|view|
 ——————     ——————————     —————     ————

```
* dispatcher: 分发事件；
* store: 保存数据，响应事件更新数据；
* view: 订阅store中数据，渲染页面；

用户在view上的操作最终会映射为一类Action，Action传递给Dispatcher,再由Dispatcher执行注册在指定Action上的回调函数。最终完成对Store的操作，store中数据变化，view监听并作出反应。



## 2. Redux简介
Redux参考了Flux的设计，但是对Flux许多冗余部分（dispatcher）做了简化。提供若干API让我们使用reducer创建store，并能够更新store中的数据或获取store中的最新的状态。

## 3. Redux 三大原则
1. **单一数据源**
一个应用永远只有唯一的数据源。
2. **状态是只读的**。
对直接修改store的状态限制的更加彻底，定义一个reducer，根据当前触发的action，对state进行迭代，并没有修改应用状态，而是返回了一份全新的状态。
3. **状态修改均有纯函数完成。**
每一个reducer都是纯函数，意味着没有副作用，接收一定的输入，必定会得到一定的输出，，使得reducer对状态的修改变得简单、纯粹、可测试。


## 4. Redux middleware(重要)
由来：希望dispatch或reducer拥有异步请求的功能。需要可以组合的、自由插拔的插件机制，redux借鉴了koa里middleware的思想。通过串联不同的middleware实现变化多样的功能。

应用middleware后Redux处理事件的逻辑：
```js
                  _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ __ _
                 |          new dispatch                      |
 ______ callback |   ____     ____ action  ____     ________ action
|button|—————————|—>|mid1|——>|mid2|——————>|... |——>|dispatch|——————>|reducer|
 ——————          |   ————     ————         ————     ————————  |
                 |_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ __ _|
```


### 4.1 middleware机制

精炼的源码：

```js
function compose(...funcs) {
    return arg => funcs.reduceRight((composed, f) => f(composed), arg);
}
```
```js
import compose from './compose';
export default function applyMiddleware(...middlewares) {
    return (next) => (reducer, initialState) => {
        let store = next(reducer, initialState);
        let dispatch = store.dispatch;
        let chain = [];
        let middlewareAPI = {
            getState: store.getState,
            dispatch: (action) => dispatch(action)
        };
        // chain数组 [f1, f2, ..., fn]
        chain = middlewares.map(middleware => middleware(middlewareAPI));
        // 新dispatch执行，chain数组从右到左依次执行
        dispatch = compose(...chain)(store.dispatch);
        // 当compose执行完，假设n=3，dispath = f1(f2(f3(store.dispatch)));
        return {
            ...store,
            dispatch
        };
    }
}
```

1. 函数式编程思想
利用柯里化，好处：易串联和共享store。
2. 给middleware分发store
创建普通的store：
    ```js
    let newStore = applyMiddleware(mid1, mid2, mid3, ...)(createStore)(reducer, null);
    ```
    每个匿名函数都可以访问相同的store，即middlewareAPI.
3. 组合串联middleware


Q: 在middleware中调用store.dispatch()会发生什么？和调用next()有区别吗？

A：两者的不同：
```js
const logger = store => next => action => {
    console.log('dispatch', action);
    next(action);
    console.log('finish', action);
}
```
```js
const logger = store => next => action => {
    console.log('dispatch', action);
    store.dispatch(action);
    console.log('finish', action);
}
```
在middleware中调用next(), 效果是进入下一个middleware，如下图所示。在middleware中调用store.dispatch()和在其他任何地方调用的效果一样。
![Redux middleware 流程图](../../public/redux-middleware.jpeg)


### 4.2 Redux异步流
常用3个middleware来介绍发异步请求。

#### 4.2.1 redux-thunk
理论上发异步请求是在action，可是action都是同步情况，如何支持？
**Thunk函数**：针对多参数的柯里化，以实现对函数的惰性请求。只要参数有回调函数，都可以写Thunk函数形式。

**redux-thunk源码**：
```js
function createThunkMiddleware(extraArgument) {
    return ({dispatch, getState}) => next => action => {
        // 这里action即为一个Thunk函数
        if (typeof action === 'function') {
            return action(dispatch, getState, extraArgument);
        }
        return next(action);
    };
}
```

![Alt text](../../public/example.gif)

模拟请求一个天气的异步请求。action可以这么写：

```js
function getWeather(url, params) {
    return (dispatch, getState) => {
        fetch(url, params)
            .then(result => {
                dispatch({
                    type: 'GET_WEATHER_SUCCESS',
                    payload: result
                });
            })
            .catch(err => {
                dispatch({
                    type: 'GET_WEATHER_ERROR',
                    error: err
                });
            });
    };
}
```

#### 4.2.2 redux-promise
异步请求都是利用promise，可以抽象promise来解决异步流的问题。
源码：
```js
import {isFSA} from 'flux-standard-action';

function isPromise(val) {
    return val && typeof val.then === 'function';
}

export default function promiseMiddleware({dispatch}) {
    return next => action => {
        //
        if (!isFSA(action)) {
            return isPromise(action) ? action.then(dispatch): next(action);
        }
        // 判断action.payload是否为promise，是就执行then，返回的结果再发送一次dispatch
        return isPromise(action.payload) ?
        action.payload.then(
            result => dispatch({...action, payload: result}),
            error => {
                dispatch({...action, payload: error, error: true});
                return Promise.reject(error);
            }
        )
        : next(action);
    };
}
```
![Alt text](../../public/example.gif)

模拟请求一个天气的异步请求。action可以简化写：

```js
const fetchData = (url, params) => fetch(url, params);

async function getWeather(url, params) {
    const result = await fetchData(url, params);
    if (result.error) {
        return {
            type: 'GET_WEATHER_ERROR',
            error: result.error
        };
    }
    return {
        type: 'GET_WEATHER_SUCCESS',
        payload: result
    };
}
```

#### 4.2.3 多异步串联
sequenceMiddleware源码：

```js
const sequenceMiddleware = ({dispatch, getState}) => next => action => {
    if (!Array.isArray(action)) {
        return next(action);
    }
    // 每个action都按照顺序执行
    return action.reduce((result, currAction) => {
        // 使用Promise.then方法串联数组
        return result.then(() => {
            return Array.isArray(currAction) ?
                Promise.all(currActoin.map(item) => dispatch(action))
                : dispatch(currAction);
        });
    }, Promise.resolve());
}
```

![Alt text](../../public/example.gif)
假设先获取当前城市，再获取当前城市天气：
```js
function getCurrCity(ip) {
    return {
        url: '/api/getCurrCity.json',
        params: {ip},
        types: [null, 'GET_CITY_SUCCESS', null]
    };
}

function getWeather(cityId) {
    return {
        url: '/api/getWeatherInfo.json',
        params: {cityId},
        types: [null, 'GET_WEATHER_SUCCESS', null]
    };
}

function loadInitData(ip) {
    return [
        getCurrCity(ip),
        (dispatch, state) => {
            dispatch(getWeather(getCityIdWithState(state)));
        }
    ];
}
```