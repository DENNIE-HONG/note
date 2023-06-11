# ES6
## 1. 箭头函数与普通函数区别？
1. 函数体内的this对象不是使用时所在的对象
2. 不可当构造函数，不能通过new关键字调用。
js函数内部2个方法：[[call]]和[[constructor]]。当通过new调用函数，执行[[constructor]]方法，创建一个实例对象，然后执行函数体，将this绑定到实例上。箭头函数并没有[[constructor]]方法。
3. 不可使用arguments对象
4. 不可使用yield命令.不作Generator函数
5. 没有原型,不存在prototype属性；
6. 没有super：不能通过super来访问原型的属性；

## 2. 数字
### 浮点数运算问题
例如：0.1 + 0.2 === 0.3 // false
Number.EPSILON表示浮点运算舍入操作的安全误差范围。
```js
function withinMarginOfError(left, right) {
  return Math.abs(left - right) < Number.EPSILON;
}

withinMarginOfError(0.1 + 0.2, 0.3); // true
```

## 3. 模块
模块公开API中暴露的属性和方法并不仅是普通的值或者引用的赋值。是内部模块定义中的标识符的实际绑定（几乎类似于**指针**）  
即使导出一个私有变量是字符串类型。导出的都是这个变量的绑定，模块修改了这个变量，外部导入绑定到新的值。不是复制！！！

**默认导出的陷阱**
```js
function foo(...) {
  ...
}
export default foo;
```
导出的是函数表达式绑定。如果foo=xxx,模块导入得到的仍然是原来导出的函数。
2. export {foo as default};
导出绑定绑定到foo标识符而不是它的值，如修改了foo,导入一侧也更新！！

**注意**：
除了export default 导出一个表达式绑定，其他：导出局部标识符的绑定。在导出之后，在模块内部修改某个值，外部导入的绑定会访问到新的值！
```js
import * as hello from 'world';
hello.default = 42; //typeError!
hello.bar = 42; // typeError
```
局部导入绑定的只读性限制了无法从导入的绑定修改！

## 4. Symbol
#### Symbol.toStringTag
原型的@@toStringTag符号指定了在[object ---]字符串化时使用的字符串值。
例如：
```js
function Foo() {}
const a = new Foo();
a.toString(); // [object Object]
a instanceof Foo; // true

// 改
Foo.prototype[Symbol.toStringTag] = "Foo";
const b = new Foo();
a.toString(); // [object Foo]

```

#### Symbol.hasInstance
@@hasInstance符号是在构造器函数上的一个方法，接受实例对象值，通过返回true/false来表示这个值是否可以被认为是一个实例。
例如：
```js
function Foo(greeting) {
  this.greeting = greeting;
}
Object.definePropery(Foo, Symbol.hasInstance, {
  value: function(inst) {
    return inst.greeting === 'hello';
  }
});
const a = new Foo('hello');
const b = new Foo('world');
a instanceof Foo; // true
b instanceof Foo; // false
```

#### Symbol.isConcatSpreadable
用来指示如果传给一个数组的concat(...), 是否应该将其展开。
例如：
```js
const a = [1, 2, 3];
const b = [4, 5, 6];
b[Symbol.isConcatSpreadable] = false;
[].concat(a, b); // [1, 2, 3, [4, 5, 6]]
```


## 5. class
### es6写的类与es5写的‘类’不同点?
1. 类的内部所有定义的方法都是不可枚举的！
```js
// es6
Object.keys(Person.prototype); // []
// es5
function Person(name) {
  this.name = name;
}
Person.prototype.sayHello = function() {
  ...
}
Object.keys(Person.prototype); //['sayHello']
```

2. 必须new调用，否则报错

## 6. Set与weakSet
区别？
### Set
* 成员不能重复；
* 只有健值，无健名，类似数组；
* 可遍历；

### weekSet
* 成员都是对象；
* 成员都是弱引用，随时消失；
* 不能遍历；

弱引用：
指不能确保其引用对象不会被垃圾回收器回收的引用。一个对象若被弱引用，则认为是不可访问的，可能在任何时刻被回收。  
例如：
```js
let map = new Map(); 
let key = new Array(5 * 1024 * 1024);
map.set(key, 1); // 强引用
key = null; // 不会导致key引用对象被回收
// 需要这样才回收
map.delete(key);
key = null;
// 当
map = new WeakMap();
...
// 下次回收执行时，该引用对象会被回收
key = null;
```
一旦不再需要，WeakMap里的健名对象和健值会自动消失，不用手动删除引用。故WeakMap内部成员取决于垃圾回收机制有没有运行，故不可遍历。


## 7. 模板字面量
使用标记的字符串模板，第一个参数的值始终是字符串值数组，其余参数获取传递到模板字符串中的表达式值。

例如：
```js
function getPersonInfo(one, two, three) {
  console.log(one);
  console.log(two); 
  console.log(three);
}

const person = "Lydin";
const age = 21;
getPersonInfo`${person} is ${age} years old`;
// ["", "is", "", "years old"]
// Lydin
// 21
```
