# TypeScript
## 编译
过程： 
1. (ts) typescrit源码 -> typescript AST
2. (ts) 类型检查器检查AST
3. (ts) typescript AST -> JavaScript源码
4. (js) JavaScript源码 -> JavaScript AST
5. (js) AST -> 字节码
6. (js) 运行时计算字节码

## 类型全解
### 对象
1. 索性签名：[key: T]: U  
2. 不要Object  
尽量避免Object类型和空对象字面量{}，因为任何类型都可以赋值给空对象类型，Object与{}的作用基本一致，'a'也是一种Object；
```ts
let danger: {}
danger = {}
danger = {x: 1}
danger = []
danger = 2
```
3. Object和{}区别  
使用{}，可以把Object原型内置方法定义为任何类型，Object要求声明的类型必须可赋值给Object原型内置的类型。
```ts
  let a : {} = {toString() { return 3}} // 通过类型检查
  let b: Object = {toString() { return 3}} // Error TS2322:Type 'number' is not assignable to type 'string'
```
### 枚举
1. 显式声明  
一个枚举可以分成几次声明，typescript会自动把各个部分合并在一起。注意，如果分开声明枚举，typescript只能推导出其中一部分的值，因此最好为枚举中的每个成员显式赋值。
2. const enum  
可以通过const enum指定使用枚举的安全子集，不允许反向查找。  
使用尽量避免内插，可以在tsconfig.json中配置preserveConstEnums:true