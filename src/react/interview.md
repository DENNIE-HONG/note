# 常问面试问题

## state和props区别？

## context缺点？
共享状态。


## react函数组件的闭包陷阱？
Hooks 函数也是闭包，它们可以访问定义在函数外部的变量。React Hooks 的闭包陷阱与普通 JavaScript 中的闭包陷阱类似

react Hooks 中的闭包陷阱主要会发生在两种情况：
* 在 useState 中使用闭包；
* 在 useEffect 中使用闭包。

### useState 中的闭包陷阱

举个例子：

```js
function Counter() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setTimeout(() => {
      setCount(count + 1);
    }, 1000);
  };
  const handleReset = () => {
    setCount(0);
  };
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}

```
在上面的代码中，我们定义了一个handleClick函数，它使用了一个闭包来缓存count的值。每次点击触发 handleClick 函数，1s 后 setCount 会将 count 值加 1，在这 1s 内，无论我们点击多少次 Increment 按钮，count 值都只会加 1, 即一直是2。  
这是因为 setCount 所接收的 count 值是在闭包中被缓存的 count 值，这个值是始终不变的。1s 后 setTimeout 中的 setCount 生效，函数式组件 Counter 重新执行，会生成新的 handleClick 方法，形成新的闭包，此时闭包中缓存的 count 值也就变成了最新的 count 值。再继续点击 Increment 按钮，又会重复上述循环，每过 1s count 值会加 1.


解决方法：
1. 直接使用 useState 的第二个参数，即 setCount 的回调函数：
```js
const handleClick = () => {
  setTimeout(() => {
    setCount(currentCount => currentCount + 1);
  }, 1000);
};

```
即setCount函数，可以接受一个回调函数作为参数。这个回调函数会接受当前state的值作为参数，然后返回一个新的state值。React会使用这个新的state值来更新组件的状态。


2. 使用useRef

```jsx
const Test = () => {
    const countRef = useRef(0);
    const [count, setCount] = useState(0);

    const handleClick = () => {
        setTimeout(() => {
            setCount(countRef.current++);
        }, 1000);
    };

    return (
        <div onClick={handleClick}>{count}</div>
    );
}
```




### useEffect闭包陷阱

举个例子：

```js
function App() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
    }, 1000);
    return () => clearInterval(timer);
  }, []);
  
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

```
由于 useEffect 只会在组件首次渲染时执行一次，因此闭包中的 count 变量始终是首次渲染时的变量，而不是最新的值。


避免方法
为了避免这种闭包陷阱，可以使用 useEffect Hook 来更新状态。例如，以下代码中，通过 useEffect Hook 来更新 count 的值，就可以避免闭包陷阱：

```js

useEffect(() => {
  const timer = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(timer);
}, [count]);


```


## 父组件更新，子组件也更新？
利用memo
