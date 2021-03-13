# [Vue源码解析]	全局API的实现原理

1、 Vue.extend(options)

用法：使用基础的Vue构造器创建一个子类，options为自定义组件选项

```javascript
/**
 * Vue.extend(options)
 */

let cid = 1

Vue.extend = function (extendOptions) {
  extendOptions = extendOptions || {}
  const Super = this
  const SuperId = Super.cid
  const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {}) 
  if (cachedCtors[SuperId]) {   //缓存中已经存在这个子类直接返回
    return cachedCtors[SuperId]
  }
  const name = extendOptions.name || Super.options.name
  if (Process.env.NODE_ENV !== 'production') {
    if (!/^[a-zA-Z][\w-]*$/.test(name)) {     //名称不符合规范时警告
      warn('')
    }
  }
  const Sub = function VueComponent (options) {
    this._init(options)
  }
  Sub.prototype = Object.create(Super.prototype)
  Sub.prototype.constructor = Sub
  Sub.cid = cid++

  Sub.options = mergeOptions(     //继承options属性，该函数将两个对象的options属性合并后返回新的options对象
    Super.options,
    extendOptions
  )

  Sub['super'] = Super

  if (Sub.options.props) {
    initProps(Sub)
  }

  if (Sub.options.computed) {
    initComputed(Sub)
  }

  Sub.extend = Super.extend
  Sub.mixin = Super.mixin
  Sub.use = Super.use

  //ASSET_TYPES = ['component', 'directive', 'filter']
  ASSET_TYPES.forEach(type => {
    Sub[type] = Super[type]
  });

  if (name) {
    Sub.options.components[name] = Sub
  }

  Sub.superOptions = Super.options
  Sub.extendOptions = extendOptions
  Sub.sealedOptions = extend({}, Sub.options)

  //缓存构造函数
  cachedCtors[SuperId] = Sub
  return Sub
}

```

理解：创建一个Sub继承父级extend、mixin、use、component、directive、filter属性，同时增加superOptions、extendOptions和sealedOptions属性。

2、Vue.nextTick([callback, context])、Vue的set（target， key， value），Vue.delete(target, key)	前面文章已讲过

3、Vue.directive(id, [definition])、Vue.filter(id, [definition])、Vue.component(id, [definition])实现形式相同，而且在Vue源码中，其确实也是在一处实现的

```javascript
/**
 * Vue.directive()、Vue.filter()、Vue.component()
 */

Vue.options = Object.create(null)

//ASSET_TYPES = ['component', 'directive', 'filter']
ASSET_TYPES.forEach(type => {
  Vue.options[type + 's'] = Object.create(null)
})

ASSET_TYPES.forEach(type => {
  Vue[type] = function (id, definition) {
    if (!definition) {       //获取
      return this.options[type + 's'][id]
    } else {
      if (type === 'component' && isPlainObject(definition)) {    //注册组件
        definition.name = definition.name || id
        definition = Vue.extend(definition)
      }

      if (type === 'directive' && typeof definition === 'function') {   //自定义指令
        definition = {bind:definition, update:definition}
      }

      this.options[type + 's'][id] = definition   //挂载到options上
      return definition
    }
  }
})
```

4、Vue.use（plugin）

用法：安装Vue.js插件，如果插件是对象须提供install方法，是函数则其自身就为install方法

```javascript
/**
 * Vue.use（plugin)
 */

 Vue.use = function (plugin) {
   const installPlugins = (this._installPlugins || (this._installPlugins = [])) //  存放全局插件的数组
   if (installPlugins.indexOf(plugin) > -1) {   //    已有该插件直接返回
     return this
   }

   //其他参数
   const args = toArray(arguments, 1)   //除了第一个参数所有参数复制给args
   args.unshift(this)   //将Vue添加到args第一个元素
   if (typeof plugin.install === 'function') {    //  传入的是对象
     plugin.install.apply(plugin, args)
   } else if (typeof plugin === 'function') {     //传入的是方法
     plugin.apply(null, args)
   }
   installPlugins.push(plugin)
   return this
 }
```

5、Vue.mixin(mixin)

用法：全局注册混入一个mixin，影响注册之后创建的每一个Vue.js实例

```javascript
 /**
  * Vue.mixin(mixin)
  */

Vue.mixin = function (mixin) {
  this.options = mergeOptions(this.options, mixin)    //将mixin和原本的options合并成一个对象
  return this
}
```

由于每次创建Vue实例都会跳用initMixin（），从而使得mixin影响注册之后的每一个组件。

6、Vue.version

用法：提供Vue.js安装版本号

