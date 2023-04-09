# http等
## 1. get/post
除了http层区别，在tcp/ip层面也有区别  
get请求：
```
headers + data  --->   服务器     
tcp数据包       <--- 200
```

post请求：  
```js
headers  ----->    服务器
tcp包   <------
        100 continue

data     ----->    服务器
tcp包    <----
          200
```

## 2. 双向通信
### 2.1 webSocket
允许服务器主动发送信息给客户端，双向数据流协议
与http是两码事。
* 传统TCP socket：标准化的API
* webSocket: 网络协议  

### 2.2 轮询
借助于setInterval等方式，客户端不断发送请求并得到相应。但轮询间隔过长，用户收到消息不及时，轮询间隔太短，增加服务器负担。

### 2.3 长轮询
客户端发出请求后，服务端用while(true)等方式阻塞住请求，直到有可用数据发送响应，又称为“comet”、“反向ajax”。但仍然是基于http的一种慢响应。
### 2.4 HTTP流（基于iframe）
```

           connect
          --------------> 
 _______           push      ________
|client | <--------------   | server |
|_______|           push    |________|
          <---------------
          close connection
          ---------------->
```