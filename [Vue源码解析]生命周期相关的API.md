# [Vue源码解析]生命周期相关的API

+ 生命周期相关的实例方法有四个，vm.$forceUpdate和vm.$destroy是在lifecycleMixin中挂载到Vue原型上去的，vm.$nextTick是在renderMixin中挂载到Vue原型上去的，vm.$mount则是在跨平台代码中挂载上去的。

  1、vm.$forceUpdate()

  作用：迫使当前实例及插入插槽内容的子组件重新渲染

  ```javascript
  /**
   * vm.$forceUpdate()
   */
  
  Vue.prototype.$forceUpdate = function () {
    const vm = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }
  ```

  理解：调用·watcher上的update方法通知外界即组件重新渲染数据

  2、vm.$destroy()

  用法：销毁一个实例，清理该实例与其他实例的连接，并解绑其全部指令及监听器，同时会触发beforeDestroy和destroyed的钩子函数。

  首先说明一点，在创建watcher监测状态的时候，实例上有一个属性vm._watchers数组会将该watcherpush进来，也就是说,vm.\_watcher中保存了我们所有的watcher，包括Vue监测响应式数据自己生成的和用户通过vm.$watch()方法生成的watcher

  ```javascript
  /**
   * vm.$destroy()
   */
  
  Vue.prototype.$destroy = function () {
    const vm = this
    if (vm.isBeingDestroyed) {    //判断是否正在销毁，如果是避免重复进行操作
      return
    }
    callHook(vm, 'beforeDestroy')   //  调用beforeDestroy生命周期函数
    vm._isBeingDestroyed = true
  
    //删除自己与父组件的连接
    const parent = vm.$parent
    if (parent && !parent.isBeingDestroyed && !vm.$options.abstract) {    //存在且没被销毁且该实例不是抽象组件
      removeEventListener(parent.$children, vm)   //该属性中保存着所有子组件实例
    }
  
    //从watcher监听的所有状态的依赖列表中移除watcher
    if (vm._watcher) {
      vm._watcher.teardown()
    }
    let i = vm._watcher.length
    while (i--) {
      vm._watcher[i].teardown()
    }
  
    vm._isDestroyed = true
    //在vnode树上触发destroy钩子函数解绑指令
    vm.__patch__(vm.vnode, null)
    callHook(vm, 'destroyed')   //触发destroyed生命周期钩子函数
    //移除所有事件的监听器
    vm.$off()
  }
  ```

  3、vm.$nextTick()

  + 用法：该方法接收一个回调函数作为参数，将这个回调函数延迟到下次DOM更新周期之后执行，如果没有提供回调函数则会返回一个Promise
  + 在Vue.js中，状态变化会通过dep（依赖收集）通知到watcher，使watcher推入到一个渲染队列中，在下一次事件循环中再让watcher触发渲染(异步渲染操作)

  - 这是由于变化侦测的通知会通知到watcher，不过如果数据更新很频繁，就会导致watcher收到很多通知，如果watcher收到通知就开始渲染会导致资源浪费，其实完全可以等所有状态修改完毕后再由虚拟DOM一次性渲染，所以引入渲染队列来存放watcher（需要判断队列中是否已有该watcher，因为一次渲染到最新，如果已有的话就不需要重复渲染了）。当下一次时间循环时让队列中的watcher触发渲染流程进行统一渲染。

  - js异步事件处理完毕后会加入到事件队列中，被放入后事件不会立即执行其回调，而是等待当前任务栈中所有任务执行完毕后，主线程会去查找事件队列中是否有任务。

  - 异步任务有两种类型：微任务和宏任务，不同类型的任务会被分配到不同的任务队列中

    微任务：

    1. Promise.then

    2. MutationObserver
    3. Object.observe
    4. Process.nextTick

    宏任务：

    1. setTimeout
    2. setInterval
    3. setImmediate
    4. MessageChannel
    5. requestAnimationFrame
    6. I/O
    7. UI交互事件

