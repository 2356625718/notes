# [Vue源码解析] 生命周期、指令系统、过滤器

分为四个阶段：初始化阶段、模版编译阶段、挂载阶段和卸载阶段

errorCaptured与错误处理：用户代码的错误会通过handleError函数唤起errorCaptured钩子函数，这个函数的作用是捕获子孙组件的错误，可返回false避免错误继续向上传播，如果全局的config.errorCaptured被定义，那么所有的错误也会发给他。

初始化实例属性：initLifeCycle挂载$paren,$root,$children,$refs和内部属性_watcher,_isDestroyed和_isBeingDestroyed

初始化事件：initEvents将v-on监听的函数添加到vm._events中

初始化inject：inject/provide通过祖先组件向子孙组件注入依赖，通过resolveInject自底向上查找inject参数中对应的provide

初始化状态：props、methods、data、computed、watch初始化

+ 初始化props：会从vm.$options.propsData和vm._props中取传递的props键和值然后根据propsOptions用户的选项获取，横线形式的props会转换为小驼峰形式。

+ 初始化methods： 将methods中的方法依次挂载到vm上

+ 初始化data：如果是函数后执行再赋值，对比props有无同名属性，最后将data挂载到vm._data上，并设置代理使用户可以通过vm.state访问vm._data.state

+ 初始化computed：会生成一个计算属性的watcher，属性变化时会通知计算属性的watcher，再让计算属性的watcher通知组件的watcher重新渲染，为什么要这么做呢，因为计算属性需要缓存，我们可以在计算属性的watcher和组件的watcher上设置一个dirty属性，计算属性的dirty为true时表示再次读取计算属性时需要重新计算，重新缓存，且通知组件watcher重新渲染，这时组件watcher会去读取计算属性进而重新计算。

+ 初始化watch：遍历用户提供的watch，调用vm.$watch创建watcher。

  

常用指令在模版编译阶段会编译成为directives属性，最终会传入VNode中，新旧VNode对比完成后会根据对比结果触发bind、inserted、update、componentUpdated与unbind钩子函数，这时指令就生效了。

自定义指令：新旧VNode中存在directives属性时执行_update，通过对新旧指令集合的对比，判断出更新的指令集



过滤器：将管道符号前面的值作为参数传给后面的函数，可以通过管道符号串联多个过滤器，也可以传入参数。

