### 一、数据类型

#### 1、基本数据类型

Number、String、Boolean、Null、undefined、Symbol、Bigint

Bigint 是最近新引入的基本数据类型

#### 2、引用数据类型

对象、数组、函数

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
  console.log(obj2) // {name: "yang", res: {value: 123}}
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
  console.log(obj2) // {name: "yang", res: {value: 123}}
  obj2.res.value = 456
  console.log(obj2) // {name: "haha", res: {value: 456}}
  console.log(obj1) // {name: "haha", res: {value: 456}}
  obj2.name = 'haha'
  console.log(obj2) // {name: "haha", res: {value: 456}}
  console.log(obj1) // {name: "yang", res: {value: 456}}
  ```

- 对于数组实现浅拷贝还可以使用**slice**、**concat**

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

- 第一版

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

- 第二版

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

- 第三版

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

  ### 总结

至此, 深浅拷贝已全部实现，但学无止境，我们可以考虑使用 `Proxy`提升深拷贝性能，通过拦截 `set` 和 `get` 实现，当然 `Object.defineProperty()` 也可以。感兴趣的同学可以查看这文章，文章介绍的很详细[头条面试官：你知道如何实现高性能版本的深拷贝嘛？]( https://juejin.im/post/5df7175fe51d45582512962c )

文章中有什么问题， 欢迎大家积极指出， 不胜感激！！！
