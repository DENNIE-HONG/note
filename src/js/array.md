# 数组
## 数组方法
注意：
* 当every()遍历数组为空时，也返回true
* forEach、map、filter、every、some作用于稀疏数组时，不会在实际上不存在的元素调用：比如
```
const array = Array(3); //并没有元素
```
