### 前言

面试官：如何手写一个深拷贝？
应聘者： JSON.parse(JSON.stringify(object))
面试官：有什么优缺点？
应聘者：不能序列化函数， 忽略undefined... 
面试官：有其他方法？ 怎么解决循环引用？
应聘者：for循环 。。。

### 一、数据类型

#### 1、基本数据类型

Number、String、Boolean、Null、undefined、Symbol、Bigint

Bigint 是最近新引入的基本数据类型

#### 2、引用数据类型

Object、Array、Function等

数据类型不是本文重点， 重点是实现深浅拷贝



```! 
下面是要copy的对象, 之后的代码都会直接使用$obj， 之后不会再次声明
```
```js
// lmran
var $obj = {
    func: function () {
        console.log('this is function')
    },
    date: new Date(),	
    symbol: Symbol(),
    a: null,
    b: undefined,
    c: {
        a: 1
    },
    e: new RegExp('regexp'),
    f: new Error('error')
}

$obj.c.d = $obj
```



### 二、 浅拷贝

#### 1、什么是浅拷贝

一句话可以说就是：对对象而言，它的第一层属性值如果是基本数据类型则完全拷贝一份数据，如果是引用类型就拷贝内存地址。确实拷贝的很浅[偷笑]

#### 2、实现

- `Object.assign()`

  ```js
  // lmran
  let obj1 = {
      name: 'yang',
      res: {
          value: 123
      }
  }
  
  let obj2 = Object.assign({}, obj1)
  obj2.res.value = 456
  console.log(obj2) // {name: "haha", res: {value: 456}}
  console.log(obj1) // {name: "haha", res: {value: 456}}
  obj2.name = 'haha'
  console.log(obj2) // {name: "haha", res: {value: 456}}
  console.log(obj1) // {name: "yang", res: {value: 456}}
  ```

  

- 展开语法 `Spread`

  ```js
  // lmran
  let obj1 = {
      name: 'yang',
      res: {
          value: 123
      }
  }
  
  let {...obj2} = obj1
  obj2.res.value = 456
  console.log(obj2) // {name: "haha", res: {value: 456}}
  console.log(obj1) // {name: "haha", res: {value: 456}}
  obj2.name = 'haha'
  console.log(obj2) // {name: "haha", res: {value: 456}}
  console.log(obj1) // {name: "yang", res: {value: 456}}
  ```

- `Array.prototype.slice`
 ```js
  // lmran
  const arr1 = [
      'yang',
      {
          value: 123
      }
  ];
  
  const arr2 = arr1.slice(0);
  arr2[1].value = 456;
  console.log(arr2); // ["yang", {value: 456}]
  console.log(arr1); // ["yang", {value: 456}]
  arr2[0] = 'haha';
  console.log(arr2); // ["haha", {value: 456}]
  console.log(arr1); // ["yang", {value: 456}]
 ```
- `Array.prototype.concat`
```js
  // lmran
  const arr1 = [
      'yang',
      {
          value: 123
      }
  ];
  
  const arr2 = [].concat(arr1);
  arr2[1].value = 456;
  console.log(arr2); // ["yang", {value: 456}]
  console.log(arr1); // ["yang", {value: 456}]
  arr2[0] = 'haha';
  console.log(arr2); // ["haha", {value: 456}]
  console.log(arr1); // ["yang", {value: 456}]
```
  ```!
  实际上对于数组来说， 只要不修改原数组， 重新返回一个新数组就可以实现浅拷贝，比如说map、filter、reduce等方法
  ```

### 三、深拷贝

#### 1、什么是深拷贝

深拷贝就是不管是基本数据类型还是引用数据类型都重新拷贝一份， 不存在共用数据的现象

#### 2、实现

- 暴力版本 ` JSON.parse(JSON.stringify(object)) `

  ```js
  // lmran
  let obj = JSON.parse(JSON.stringify($obj))
  console.log(obj) 			// 不能解决循环引用
  /*
  	VM348:1 Uncaught TypeError: Converting circular structure to JSON
      at JSON.stringify (<anonymous>)
      at <anonymous>:1:17
  */
  delete $obj.c.d
  let obj = JSON.parse(JSON.stringify($obj))
  console.log(obj) 			// 丢失了大部分属性
  /*
  	{
          a: null
          c: {a: 1}
          date: "2020-04-05T09:51:32.610Z"
          e: {}
          f: {}
    }
  */
  ```

  存在的问题：

  1、会忽略 `undefined`

  2、会忽略 `symbol`

  3、不能序列化函数

  4、不能解决循环引用的对象

  5、不能正确处理`new Date()`

  6、不能处理正则

  7、不能处理new Error()

- 第一版 基础版本 

  递归遍历对象属性 

  ```js
  // lmran
  function deepCopy (obj) {
      if (obj === null || typeof obj !== 'object') {
          return obj
      }
      
      let copy = Array.isArray(obj) ? [] : {}
      Object.keys(obj).forEach(v => {
          copy[key] = deepCopy(obj[key])
      })
  
      return copy
  }
  
  deepCopy($obj)
  /*
  VM601:23 Uncaught RangeError: Maximum call stack size exceeded
      at <anonymous>:23:30
      at Array.forEach (<anonymous>)
      at deepCopy (<anonymous>:23:22)
  */
  delete $obj.c.d
  deepCopy($obj)
  /*
  {
      a: null
      b: undefined
      c: {a: 1}
      date: {}
      e: {}
      f: {}
      func: ƒ ()
      symbol: Symbol()
  }
  */
  ```

  存在的问题是： 

  1、不能解决循环引用的对象

  2、不能正确处理`new Date()`

  3、不能处理正则

  4、不能处理new Error()

