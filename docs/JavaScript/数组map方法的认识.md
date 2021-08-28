### 前言

### map方法的基本用法

> MDN上对map方法的介绍是：**map() 方法创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。**

从官方的介绍中我们可以了解到，map方法**不会改变**调用它的数组（但是我们可以在提供的回调函数中修改），而是返回一个新的数组。



#### 举个例子：

- 对数组中的元素进行处理

```javascript

let arr = [1, 2, 3]
let mappedArr = arr.map(value => {
  return value + 1
})
console.log(mappedArr) // [2, 3, 4]

```

### 原理剖析
通过上面的例子，我们大概知道了map函数工作基本工作原理了。什么？就这能知道工作原理了？

额，那我们进一步剖析。


```javascript

let mappedArr = arr.map(function callback(currentValue[, index[, array]]) {
 // Return element for new_array 
}[, thisArg])

```

> map方法可以接受两个参数：第一个参数`callback`是一个函数，这个参数是必需的。`callback`中有三个可选参数，分别是当前的正在处理的数组元素、当前处理元素的索引、调用map方法的数组；第二个参数是可选的，它的作用是用来定义执行`callback`时的this指向。

知道了各个参数的作用之后，就可以搞事情了。

#### 实现基本版的自定义map

```javascript

Array.prototype._map = function (callback, thisArg) {
  let len = this.length
  let result = []
  for (let i = 0; i < len; i++) {
    // 通过调用call方法,将callback的this指向thisArg,
    // 后面依次传入当前处理的元素、当前元素下标、调用map方法的数组
    // 当thisArg为undefined或者null时,this就指向全局的window了.这正好与原生的map方法一致.
    let temp = callback.call(thisArg, arr[i], i, arr)
    result[i] = temp
  }
  return result
}
let arr = [1, 2, 3]
let result = arr._map(function (value, index, arr) {
  return value + 1
})
console.log(result) // [2, 3, 4]

```

这样就实现了一个数组的map方法，但是这样实现的与原生的有很多区别：

##### 原生的map方法会跳过被用`delete`删除或者未定义的元素


```javascript

// 注意: 这里的arr[3]就是一个未定义的元素
let arr = [1, 2, 3, , undefined]
let result = arr._map(function (value, index, arr) {
  return value + '1'
})

```


- 原生的结果

```javascript

console.log(arr) // [1, 2, 3, empty, undefined]
console.log(result) // ["11", "21", "31", empty, "undefined1"]

```

- 上面自定义map方法的结果

```javascript

console.log(arr) // [1, 2, 3, empty, undefined]
console.log(result) // ["11", "21", "31", "undefined1", "undefined1"]

```

这里未定义的元素我觉得应该是在数组中通过索引获取不到值的元素，也就是这个元素根本不存在。我上面自定义的map方法其实是没有注意到这一点的，当获取一个数组中不存在的元素，肯定返回的是undefined，然后就得到了上面的结果。

那我们该怎么去解决这一问题呢？

##### `in`运算符

`in`运算符是用来检测一个对象是否含有特定的属性。但是说到`in`就不得不提一下`Object.prototype.hasOwnProperty()`这个方法了，虽然二者都可以用来检测某个对象上是否有特定的属性，但是后者会忽略掉从原型上继承过来的属性。

- 举个例子
  - 使用`in`运算符

在JavaScript中，数组也是对象，数组的索引就可以看做是属性了。从图中可以看到，数组arr自身是没有`push`方法的，但是通过原型链可以找到`arr.__proto__.push`，所以呢，图中最后一个表达式的只为`true`。
  - 使用`Object.prototype.hasOwnProperty()`方法


`hasOwnProperty`这个方法局限在对象自身的属性上，它会忽略掉从原型链上继承过来的属性。

##### 完善功能

1. 对未定义的元素，让回调不被其调用；
2. 对于传入的callback不是函数给出异常提示。

代码如下：

```javascript

Array.prototype._map = function (callback, thisArg) {
  let result
  // callback必须是一个函数,否则抛出异常
  if (Object.prototype.toString.call(callback) !== '[object Function]') {
    throw new Error(callback + 'is not a function')
  }
  let len = this.length
  // 创建结果数组,长度和原数组相同
  result = new Array(len)
  for (let i in this) {
    let currentVal, mappedVal
    currentVal = this[i]
    mappedVal = callback.call(thisArg, currentVal, i, this)
    result[i] = mappedVal
  }
  return result
}

```

- 测试功能

```javascript

let arr = [1, 2, 3, , undefined]
let result = arr._map(function (value) {
  return value + '1'
})
console.log(arr) // [1, 2, 3, empty, undefined]
console.log(result) // ["11", "21", "31", empty, "undefined1"]

```
 