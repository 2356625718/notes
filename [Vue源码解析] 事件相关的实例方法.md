# [Vue源码解析] 事件相关的实例方法

在Vue中有五个重要的函数用来向Vue的原型中挂载方法:

```javascript
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
```

在initMixin中会向Vue构造函数的prototype属性添加_init方法，执行new Vue（）时，会调用\_init方法来实现一系列初始化操作，包括整个生命周期的流程以及响应式系统流程的启动等。

在stateMixin中向Vue原型上挂载了vm.$watch,vm.$set,vm.$delete这三个与数据相关的实例方法

在eventsMixin中挂载了四个与事件相关的实例方法：vm.$on、vm.$once、vm.$off、vm.$emit

首先要说明一个属性：vm.\_events，此属性是在new Vue的时候调用\_init方法挂载的vm._events = Object.create(null),该属性用来存储事件。

1、vm.$on(event, callback)

```javascript
/**
 * vm.$on源码
 * @param {*} event 
 * @param {*} fn 
 */
Vue.prototype.$on = function (event, fn) {
  const vm = this
  if (Array.isArray(event)) {
    for (let i = 0, l = event.length; i < l; i++) {
      this.$on(event[i],fn)
    }
  } else {
    (vm._events[event] || (vm._events[event] = [])).push(fn)
    return vm
  }
}
```



理解：传入一个事件和回调函数，如果事件是一个事件数组，则递归调用，如果不是则在vm._events上挂载该事件（key）的回调函数（value）

2、vm.$off([event, callback])

+ 用法：

  1. 如果没有提供参数，则移除所有事件的监听器

  2. 提供了事件，移除该事件对应的所有监听器

  3. 同时提供了事件与回调，则只移除这个回调的监听器

     

```javascript
/**
 * vm.$off源码
 */

 Vue.prototype.$on = function (event, fn) {
   const vm = this
   //没有参数移除所用监听器
   if (!arguments.length) {
     vm._events = Object.create(null)
     return vm
   }

   //event支持数组
   if (Array.isArray(event)) {
     for (let i = 0, l = event.length; i < 1; i++) {
       this.$off(event[i], fn)
     }
     return vm
   }
   //取出对应的监听器
   const cbs = vm._events[event]
   if (!cbs) {
     return vm
   }

   //没有fn参数时移除该事件的所用监听器
   if (arguments.length === 1) {
     vm._events = null
     return vm
   }

   //有fn参数时只移除与fn相同的监听器
   if (fn) {
     let cb
     let i = cbs.length
     while (i--) {
       cb = cbs[i]
       if (cb === fn || cb.fn === fn) {
         cbs.splice(i, 1)
         break
       }
     }
   }
   return vm
 }
```





3、vm.$once(event, callback)

用法:监听一个自定义事件，但是只触发一次，在第一次触发后移除监听器

```javascript
 /**
  * vm.$once源码
  */

Vue.prototype.$once = function (event, fn) {
  const vm = this
  function on () {
    vm.$off(event, on)
    fn.apply(vm, arguments)
  }
  on.fn = fn
  vm.$on(event, on)
  return vm
}
```

理解：很清晰,$once注册监听器其实使用$on注册的，而且注册的函数是我们自定义的函数，触发事件时会先摘除该事件监听器，然后执行原来的回调函数



4、vm.$emit(event, [...args])

用法：触发当前实例上的事件，附加参数都会传给监听器回调

```javascript
/**
 * vm.$emit
 */


Vue.prototype.$emit = function (event) {
  const vm = this
  let cbs = vm._events[event]
  if (cbs) {
    const args = toArray(arguments, 1)
    for (let i = 0, l = cbs.length; i < l; i++) {
      try {
        cbs[i].apply(vm, args)
      } catch (e) {
        handleError(e, vm, `event handler for "${event}"`)
      }
    }
  }
  return vm
}
```



