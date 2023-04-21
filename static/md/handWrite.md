### 重写 new 方法
```js
function _new(fn,...rest) {
  // 新建一个新对象，原型绑定到构造函数的 prototype 上
  const obj = Object.create(fn.prototype);

  // 以新对象作为 this, 调用构造函数
  const res = fn.apply(obj, rest);

  // 如果构造函数的返回值的类型是对象，就直接返回返回值，如果不是，就返回新对象
  return res instanceof Object ? res : obj;
}

function Person(this:any, name:string, age:number) {
  this.name = name
  this.age = age
}

const person = _new(Person, 'olokin', 18)
```


### 重写 instanceof 方法
```js
// 逐层往上查找原型，如果最终的原型为 null，证明不存在原型链中
function _instanceOf(source:any, target:any) {
  // 基本类型、null 直接返回 false，因为不满足 instanceof 的左侧参数是对象或者函数
  if (!['function', 'object'].includes(typeof source) || source === null) return false
  
  // 获取参数的原型对象
  let proto = Object.getPrototypeOf(source)

  while (true) {
    // 当找到原型链尽头时还没找到
    if (proto === null) return false
    if (proto === target.prototype) return true
    proto = Object.getPrototypeOf(proto)
  }
}

console.log('123', String); // false
console.log(String('123'), String); // true
console.log(null, Object); // false
console.log(Date, Function); // true
```


### 柯里化
最基础的用法是缓存传参，把接受多个参数的函数变成接受一个单一参数（或部分）的函数，并返回接受剩余的参数和结果的新函数
```js
function add(...args) {
  return args.reduce((a, b) => a + b)
}

function currying(fn) {
  let arr = []

  return function temp(...args) {
    if (args.length) {
      arr = [...arr, ...args]
      return temp
    } else {
      const val = fn.apply(this, arr)
      arr = []
      return val
    }
  }
}

const addCurry = currying(add)

console.log(addCurry(1)(2)(3)(4)(5)())
console.log(addCurry(1)(2)(3)(4, 5)())
console.log(addCurry(1)(2)(3, 4, 5)())
console.log(addCurry(1)(2, 3, 4, 5)())
```