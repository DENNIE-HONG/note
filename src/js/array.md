# 数组
## 1 数组方法
注意：
* 当every()遍历数组为空时，也返回true
* forEach、map、filter、every、some作用于稀疏数组时，不会在实际上不存在的元素调用：比如
```
const array = Array(3); //并没有元素
```

### concat
返回新数组
### slice
返回新数组
### splice
修改原数组，返回删除元素组成的数组。
```js
const a = [1, 2, 3, 4, 5, 6, 7, 8];
a.splice(4); // 返回[5, 6, 7, 8]
console.log(a); // [1, 2, 3, 4]
```
### map
返回新数组

### push
push原理例子
```js
let obj = {"1": 'a', '2': 'b', length: 2};
Array.prototype.push.call(obj, 'c');
console.log(obj); 

// 解,push的原理
Array.prototype.push = function(target) {
  obj[target.length] = target;
  obj.length++;
}
// obj[2] = c, length = 3
// {"1": 'a', "2": 'c', length: 3}
```



## 2. 稀疏数组
数组直接量允许有可迭的结尾逗号，[,,]是2个元素不是3个。  
稀疏数组与省略数组区别：
```js
const a1 = [,,,]; // [undefined, undefined,undefined]
const a2 = new Array(3); // 数组没元素
O in a1; // true
0 in a2; // false
```
避免空槽值：
```js
const c = Array.from({length: 4});
```

## 3. 类数组
拥有数值length属性，非负整数属性的对象。数组方法可用在类数组上。
例如：
```js
const a = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};
console.log(Array.prototype.join.call(a, '+')); // 'a+b+c' 
```
注意：字符串可看作类数组，但其是只读的，push()、sort()、reverse()、splice()等改变原数组方法在字符串上无效！
```js
const s = "JavaScript";
console.log(Array.prototype.join.call(s, ' ')); // J a v a S c r i p t
```

## 数组小式一笔
### 扁平化
数组展开？
```js
function flatten(arr) {
  return [].concat(
    ...arr.map(x => Array.isArray(x) ? flatten(x): x)
  );
}
```

### 创建数组
例子：不用循环，创建一个长度为100，每个元素等于下标的数组？
1. 递归
```js
function createArray(len, arr = []) {
  if (len > 0) {
    arr[--len] = len;
    createArray(len, arr);
  }
  return arr;
}
```
2. 
```js
 [...Array(100).keys()]
```