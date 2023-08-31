# nginx
nginx在应用中的作用：
* 解决跨域；
* 请求过滤；
* 配置gzip；
* 负载均衡；
* 静态资源服务器

## nginx.conf
* Events
* http
  * upstream
  * server
  * server
    * location
  
  
upstream: 配置后端服务器具体地址，负载均衡。  
location： 配置请求路由，以及页面处理。  
例如：跨域
前端：fe.server.com
后端: dev.server.com

```conf
server {
  listen 80;
  Server_name fe.server.com;
  location / {
    proxy_pass dev.server.com;
  }
}
```
负载均衡：
```conf
upstream balanceServer {
  server 10.1.22.33:12345;
  server 10.1.22.34:12345;
  ...
}
server {
  server_name fe.server.com;
  listen 80;
  location / api {
    proxy_pass http://balanceServer;
  }
}
```

## nginx负载均衡策略
### 1. 轮询策略
默认，将所有客户端请求分配给每个服务器，依次分配，不考虑服务器压力。
### 2. 最小连接数策略
优先分给压力小的服务器，平衡队列长度。
```conf
upstream balanceServer {
  least_conn;
  server 12.1.22.33:12345;
  ...
}
```
### 3. 最快响应时间策略
依赖nginxPlus，优先分配给响应时间最短的服务器。
```conf
upstream balanceServer {
  fair;
  ...
}
```
### 4. 客户端IP绑定
同一个ip的请求永远值分配一台服务器。有效解决动态网页存在的session共享问题。
```conf
upstream balanceServer {
  ip_hash;
  server ...
}
```
