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
提取原数组中索引start到end的元素。包括start，不包括end。end被省略，slicet默认到数组末尾
### splice
修改原数组，返回删除元素组成的数组。
```js
const a = [1, 2, 3, 4, 5, 6, 7, 8];
a.splice(4); // 返回[5, 6, 7, 8]
console.log(a); // [1, 2, 3, 4]
```

模拟实现：
```js
Array.prototype._splice = function (start, deleteCount, ...addList) {
    if (start < 0) {
        if (Math.abs(start) > this.length) {
            start = 0;
        } else {
            start += this.length;
        }
    }

    if (typeof deleteCount === 'undefined') {
        deleteCount = this.length - start;
    }

    const removeList = this.slice(start, start + deleteCount);

    const right = this.slice(start + deleteCount);

    let addIndex = start;
    addList.concat(right).forEach(item => {
        this[addIndex] = item
        addIndex++;
    })
    this.length = addIndex;

    return removeList
}
```






### shift
用于把数组的第一个元素从其中删除，并返回第一个元素的值;
易错题：
```js
const a = [[1], [2], [3]];
const b = [...a];
b.shift().shift();
console.log(a);
// [[], [2], [3]]
```




### map
返回新数组

<font color="red">1.循环实现数组map方法：</font>

```js
const selfMap = function(fn, context) {
  let arr = Array.prototype.slice.call(this);
  let mappedArr = Array();
  for (let i = 0; i < arr.length; i++) {
    // 判断稀疏数组情况
    if (!arr.hasOwnProperty(i)) {
      continue;
    }
    mappedArr[i] = fn.call(context, arr[i], i, this);
  }
  return mapperArr;
}
Array.prototype.selfMap = selfMap;
```

<font color="red">2.使用reduce实现数组map方法：</font>

```js
const selfMap2 = function(fn, context) {
  let arr = Array.prototype.slice.call(this);
  return arr.reduce((pre, cur, index) => {
    return [...pre, fn.call(context, cur, index, this)];
  }, []);
}
```



### push
push原理例子
```js
let obj = {"1": 'a', '2': 'b', length: 2};
Array.prototype.push.call(obj, 'c');
console.log(obj);

// 解,push的原理
Array.prototype.push = function(target) {
  this[this.length] = target;
  this.length++;
}
// obj[2] = c, length = 3
// {"1": 'a', "2": 'c', length: 3}
```

### reduce

<font color="red">循环实现数组的reduce方法：</font>

```js
Array.prototype.selfReduce = function(fn, initialValue) {
  let arr = Array.prototype.slice.call(this);
  let res;
  let startIndex;
  if (initialValue === undefined) {
    // 找到第一个非空单元的元素和下标
    for (let i = 0; i < arr.length; i++) {
      if (!arr.hasOwnProperty(i)) continue;
      startIndex = i;
      res = arr[i];
      break;
    }
  } else {
    res = initialValue;
  }
  for (let i = ++startIndex || 0; i < arr.length; i++) {
    if (!arr.hasOwnProperty(i)) continue;
    res = fn.call(null, res, arr[i], i , this);
  }
  return res;
}
```


### for of
循环可通过break、continue、return 提前终止。



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

### Array.from
Array.from的映射功能：第2个参数是一个映射回调；
举个🌰：
```js
const arrLike = {
  length: 4,
  2: "foo"
};
Array.from(arrLike, function mapper(val, idx) => {
  if (typeof val === "string") {
    return val.toUpperCase();
  } else {
    return idx;
  }
});
// [0,1,"FOO",3]
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

公司：滴滴、快手、携程

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

### 去重方法
例子：用reduce

```js
Array.prototype.unique = function() {
    return this.sort().reduce((init, current) => {
        if (init.length === 0 || init[init.length-1] !== current) {
            init.push(current);
        }
        return init;
    }, []);
}
```

例子：用map方法

```js
Array.prototype.unique = function() {
    const tmp = new Map();
    return this.filter((item) => {
        return !tmp.has(item) && tmp.set(item, 1);
    });
}
```

