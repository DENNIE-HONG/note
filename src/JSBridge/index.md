# JSBridge

给javascript提供调用Native功能的接口。让混合开发中的【前端部分】可以方便地使用地址信息、摄像头、支付等Native功能。

Native <---> JSBridge <----> JavaScript

```js

                 ____
         ——————>| H5 |-————————
        |        ————          |
      __|______           _____↓___________
     |JSBridge |         |WebViewJavascript|
      —————————          |Bridge.post      |
        ↑                 —————————————————
        |       ______         |
         ——————|Native|<———————
                ——————
```

javascript 调用Native：
* 注入API
* 拦截URL SCHEMA

## 1. 注入API
通过WebView提供的接口，向Javascript的context(window)中注入对象、方法，
即调用时就直接执行相应的Native代码。

## 2. 拦截URL SCHEMA
类似url，protocd和host是自定义，缺点：url长度隐患。




## 接口简易实现
伪代码：

```js
(function(){
    let id = 0;
    const callbacks = {};
    const registerFuns = {};
    window.JSBridge = {
        // 调用Native
        invoke: function(bridgeName, callback, data) {
            var thisId = id++;
            callbacks[thisId] = callback;
            nativeBridge.postMessage({
                bridgeName,
                data: data || {},
                callbckId: thisId
            });
        },
        receiveMessage: function(msg) {
            const {bridgeName} = msg;
            const {data = {}, callbackId, responseId} = msg;
            if (callbackId) {
                if (callbacks[callbackId]) {
                    // 执行调用
                    callbacks[callbackId](msg.data);
                }
            } else if (bridgeName) {
                if (registerFuns[bridgeName]) {
                    let ret = {};
                    let flag = false;
                    registerFuns[bridgeName].forEach((callback) => {
                        callback(data, function(r)) {
                            flag = true;
                            ret = Object.assign(ret, r);
                        });
                    });
                    if (flag) {
                        nativeBridge.postMessage({
                            responseId,
                            ret
                        });
                    }
                }
            }
        },
        register: function(bridgeName, callback) {
            if (!registerFuns[bridgeName]) {
                registerFuns[bridgeName] = [];
            }
            registerFuns[bridgeName].push(callback);
        }
    };
})();

```
