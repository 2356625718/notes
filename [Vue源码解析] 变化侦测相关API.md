# [Vue源码解析] 变化侦测相关API

1、vm.$watch(expOrfn, callback, {options})

用法：用来观察单个数据或多个数据在Vue实例上的的变化。

```javascript
/**
 * 方法实现
 * @param {*} expOrFn 
 * @param {*} cb 
 * @param {*} options 
 */
Vue.prototype.$watch = function (expOrFn, cb, options) {
  const vm = this
  options = options || {}
  const watcher = new Watcher(vm, expOrFn, cb, options)
  if (options.immediate) {
    cb.call(vm, watcher.value)
  }
  return function unwatchFn () {
    watcher.teardown()
  }
}

/**
 * 1.由于expOrFn可以是函数或者keypath，所以在Watcher中需要对其类型进行判断
 * 2.由于函数返回一个unwatchFn方法来取消侦测，所以需要在Watcher中记录哪些dep数组存储了该Watcher，通知他们将其删除
 */

class Watcher {
  constructor (vm, expOrFn, cb) {
    this.vm = vm
    if (options) {
      this.deep = !!options.deep
    } else {
      this.deep = false
    }
    this.deps = []
    this.depIds = new Set()
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrPath) //解析keypath为数据
    }
    this.cb = cb
    this.value = this.get()
  }

  get () {
    window.target = this
    let value = this.getter.call(vm, vm)
    if (this.deep) {    //如果该参数为true，则递归侦测元素使其子数据也变为响应式数据
      traverse(value)
    }
    window.target = undefined
    return value
  }
}

const seenObjects = new Set()

function traverse (val) {
  _traverse(val, seenObjects)
  seenObjects.clear()
}

function _traverse (val, seen) {
  let i,key
  const isA = Array.isArray(val)
  if ((!isA && !isObject(val)) || Object.isFrozen(val)) {
    return
  }
  if (val.__ob__) {   //判断是否已经是响应式数据，并通过id判断是否已被dep所收集
    const depId = val.__ob__.dep.id
    if (seen.has(depId)) {
      return
    }
    seen.add(depId)
  }
  if (isA) {
    i = val.length
    while (i--) _traverse(val[i], seen) //取值时会触发getter存储依赖
  } else {
    keys = objects.keys(val)
    i = keys.length
    while (i--) _traverse(val[keys[i]], seen) //取值时会触发getter存储依赖
  }
}

/**
 * 从所有依赖项的Dep列表中将自己移除
 */

 teardown () {
   let i = this.deps.length
   while (i--) {
     this.deps[i].removeSub(this)
   }
 }

 //Dep中
 removeSub (sub) {
   const index = this.PushSubscription.indexOf(sub)
   if (index > -1) {
     return this.PushSubscription.splice(index,1)
   }
 }
```

核心思想：通过该函数将数据转换为响应式数据，watcher中存放了存储这个watcher的Dep数组，如果调用unwatch()方法则删除Dep数组中的该项依赖，使得外界的这个数据无法再被dep通知到watcher再到外界，也就使其失去响应。

2.vm.$set(target, key, val)

用法：新增属性同时如果target是响应式数据使其也成为响应式数据

```javascript
/**
 * vm.$set(target, key, val)
 */
Vue.prototype.$set = function (target, key, val) {
  if (Array.isArray(target) && isValidArrayIndex(key)) {    //函数且下标有效则更新或添加,splice是拦截器上的方法
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    return val
  }

  if (key in target && !(key in Object.prototype)) {    //对象且属性已存在则更新,会触发setter上的方法
    target[key] = val
    return val
  }

  const ob = target.__ob__
  if (target._isVue || (ob && ob.vmCount)) {    //操纵Vue实例或根数据如处于开发环境下则发出警告
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data' +
      'at runtime - declare it upfront in the data option.'
    )
    return val  
  }
  if (!ob) {    //  target不是响应式数据，则不需要对新的属性进行侦测
    target[key] = val
    return val
  }
  defineReactive(ob.value, key, val)    // 排除过后就侦测该新添加的属性 
  ob.dep.notify()
  return val
}
```

​	3.vm.$delete(target, key)

用法：删除属性后通知依赖数组数据发生改变

```javascript
/**
 * vm.$delete(target, key)
 * @param {*} target 
 * @param {*} key 
 */
Vue.prototype.$delete = function del (target, key) {
  if (Array.isArray(target) && isValidArrayIndex(key)) {  //数组且下标有效则删除，splice是拦截器上的方法
    target.splice(key, 1)
    return
  }
  const ob = target.__ob__
  if (target._isVue || (ob && ob.vmCount)) {
    prodcess.env.NODE_ENV == 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data' +
      'at runtime - declare it upfront in the data option.'
    )
    return
  }
  if (!hasOwn(target, key)) {   //没有这个属性删除失败
    return
  }
  delete target[key]

  if(!ob) {
    return
  }
  ob.dep.notify()   //响应式数据则发送通知
}
```