```javascript
/**
 * vm.$forceUpdate()
 */

Vue.prototype.$forceUpdate = function () {
  const vm = this
  if (vm._watcher) {
    vm._watcher.update()
  }
}

/**
 * vm.$destroy()
 */

Vue.prototype.$destroy = function () {
  const vm = this
  if (vm.isBeingDestroyed) {    //判断是否正在销毁，如果是避免重复进行操作
    return
  }
  callHook(vm, 'beforeDestroy')   //  调用beforeDestroy生命周期函数
  vm._isBeingDestroyed = true

  //删除自己与父组件的连接
  const parent = vm.$parent
  if (parent && !parent.isBeingDestroyed && !vm.$options.abstract) {    //存在且没被销毁且该实例不是抽象组件
    removeEventListener(parent.$children, vm)   //该属性中保存着所有子组件实例
  }

  //从watcher监听的所有状态的依赖列表中移除watcher
  if (vm._watcher) {
    vm._watcher.teardown()
  }
  let i = vm._watcher.length
  while (i--) {
    vm._watcher[i].teardown()
  }

  vm._isDestroyed = true
  //在vnode树上触发destroy钩子函数解绑指令
  vm.__patch__(vm.vnode, null)
  callHook(vm, 'destroyed')   //触发destroyed生命周期钩子函数
  //移除所有事件的监听器
  vm.$off()
}


/**
 * vm.$nextTick()是调用全局的nextTick方法执行的
 */

Vue.prototype.$nextTick = function (fn) {
  return nextTick(fn, this)
}

/**
 * Vue.nextTick()
*/

const callbacks = []    //所有回调函数数组
let pending = false     //一次事件循环添加一个回调函数的flag

function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)   //深拷贝，转类数组为数组
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()         //将回调函数添加进微任务队列中
  }
}

let microTimerFunc    //将回调函数转换为微任务
let macroTimerFunc    //将回调函数转换为宏任务
let useMacroTask = false  //false为微任务，true为宏任务

if (typeof setImmediate !== 'undefined' && isNative(setImmediate)){    //判断是否支持IE宏任务setImmediate
  microTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (typeof MessageChannel !== 'undefined' && (isNative(MessageChannel) || MessageChannel.toString() === '[object MessageChannelConstructor]')) {   //判断是否支持MessageChannel宏任务
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
}

if (typeof Promise !== 'undefined' && isNative(Promise)) {    //判断是否支持Promise微任务
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
  }
} else {
  microTimerFunc = macroTimerFunc
}

export default function withMacroTask (fn) {    //给回调函数包装函数_withTask，如果回调函数执行过程中修改了状态，更新DOM的操作会被推入宏任务队列
  return fn._withTask || (fn._withTask = function () {
    useMacroTask = true
    const res = fn.apply(null, arguments)
    useMacroTask = false
    return res
  })
}

export function nextTick (cb, ctx) {    //回调函数和context上下文
  let _resolve
  callbacks.push(() => {
    if (cb) {
      cb.call(ctx)
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
}

if (!pending) {     //宏任务还是微任务
  pending = true
  if (useMacroTask) {
    macroTimerFunc
  } else {
    microTimerFunc
  }
}

if (!cb && typeof Promise !== 'undefined') {
  return new Promise (resolve => {
    _resolve = reslove
  })
}

```

4、vm.$mount([element|Selector])

+ 用法：实例中没有el选项时处于未挂载状态，我们可以通过vm.$mount手动挂载一个未挂载的实例。

  ```javascript
  /**
   * 完整版vm.$mount实现原理
   */
  
  const mount = Vue.prototype.$mount
  Vue.prototype.$mount = function (el) {
    el = el && query(el)    //元素或者选择器对应的元素
  
    const options = this.$options //this.$options上有用户设置的一些参数，如template或render
    if (!options.render) {
      let template = options.template
      if (template) {
        if (typeof template === 'string') {
          if (template.charAt(0) === '#') {   //选择器
            template = idToTemplate(template)   //选择器内部的HTML代码
          }
        } else if (template.nodeType) {   //dom节点
          template = template.innerHTML
        } else {    //错误类型
          if (ProcessingInstruction.env.NODE_ENV !== 'production') {
            warn('invalid template option:' + template, this)
          }
          return this
        }
      } else if (el) {    //元素
        template = getOuterHTML(el)
      }
    }
  
    if (template) {
      //生成渲染函数挂载到options上
      const {render} = compileToFunctions(
        template,
        {...},
        this
      )
      options.render = render
    }
  
    return mount.call(this, el) //函数劫持
  }
  
  /**
   * 运行时版本vm.$mount实现原理
   */
  
  Vue.prototype.$mount = function (el) {
    el = el && inBrowser ? query(el) : undefined
    return mountComponent(this, el)
  }
  
  export function mountComponent (vm, el) {
    if (!vm.$options.render) {
      vm.$options.render = createEmptyVNode //没有渲染函数则添加一个注释节点
      if (ProcessingInstruction.env.NODE_ENV !== 'prodection') {
        //在开发环境下发出警告
      }
    }
  
    //触发生命周期钩子
    callHook(vm, 'beforeMount')
  
    //挂载
    vm._watcher = new Watcher(vm, () => {   //Watcher第二项参数为函数时会侦测函数中所有数据变化，所以数据出现变化时，watcher会再次进入渲染流程，如此反复，直到实例被销毁。
      vm._update(vm._render())          
    }, noop)
  
    callHook(vm, 'mounted')
    return vm
  }
  
  ```

  