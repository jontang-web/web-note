### 常用JavaScript工具库：
1. [lodash.js](https://www.lodashjs.com/)
2. [moment.js](http://momentjs.cn/) 
3. [RxJS](https://cloud.tencent.com/developer/doc/1257)


##### 1，实现一个call函数

```
//call函数调用方法，参数是数组元素
fn.call(this, ...arguments);

// 将要改变this指向的方法挂到目标this上执行并返回
Function.prototype.mycall = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('not funciton')
  }
  context = context || window
  context.fn = this
  let arg = [...arguments].slice(1)
  let result = context.fn(...arg)
  delete context.fn
  return result
} 

```
##### 2，实现一个apply函数

```
//apply函数调用方法，参数是数组
fn.apply(this, [...arguments]);

Function.prototype.myapply = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('not funciton')
  }
  context = context || window
  context.fn = this
  let result
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```
##### 3，实现一个bind函数

```
//bind函数调用方法，bind函数执行结果返回一个函数，必须在bind后加()执行返回的函数才能得到最终结果
fn.bind(this, ...arguments)();

Function.prototype.mybind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  let _this = this
  let arg = [...arguments].slice(1)
  return function F() {
    // 处理函数使用new的情况
    if (this instanceof F) {
      return new _this(...arg, ...arguments)
    } else {
      return _this.apply(context, arg.concat(...arguments))
    }
  }
}
```
##### 4，instanceof的原理
```
// 右边变量的原型存在于左边变量的原型链上
function instanceOf(left, right) {
  let leftValue = left.__proto__
  let rightValue = right.prototype
  while (true) {
    if (leftValue === null) {
      return false
    }
    if (leftValue === rightValue) {
      return true
    }
    leftValue = leftValue.__proto__
  }
}
```
##### 5，Object.create的基本实现原理

```
function create(obj) {
  function F() {};
  F.prototype = obj;
  return new F()
}
```
##### 6，new本质

```
function myNew (fun) {
  return function () {
    // 创建一个新对象且将其隐式原型指向构造函数原型
    let obj = {
      __proto__ : fun.prototype
    }
    // 执行构造函数
    fun.call(obj, ...arguments)
    // 返回该对象
    return obj
  }
}

function person(name, age) {
  this.name = name
  this.age = age
}
let obj = myNew(person)('chen', 18) // {name: "chen", age: 18}
```
##### 7，实现一个基本的Promise

```
// ①自动执行函数，②三个状态，③then
class Promise {
  constructor (fn) {
    // 三个状态
    this.state = 'pending'
    this.value = undefined
    this.reason = undefined
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled'
        this.value = value
      }
    }
    let reject = value => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        this.reason = value
      }
    }
    // 自动执行函数
    try {
      fn(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }
  // then
  then(onFulfilled, onRejected) {
    switch (this.state) {
      case 'fulfilled':
        onFulfilled()
        break
      case 'rejected':
        onRejected()
        break
      default:
    }
  }
}
```
##### 8，Promisify实现
```
function promisify(fn) {
    return function () {
        var args = Array.prototype.slice.call(arguments);
        return new Promise(function (resolve) {
            args.push(function (result) {
                resolve(result);
            });
            fn.apply(null, args);
        })
    }
}
```


##### 9，实现浅拷贝

```
// 1. ...实现
let copy1 = {...{x:1}}

// 2. Object.assign实现

let copy2 = Object.assign({}, {x:1})
```
##### 10，实现一个基本的深拷贝

```
// 1. JOSN.stringify()/JSON.parse()
let obj = {a: 1, b: {x: 3}}
JSON.parse(JSON.stringify(obj))

// 2. 递归拷贝
function deepClone(obj) {
  let copy = obj instanceof Array ? [] : {}
  for (let i in obj) {
    if (obj.hasOwnProperty(i)) {
      copy[i] = typeof obj[i] === 'object' ? deepClone(obj[i]) : obj[i]
    }
  }
  return copy
}
```
##### 11，实现一个基本的Event Bus
常用于解决组件之间的通信问题（发布订阅模式）
```
//emitter.js
class EventEmitter {
  constructor () {
    // 存储事件
    this._events = this._events || new Map()
  }
  // 监听事件
  addListener (type, fn) {
    const handle = this._events.get(type);
    if (!handle) {
        this._events.set(type, [fn]);
    }else{
        this._events.set(type, [...handle, fn]);
    }
  }
  // 触发事件
  emit (type, ...args) {
    let handle = this._events.get(type);
    if(handle){
        handle.map(fn => {
            fn.apply(this, args);
        });
    }else{
        handler.call(this);
    }
    return true;
  }
}
const emitter = new EventEmitter();
export default emitter


import emitter from 'emitter';
// 监听事件
emitter.addListener('ages', age => {
  console.log(age)
})
// 触发事件
emitter.emit('ages', 18)  // 18
```
##### 12，实现一个双向数据绑定

```
let obj = {}
let input = document.getElementById('input')
let span = document.getElementById('span')
Object.defineProperty(obj, 'text', {
  configurable: true,
  enumerable: true,
  get() {
    console.log('获取数据了')
    return obj['text']
  },
  set(newVal) {
    console.log('数据更新了')
    input.value = newVal
    span.innerHTML = newVal
    obj['text'] = newVal
  }
})
input.addEventListener('keyup', function(e) {
  obj.text = e.target.value
})
```
##### 13，实现一个简单路由

```
class Route{
  constructor(){
    // 路由存储对象
    this.routes = {}
    // 当前hash
    this.currentHash = ''
    // 绑定this，避免监听时this指向改变
    this.freshRoute = this.freshRoute.bind(this)
    // 监听
    window.addEventListener('load', this.freshRoute, false)
    window.addEventListener('hashchange', this.freshRoute, false)
  }
  // 存储
  storeRoute (path, cb) {
    this.routes[path] = cb || function () {}
  }
  // 更新
  freshRoute () {
    this.currentHash = location.hash.slice(1) || '/'
    this.routes[this.currentHash]()
  }   
}
```
##### 14，rem实现原理

```
function setRem () {
  let doc = document.documentElement
  let width = doc.getBoundingClientRect().width
  // 假设设计稿为宽750，则rem为10px
  let rem = width / 75
  doc.style.fontSize = rem + 'px'
}
```
##### 15，手写Ajax

```
// 1. 简单实现

// 实例化
let xhr = null;
if(window.XMLHttpRequest){
	xhr = new XMLHttpRequest();
}else{
	xhr = new ActiveXObject('Mircsoft.XMLHTTP');
}
// 初始化
xhr.open(method, url, async);
// 发送请求
if(method.toLowerCase()==='post'){
    xhr.setRequestHeader("Content-type","application/x-www-form-urlencoded;charset=UTF-8"); 
    xhr.send(data);
    //data是string类型，且仅限post请求方式
}else{
    xhr.send();
}
// 设置状态变化回调处理请求结果
xhr.onreadystatechange = () => {
  if (xhr.readyStatus === 4 && xhr.status === 200) {
    console.log(xhr.responseText)
  }
}


// 2. 基于promise实现 

function ajax (options) {
  // 请求地址
  const url = options.url
  // 请求方法
  const method = options.method.toLocaleLowerCase() || 'get'
  // 默认为异步true
  const async = options.async
  // 请求参数
  const data = options.data
  // 实例化
  const xhr = new XMLHttpRequest()
  // 请求超时
  if (options.timeout && options.timeout > 0) {
    xhr.timeout = options.timeout
  }
  // 返回一个Promise实例
  return new Promise ((resolve, reject) => {
    xhr.ontimeout = () => reject && reject('请求超时')
    // 监听状态变化回调
    xhr.onreadystatechange = () => {
      if (xhr.readyState == 4) {
        // 200-300 之间表示请求成功，304资源未变，取缓存
        if (xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
          resolve && resolve(xhr.responseText)
        } else {
          reject && reject()
        }
      }
    }
    // 错误回调
    xhr.onerror = err => reject && reject(err)
    let paramArr = []
    let encodeData
    // 处理请求参数
    if (data instanceof Object) {
      for (let key in data) {
        // 参数拼接需要通过 encodeURIComponent 进行编码
        paramArr.push(encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
      }
      encodeData = paramArr.join('&')
    }
    // get请求拼接参数
    if (method === 'get') {
      // 检测url中是否已存在 ? 及其位置
      const index = url.indexOf('?')
      if (index === -1) url += '?'
      else if (index !== url.length -1) url += '&'
      // 拼接url
      url += encodeData
    }
    // 初始化
    xhr.open(method, url, async)
    // 发送请求
    if (method === 'get') xhr.send(null)
    else {
      // post 方式需要设置请求头
      xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded;charset=UTF-8')
      xhr.send(encodeData)
    }
  })
}
```
##### 16，实现拖拽

```
window.onload = function () {
  // drag处于绝对定位状态
  let drag = document.getElementById('box')
  drag.onmousedown = function(e) {
    var e = e || window.event
    // 鼠标与拖拽元素边界的距离 = 鼠标与可视区边界的距离 - 拖拽元素与边界的距离
    let diffX = e.clientX - drag.offsetLeft
    let diffY = e.clientY - drag.offsetTop
    drag.onmousemove = function (e) {
      // 拖拽元素移动的距离 = 鼠标与可视区边界的距离 - 鼠标与拖拽元素边界的距离
      let left = e.clientX - diffX
      let top = e.clientY - diffY
      // 避免拖拽出可视区
      if (left < 0) {
        left = 0
      } else if (left > window.innerWidth - drag.offsetWidth) {
        left = window.innerWidth - drag.offsetWidth
      }
      if (top < 0) {
        top = 0
      } else if (top > window.innerHeight - drag.offsetHeight) {
        top = window.innerHeight - drag.offsetHeight
      }
      drag.style.left = left + 'px'
      drag.style.top = top + 'px'
    }
    drag.onmouseup = function (e) {
      this.onmousemove = null
      this.onmouseup = null
    }
  }
}
```
##### 17，实现一个节流函数（throttle）

```
##原理：在固定时间间隔内只执行一次操作
##应用场景：耗时操作，比如用户点击按钮发送请求，12306抢票，秒杀等
function throttle (fn, delay) {
  // 利用闭包保存时间
  let prev = Date.now()
  return function () {
    let now = Date.now();
    if (now - prev >= delay) {
      fn.apply(this, arguments);
      //或者 fn.call(this, ...arguments);
      prev = Date.now()
    }
  }
}

function fn () {
  console.log('节流')
}
Element.addEventListener('scroll', throttle(fn, 1000)); 
```
##### 18，实现一个防抖函数（debounce）

```
##原理：当用户停止操作并过指定时间后执行期望动作，如果在指定时间内又进行操作，会重新记时
##应用场景：页面滚动，用户输入请求等，例如input远程提示，用户滚动页面时适配
function debounce (fn, delay) {
  // 利用闭包保存定时器
  let timer = null
  return function () {
    let context = this;
    let arg = arguments;
    // 在规定时间内再次触发会先清除定时器后再重设定时器
    timer && clearTimeout(timer);
    timer = setTimeout(function () {
      fn.apply(context, arg)
    }, delay);
  }
}

function fn () {
  console.log('防抖')
}

Element.addEventListener('scroll', debounce(fn, 1000));
```
