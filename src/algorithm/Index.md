# 简单算法
## 1. 排序
### 冒泡排序
每一项与后一项对比，o(n^2)

### 选择排序
找到数组最小值，放在第一位，...以此类推。o(n^2)

```js
function selectSort(arr) {
  const len = arr.length;
  let temp = null;
  let minIndex = null;
  for (let i = 0; i < len - 1; i++) {
    // 假设当前索引为最小索引
    minIndex = i;
    for (let j = i +1; j < len; j++) {
      // 比假设值还小
      if (arr[j] < arr[minIndex]) {
        // 找到最小索引位置
        minIndex = j; 
      }
    }
    // 当前项值和最小值交换位置
    if (i !== minIndex) {
      temp = arr[i];
      arr[i] = arr[minIndex];
      arr[minIndex] = temp;
    }
  }
  return arr;
}


```

### 插入排序
o(n^2)，类似打扑克牌排大小
思路：将第一个元素视为有序序列，遍历数组，将之后元素依次插入这个构建的有序序列中。
```js
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    const element = arr[i];
    for (var j = i - 1; j >= 0; j--) {
      const tmp = arr[j];
      if (tmp > element) {
        arr[j + 1] = tmp;
      } else {
        break;
      }
    }
    arr[j+1] = element;
  }
  return arr;
}

```

### 归并排序
数组分成足够小，合并2个有序数组，o(nlogn)
1. 把长度为n的输入排序分成2个长度为n/2的子序列；
2. 对这2个子序列分别归并排序；
3. 子序列合成一个最终排序；

```js

function merge(left, right) {
  const result = [];
  whilre(left.length && right.length) {
    if (left[0] <= right[0]) {
      result.push(left.shift());
    } else {
      result.push(right.shift());
    }
  }
  while(left.length) {
    result.push(left.shift());
  }
  while(right.length) {
    result.push(right.shift());
  }
  return result;
}
function mergeSort(arr) {
  const len = arr.length;
  if (len < 2) {
    return arr;
  }
  const middle = Math.floor(len / 2);
  const left = arr.slice(0, middle);
  const right = arr.slice(middle);
  return merge(mergeSort(left), mergeSort(right));
}
```



### 快排
时间复杂度：o(nlogn)  
快排三步： 
1. 任选一个元素作为“基准”
2. 将小于“基准”和大于“基准”元素分别放到2个新数组中，等于“基准”的元素可以放在任一数组
3. 对于2个新数组不断重复1.2步，直到数组只剩下一个元素，这时step2的数组已经有序（leftArray + 基准元素 + rightArray）

```js
function quickSort(a) {
  if (a.length <= 1) {
    return a;
  }
  const mid = Math.floor(a.length / 2);
  const midItem = a.splice(mid, 1)[0];
  const left = [];
  const right = [];
  a.forEach((item) => {
    if (item <= midItem) {
      left.push(item);
    } else {
      right.push(item);
    }
  });

  const _left = quickSort(left);
  const _right = quickSort(right);
  return _left.concat(midItem, _right);
}
```
优化？用一句话写快排？
```js
function quickSort(a) {
  return a.length <= 1 ? a : quickSort(a.slice(1).filter((item) => item <= a[0])).concat(a[0], quickSort(a.slice(1).filter((item) => item > a[0])));
}
```


### 桶排序原理
n个数据，m个桶，每个桶有k = n / m 个元素。
每个桶快排，时间复杂度o(klogk), m个桶o(m*klogk) = o(n* log n/m), 当n约定于m时，趋向于 o(n).

适用：数据存在外部磁盘中，数据大而内存有限。10G订单按照金额排序，内存几百MB?
1. 扫描下确定金额范围，比如1万-10万之间，分到100个桶中，[1, 1000],[1001, 2000] ...每个桶对应一个文件。
2. 金额分布均匀，每个文件大约100M数据，每个小文件读入内存，快排按顺序读取数据，写入另外一个文件中，即排好的顺序了。


### 基数排序原理
如有10万个手机号进行排序，怎么做到时间复杂度为o(n) ?
手机号a,b, a手机号前几位比b大，后面几位就不用看了。
从手机号码最后一位开始，按每一位数字对手机号码排序，依次往前，经11次排序。  
举个例子：
```js
h k e     i b a   h a c    i b a
i b a     h a c   i b a    i k f
h z g ->  h k e-> h k e -> h a c
i k f     i k f   i k f    h k e
h a c     h z g   h z g    h z g

```

## 2. 乱序
### 普通版
```js
function shuffle(arr) {
  return arr.sort(() => Math.random() - 0.5);
}
```
问题：并不是真正随机
v8处理sort时，当目标数组长度 < 10时使用插入排序，反之使用快速排序。
元素之间比较次数 << n*(n-1)/2, 某些元素之间没比较的机会（也就无交换的可能）。

### 真正的乱序(Fisher-Yates算法)
1. 排序好数组：1 2 3 4 5 6 7 8 9
2. 末尾开始，选最后一个数：1 2 3 4 5 6 7 8 「9」
3. 在9个位置中随机选中一个位置，与最后一个元素交换。
```js
1 2 [9] 4 5 6 7 8 [3]
     \____________/
```
4. 对倒数第2个元素，除去最后一个位置，再随机选一个变换；
```js
 [8] 2 9 4 5 6 7 [1] 3
  \______________/
```
5. 以此类推；
```js
function shuffles(arr) {
  let m = arr.length;
  while(m > 1) {
    let index = Math.floor(Math.random() * m--);
    [arr[m], arr[index]] = [arr[index], arr[m]];
  }
  return arr;
}
```