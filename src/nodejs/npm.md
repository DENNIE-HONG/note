# NPM
## npm插件包集合
假设App中有"dependencies": 
* A: "1.0.0", A@1.0.0 -> D@1.0.0
* B: "1.0.0", B@1.0.0 -> D@2.0.0
* C: "1.0.0", C@1.0.0 -> D@1.0.0

### npm结构
npm2.x结构
```js
|
|-A@1.0.0
| |_node_modules
|    |_D@1.0.0
|
|-B@1.0.0
| |_node_modules
|    |_D@2.0.0
|
|-C@1.0.0
|  |_node_modules
|     |_D@1.0.0
|
```
层级结构明细，可能有大量冗余问题，嵌套深。

npm3.x ———扁平结构：
新的包安在第一级目录，如遇已存在包，按版本判断，已存在的忽略。
```js
|
|-A@1.0.0
|-D@1.0.0
|-B@1.0.0
| |_node_modules
|    |_D@2.0.0
|
|-C@1.0.0 
```

npm5.x -package-lock.json: 锁定大版本，每次install拉取大版本下的最新版本。

### 依赖

* dependencies: 业务依赖； 
* devDependencies: 开发依赖，不应属于线上代码；
* peerDependencies: 同伴依赖;  
比如element-ui@2.6.3要求宿主环境安装指定的vue。
"peerDependencies": {  
  "vue": "^2.5.16"  
}
* bundledDependencies: 打包依赖；
* optionalDependencies: 可选依赖；

业务项目：dependencies和devDependencies无本质区别，规范作用；
用于发布npm包：dependencies包会在安装该包一起下载。

### 版本包
主版本号.次版本号.修订号(x.y.z)
比如：
* ^1.2.3  等价于  >= 1.2.3 < 2.0.0 即1不变；
* ^0.2.3 等价于 >=0.2.3 <0.3.0 最左为0，只要2不变；
* ^0.0.3 等价于 >=0.0.3 < 0.0.4 
* ~1.2.3： 只兼容补丁更新的版本号；
* ~1.2.3 等价于 >=1.2.3 <1.3.0
* ~1.2 等价于 >=1.2.0 <1.3.0


## 包管理
1. 大版本相同：package.json次版本号 > package-lock.json , install完会更新次版本号，并改写package-lock.json；
2. 大版本号不相同：install后依据package.json大版本下最新，并更新至package-lock.json


## 构建
执行npm run ：
1. 自动新建一个shell；
2. 这个shell将当前项目的node-modules/.bin的绝对路径加入到环境变量path中；
3. 执行结束，再将path恢复原样；

script参数：   
以空格分割的任何字符串，可通过process.argv访问。  
比如:
```js
“vue-cli-service serve --mode-dev --mobile -config build/example.js”
process.argv: [
  'user/local/caller/node/7.7.1_1/bin/node',
  '/user/mac/Vue-projects/hao-cli/node_modules/.bin/vue-cli-service',
  'serve',
  '--mode=dev',
  '--mobile',
  '-config',
  'build/example.js'
]
```
串行执行 && ， 并行执行 &  
执行npm scripts命令都经历了pre和post2个钩子。
如果都定义依次执行：
* npm run prexxx
* npm run xxx
* npm run postxxx