- 第二版  循环引用

  先解决解决循环遍历问题， 解决办法是将对象，对象属性存储在数组中查看下次遍历时有无已经遍历过的对象，有则直接返回， 否则继续遍历

```js
// lmran
function deepCopy (obj, cache = []) {
    if (obj === null || typeof obj !== 'object') {
        return obj
    }

    const item = cache.filter(item => item.original === obj)[0]
    if (item) return item.copy
    
    let copy = Array.isArray(obj) ? [] : {}
    cache.push({
        original: obj,
        copy
    })

    Object.keys(obj).forEach(key => {
        copy[key] = deepCopy(obj[key], cache)
    })

    return copy
}
deepCopy($obj)
/*
{
    a: null
    b: undefined
    c: {a: 1, d: {…}}
    date: {}
    e: {}
    f: {}
    func: ƒ ()
    symbol: Symbol()
}

```

完美解决了循环引用问题, 但是仍然存在几个小问题，都属于同一类问题

- 第三版 特殊值处理

  对于最终的几个对象的处理，可以判断类型， 重新new一个返回就可以了

```js
// lmran
function deepCopy (obj, cache = []) {
    if (obj === null || typeof obj !== 'object') {
        return obj
    }

    if (Object.prototype.toString.call(obj) === '[object Date]') return new Date(obj)
    if (Object.prototype.toString.call(obj) === '[object RegExp]') return new RegExp(obj)
    if (Object.prototype.toString.call(obj) === '[object Error]') return new Error(obj)
    const item = cache.filter(item => item.original === obj)[0]
    if (item) return item.copy
    
    let copy = Array.isArray(obj) ? [] : {}
    cache.push({
        original: obj,
        copy
    })

    Object.keys(obj).forEach(key => {
        copy[key] = deepCopy(obj[key], cache)
    })

    return copy
}
deepCopy($obj)
/*
{
    a: null
    b: undefined
    c: {a: 1, d: {…}}
    date: Fri Apr 10 2020 20:06:08 GMT+0800 (中国标准时间) {}
    e: /regexp/
    f: Error: Error: error at deepCopy (<anonymous>:8:74) at <anonymous>:19:21 at Array.forEach (<anonymous>) at deepCopy (<anonymous>:18:22) at <anonymous>:24:1
    func: ƒ ()
    symbol: Symbol()
}
*/
```

- 第四版 函数引用相同

  到这里基本功能似乎已经实现， 但是还存在一个问题， 就是函数引用了同一个内存地址, 对于这个问题网上大部分都是直接返回或者返回为对象， 包括lodash也是这么处理的

  ```js
   const isFunc = typeof value == 'function'
   if (isFunc || !cloneableTags[tag]) {
          return object ? value : {}
   }
  ```

  那么如何结局这个问题就需要用eval函数， 虽说这个函数已经不推荐使用， 但是它还是能解决问题的。 函数这里分两种： `普通函数`和`箭头函数`， 区分这两者只需要看有无`prototype`, 有`prototype`属性就属于普通函数， 没有就是箭头函数

```js
// lmran
function deepFunction(func) {
	let fnStr = func.toString()
	return func.prototype ? eval(`(${fnStr})`) : eval(fnStr)
}

function deepCopy (obj, cache = []) {
    if (obj === null || (typeof obj !== 'object' && obj !== 'function')) {
        return obj
    }

    if (Object.prototype.toString.call(obj) === '[object Date]') return new Date(obj)
    if (Object.prototype.toString.call(obj) === '[object RegExp]') return new RegExp(obj)
    if (Object.prototype.toString.call(obj) === '[object Error]') return new Error(obj)
    if (Object.prototype.toString.call(obj) === '[object Function]') return deepFunction(obj)
    const item = cache.filter(item => item.original === obj)[0]
    if (item) return item.copy
    
    let copy = Array.isArray(obj) ? [] : {}
    cache.push({
        original: obj,
        copy
    })

    Object.keys(obj).forEach(key => {
        copy[key] = deepCopy(obj[key], cache)
    })

    return copy
}
deepCopy($obj)
```

  ### 总结

至此, 深浅拷贝已全部实现，但学无止境，我们还可以考虑使用 `Proxy`提升来深拷贝性能，通过拦截 `set` 和 `get` 实现，当然 `Object.defineProperty()` 也可以。感兴趣的同学可以查看这文章，文章介绍的很详细[头条面试官：你知道如何实现高性能版本的深拷贝嘛？]( https://juejin.im/post/5df7175fe51d45582512962c )


针对@Brota掘友提出的对于函数引用相同的问题， 文中现已将该问题解决了



文章中有什么问题， 欢迎大家积极指出， 不胜感激！！！


### 参考
- [MDN   eval函数的使用]( https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval )
- [如何写出一个惊艳面试官的深拷贝?](https://juejin.im/post/5d6aa4f96fb9a06b112ad5b1)
- [头条面试官：你知道如何实现高性能版本的深拷贝嘛？]( https://juejin.im/post/5df7175fe51d45582512962c )