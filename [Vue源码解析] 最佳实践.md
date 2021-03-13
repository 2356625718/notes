# [Vue源码解析] 最佳实践

1、列表属性和v-if、v-if-else、v-else中使用key

2、路由切换组件不变这是由于两个路由用的是同一个组件，由于组件复用，而不会进行重新渲染。

解决方法：

+ 在watch中监测$route的query值

+ 在router-view组件上添加key（不推荐）



3、为所有路由同一添加query

+ 使用全局守卫beforeEach，有指定query才放行，没有调用next（path+query）二次跳转

+ 使用函数劫持:拦截router.history.transitionTo方法

  ```javascript
  const query = {referer: 'hao360cn'}
  const transitionTo = router.history.transitionTo		//保存原函数
  
  router.history.transitionTo = function (location, onComplete, onAbort) {		//对原函数改变指向，进行函数劫持
    location = typeof location === 'object'
    ? {...location, query: {...location.query, ...query}}		//将原本的query和我们要添加的统一query通过扩展运算符合并在一起
    : {path: location, query}  //没有query参数时手动添加query参数
    
    transitionTo.call(router.history, location, onComplete, onAbort)		将原函数释放回来，注意此时的location.query已经发生了我们所期望的改变。
  }
  ```

  

4、避免v-if和v-for一起使用

+ 过滤一个列表中的部分项时请使用计算属性

+ 过滤一个列表时请将v-if提到列表的副属性上去

  

5、为组件样式域有两种选择，scoped（避免使用元素选择器）和module（推荐）

6、在vuetemplate中自定义组件不能是自闭合组件