# [Vue源码解析] Array的变化侦测

前言：数组由于具有许多原生方法对其元素进行修改，而上文中说过Object的变化侦测无法就这些方法改变的属性通知其依赖，所以需要新的方法来进行Array的变化侦测。

核心思想：新建一个对象arrayMethods，称之为拦截器，用它覆盖数组的`__proto__`，即数组的原型对象Array.prototype，在数组更新数据时，即执行push、pop、unshift、shift、sort、reverse、splice这些方法时，执行拦截器其中的方法来通知依赖数组dep中的依赖数据发生变化。最开始依然适用Observer类包装数组，并使数组的原型对象指向拦截器，同时递归数组元素，使其都变为响应式数据。

图解：![image-20201123123017822](https://tva1.sinaimg.cn/large/0081Kckwgy1gkyzw8tk7aj31fc0u0wiu.jpg)

拦截器：

```javascript
/**
 * 拦截器，覆盖Array原型方法
 */

const arrayProto = Array.prototype
const arrayMethods = Object.create (arrayProto)

;[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach (function (method) {
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)   //执行拦截器重写的方法
    const ob = this.__ob__    //拿到Observer对象
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)   //调用函数改变数组时将改变的数据变为响应式数据
    ob.dep.notify()   //通知依赖数组发生变化
    return result
  })
})

//工具函数，用来定义拦截器属性
function def (obj, key, val, enumerable) {
  Object.defineProperty(obj, key, {
    value:val,
    enumerable:!!enumerable,
    writable:true,
    configurable:true
  })
}
```

Observer类：

```javascript
/**
 * Observer类
 */

const hasProto = '__proto__' in {}
const arrayKeys = Object.getOwnPropertyNames(arrayMethods)

class Observer{
  constructor (value) {
    this.value = value
    this.dep = new Dep()
    def(value,'__ob__',this)
    if (Array.isArray(value)) {
      const argument = hasProto
        ? protoAugment
        : copyAugment
      argument(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }
}

//覆盖原型对象,不支持__proto__则将arrayMethods所有属性方法赋值给数组
function protoAugment (target, src, keys) {
  target.__proto__ = src
}

function copyAugment (target, src, keys) {
  for (let i = 0,l = keys.length; i < l; i++) {
    const key = keys[i]
    def(target, key, src[key])
  }
}

//侦测Array中的每一项
function observeArray (items) {
  for (let i = 0, l = items.length; i < l; i++){
    observe(items[i])
  }
}
```

收集依赖

```javascript
/**
 * 收集依赖
 */
function defineReactive (data, key, val) {
  let childOb = observe(val)
  let dep = new Dep()
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get: function () {
      dep.depend()
      if (childOb) {
        childOb.dep.depend()    //通知Observer上的dep数组
      }
      return val
    },
    set: function (newVal) {
      if (val === newVal) {
        return
      }
      dep.notify()
      val = newVal
    }
  })
}

//创建响应式数据
function observe (value, asRootData) {
  if (!isObject(value)) {
    return
  }
  let ob
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else {
    ob = new Observer(value)
  }
  return ob
}
```

注意： 由于是通过拦截原型的方式实现，所以通过下标赋值（做项目的时候遇见过）和通过数组length清空数组操作无法侦测到数组的变化。