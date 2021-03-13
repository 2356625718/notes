# [Vue源码解析] Object的变化侦测

含义：Vue中通过对state中Object属性的追踪，在其值变化时通知相应的组件，再让组件去寻找这一属性用到的位置作出更新，本篇文章探讨对Object属性的追踪。

词义：1.observer：将对象转换为setter和getter形式。

​			2.dep：依赖数组，存对同一数据有依赖关系的watcher。（依赖也就是需要通知更新数据，对该属性有依赖的地方）

​			3.watcher：依赖。

图解：![image-20201121202032934](https://tva1.sinaimg.cn/large/0081Kckwgy1gkx28wczmij31rb0u0n3j.jpg)

代码实现：

1.将object的属性转换成响应式数据（含setter和getter）：

```javascript
/**
 * 将一个属性转换成响应式数据
 * @param {*} data 
 * @param {*} key 
 * @param {*} val 
 */

function defineReactive (data,key,val) {
  if(typeof val === 'object'){      //val值为object则递归转置
    new Observer(val)
  } 
  let dep = new Dep()
  Object.defineProperty(data,key,{
    enumerable:true,
    configurable:true,
    get:function () {
      dep.depend()    //依赖获取数据时同时添加其到该数据的依赖数组dep中
      return val
    },
    set:function (newVal) {
      if(val === newVal){
        return
      }
      val = newVal
      dep.notify()    //通知dep依赖数组数据改变
    }
  })
} 
```

2.Observer类将正常的Object转换为被侦测的Object：

```javascript
/**
 * 将object所有属性转换成响应式数据（数组除外）
 */
export class Observer {
  constructor(value){
    this.value = value
    if(!Array.isArray(value)){    //不是数组则调用walk方法
      this.walk(value)
    } 
  }
  //开始转换
  walk(obj){
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj,keys[i],obj[keys[i]])
    }
  }
}
```

3.dep数组存放依赖

```javascript
/**
 * 存储同一属性的依赖数组类
 */
export default class Dep{
  constructor () {
    this.subs = []
  }

  addSub(sub){
    this.subs.push(sub)
  }

  removeSub(sub){
    remove(subs,sub)
  }

  //调用数据时存储依赖关系
  depend () {
    if(window.target){
      this.addSub(window.target)
    }
  }

  notify(){
    const subs = this.subs.slice()
    for(let i = 0, l = subs.length; i < l; i++) {
      subs[i].update    //调用相应依赖更新函数，也就是通知组件更新内部数据
    }
  }
}

function remove (arr, item) {   //删除依赖
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

4.创建Watcher类中介指向真正的依赖地址

```javascript
/**
 * vm : Vue的实例，这里我认为是指逻辑图中的外界，也就是那些组件
 * expOrFn : Object属性的keypath
 * cb : 组件更新数据函数
 */
export default class Watcher{
  constructor (vm,expOrFn,cb) {
    this.vm = vm
    this.getter = parsePath(expOrFn)   //通过keypath获取这个对象
    this.cb = cb
    this.value = this.get()
  }

  get () {
    window.target = this  //this指向Watcher实例，此时在看上面dep中depend函数，发现调用时向上查找到了这里，于是将这个watcher实例保存到了dep数组中
    let value = this.getter.call(this.vm,this.vm) //因为getter会触发Dep类的depend方法，成功保存依赖关系
    window.target = undefined
    return value
  }

  update () {
    const oldValue = this.value
    this.value = this.get()
    this.cb.call(this.vm,this.value,oldValue)
  }
}
```

注意：由于使用Object.defineProperty设置getter和setter来追踪变化，但当新增属性和删除属性时便无法追踪，这在ES6语法中的Proxy可以得到解决，但由于ES6在浏览器中尚未广泛支持，所以还未更新，但尤大说过将来会重写该部分。现阶段解决方法可以通过vm.$set和vm.$delete来通知组件新增和删除属性。

由于本人也是初学前端，错误之处还请指出，互相进步。