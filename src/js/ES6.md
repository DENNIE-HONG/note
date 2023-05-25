# ES6
## 1. 箭头函数与普通函数区别？
1. 函数体内的this对象不是使用时所在的对象
2. 不可当构造函数
3. 不可使用arguments对象
4. 不可使用yield命令.不作Generator函数
5. 没有原型

## 数字
### 浮点数运算问题
例如：0.1 + 0.2 === 0.3 // false
Number.EPSILON表示浮点运算舍入操作的安全误差范围。
```js
function withinMarginOfError(left, right) {
  return Math.abs(left - right) < Number.EPSILON;
}

withinMarginOfError(0.1 + 0.2, 0.3); // true
```

## 模块
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
