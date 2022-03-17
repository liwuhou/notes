事情得先从一道面试题说起

```js
const arr = [1, 2, 3]

const square = (x) => new Promise(resolve) => setTimeout(() => resolve(x * x), 1000))

const test = async () => {
  arr.forEach(async (x) => {
    const res = await square(x)
    console.log(res)
  })
}

// 问：改写 test，让输出能间隔 1s 输出
```


首先，**不要在 `forEach` 和 `map` 方法内使用 `async`、`await`**，因为 `forEach` 传入的回调中没有等待回调方法执行完就继续执行下个循环的回调了。

分析下 `forEach` 内部的大致原理

```js
Array.prototype.forEach = function(cb) {
  for (let i=0; i < this.length; i++) {
    cb(this[i], i, this)
  }
}

Array.prototype.map = function(cb) {
  const o = []
  for (let i =0; i < this.length; i++) {
    const temp = cb(this[i], i, this)
    o.push(temp)
  }
  return o
}
```

这里，`forEach` 中传入的的 `cb`，即使是个异步函数也不会等他执行完，因此造成的结果是，js 线程等了 1s 之后就把结果都打印出来了。这里不用 `forEach`，直接改用 `for...in` 或者 `for...of` 就可以了

```js
const test = async() => {
  for(const x of arr) {
    const res = await square(x)
    console.log(res)
  }
}
```