# TypeScript

## 1. 特性

|ts       | js|
|---------|---|
|js的超集|一种脚本语言|
|编译期发现并纠正错误|只在运行时发现错误|
|强类型|弱类型|
|最终编译成js|可直接在浏览器中使用|
|支持模块、泛型、接口|不支持|
|社区仍在成长|成熟|

## 2. 编译
过程： 
1. (ts) typescrit源码 -> typescript AST
2. (ts) 类型检查器检查AST
3. (ts) typescript AST -> JavaScript源码
4. (js) JavaScript源码 -> JavaScript AST
5. (js) AST -> 字节码
6. (js) 运行时计算字节码

## 3. 类型全解


$$
ts的基础类型
\begin{cases}
boolean\\
number\\
string\\
Array：number[] or Array<number>\\
Enum: 枚举带名字的常量\\
Any: 可执行任何操作\\
Unknow: 可任意赋值，只能被赋值给any类型和unknow类型\\
Tuple: 单个变量存储不同类型\\
Void\\
Null和Undefined\\
object、Object类型和对象类型\\
Never
\end{cases}
$$



### 枚举
1. 显式声明  
一个枚举可以分成几次声明，typescript会自动把各个部分合并在一起。注意，如果分开声明枚举，typescript只能推导出其中一部分的值，因此最好为枚举中的每个成员显式赋值。
2. const enum  
可以通过const enum指定使用枚举的安全子集，不允许反向查找。  
使用尽量避免内插，可以在tsconfig.json中配置preserveConstEnums:true





### Any类型（顶级类型）
例如：
```ts
let notSure: any = 666;
notSure = "semiler"; // 任何操作
let value: any;
value.foo.bar; // ok
value.trim(); // ok
value(); // ok

```





### Unkonw类型
所有值也可赋值给unknow, 但不可赋值给其他string等类型。  
举个🌰：
```ts
let value:unkonw;
value.foo.bar; // Error
value.trim(); // Error
value(); //Error  禁止任务更改
```






### 对象

#### object类型
ts2.3引入的新类型，用于表示非原始类型

举个🌰：
```ts
interface ObjectConstructor {
    create(o: object | null): any;
    ...
}

const proto = {};
Object.create(proto); 
Object.create(null);
Object.create(1337); // x
```

#### Object类型
所有object类的实例的类型。
* Object接口定义了Object.prototype原型对象上属性；
* ObjectConstructor接口定义了object类的属性；

```ts
interface Object {
    constructor: Function;
    toString(): string;
    valueOf(): Object;
    toLocalString(): string;
    ...
}

interface ObjectConstructor {
    new(value ?: any): Object;
    (value ?: any): any;
    ...
}
```


#### {}类型
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




### Never类型
永不存在的值的类型。
举个🌰：

```ts
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}
```

可利用never特性实现全面性检查。
```ts
type Foo = string | number;
function controllAnalysisWithNever(foo: FOO) {
    if (typeof foo === "string") {
        //...
    } else if (typeof foo === "number") {
        // ...
    } else {
        // foo在这里never
        const check:never = foo;
    }
}
```


### 其他
#### 联合类型：
```ts
// 用来约束取值只能是其中一个
let num: 1|2 = 1;
type EventNames = 'click' | 'scroll' | 'mousemove';
```

ts2.1 引入keyof操作符，返回类型是联合类型

```ts
interface Person {
    name: string;
    age: number;
}
type k = keyof Person; // "name" | "age"

```

#### 交叉类型
通过&可将多种类型叠加到一起成为一种类型。

```ts
type PartialPointX = {x: number};
type Point = PartialPointX & {y: number};
```





## 4. 类和接口

#### 接口与类型别名的区别
1. 都可以：描述对象和函数
2. 都可以扩展：
3. 不同：类型别名可用于联合类型、元祖、原始类型；
    1. 类不能实现使用类型别名定义的联合类型

    ```ts
    type PoartialPoint = {x: number} | {y: number};
    class SomePartialPoint implements PartialPoint {
        x = 1；// error
        y = 2; // error
    }
    ```
    2. 接口可以定义多次，自动合并为单个接口


### 装饰器

#### 类装饰器

```ts
declare type ClassDecorator = <TFunction extends Function> (target: TFunction) => TFunction | void;

function Greeter(target: Function): void {
    target.prototype.greet = function(): void {
        console.log("hello");
    };
}
@Greeter
class Greeting {
    constructor() {
        ...
    }
}

let myGreeting = new Greeting();
(myGreeting as any).greet(); // "hello"
```


#### 属性装饰器

```ts
function logProperty(target: any, key: string) {
    delete target[key];
    const backingField = "_" + key;
    Object.definedProperty(target, backingField, {
        writable: true,
        enumerable: true,
        configurable: true
    });
    // property getter
    const getter = function(this: any) {
        const currVal = this[backingField];
        console.log(`Get: ${key} => ${currVal}`);
        return currVal;
    };
    // property setter
    const setter = function (this: any, newVal: any) {
        console.log(`Set: ${key} => ${newVal}`);
        this[backingField] = newVal;
    };

    Object.definedProperty(target, key, {
        get: getter,
        set: setter,
        enumerable: true,
        configurable: true
    });
}

class Person {
    @logProperty
    public name: string;
    // 定义了logProperty来跟踪用户对属性的操作
    constructor(name: string) {
        this.name = name;
    }
}

const p1 = new Person("semlinber");
p1.name = "kaku";
// Set: name => semlinber
// Set: name => kaku
```


#### 方法装饰器

$$
方法装饰器
\begin{cases}
target \\
propertyKey \\
descriptor
\end{cases}
$$

```ts
function log(target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
    let originalMethod = descriptor.value;
    descriptor.value = function(...args: any[]) {
        console.log("wrapped function: before");
        let result = originalMethod.apply(this args);
        console.log("wrapped function after");
        return result;
    }
}

class Task {
    @log
    runTask(arg: any):any {
        console.log("runTask invoked.");
        return "finished";
    }
}

let task = new Task();
let result = task.runTask("learn");
console.log("result" + result);
// wrapped function before
// runTask invoked
// wrapped function after
// result finished
```
