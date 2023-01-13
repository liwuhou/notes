除去 with 和 eval 语法，this 的指向可分为 4 种：
1. 作为对象的方法调用
2. 作为普通函数调用
3. 构造器调用
4. Function.prototype.call 和 Function.prototype.apply 调用

### 作为对象的方法调用

指向对象

```js
const obj = {
  a: 1,
  getA() {
    return this.a
  }
}
console.log(obj.getA()) // 1
```

### 作为普通函数调用

指向全局

```js
var a = 0

const obj = {
  a: 1,
  getA() {
    return this.a
  }
}
const getA = obj.getA
console.log(getA()) // 0
```

在 node 中是 global，在 es5 的 strict 模式下，指向的是 undefined

### 构造器使用

指向实例出来的对象

```js
const MyClass = function() {
  this.a = 1
}

const obj = new MyClass()

console.log(obj.a) // 1

```

