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