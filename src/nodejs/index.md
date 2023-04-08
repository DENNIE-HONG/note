# NodeJs
## 工作模式
单线程的问题：
1. cpu的利用率
2. 进程的健壮性

解决：  多进程：一个进程一个CPU，child-process  
Master-Workder模式（主从）：主进程负责调度与管理，工作进程负责业务处理。

## 通信
进程间通信：管道技术；
主进程：创建IPC通道并监听；
子进程：文件描述符 -> 连接 -> IPC
## 负载均衡
操作系统的抢占式策略：端口共同监听，TCP服务端socket套接字的文件描述符相同。但是只有一个进程能够抢到连接。

## Node中的事件循环
6个阶段
```
   ┌───────────────────────┐
┌─>│        timers         │setTimeout、setInterval预定的callback
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │某些系统操作的回调
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │执行 setImmediate() 设定的callbacks;
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │比如 socket.on(‘close’, callback) 的callback会在这个阶段执行
   └───────────────────────┘
```
例如：
```js
const fs = require('fs');
fs.readFile(_filename, () => {
  setTimeout(()=> {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
// 输出immediate timeout  
```

readFile回调在poll中进行，发现有setimmedate, 跳到check阶段执行回调，再去timer阶段执行setTimeout