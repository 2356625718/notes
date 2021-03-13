[Vue源码解析]  VNode类型

VNode类型：注释节点、文本节点、元素节点、组件节点、函数式节点、克隆节点

VNode类：

```javascript
/**
 * VNode类
 */

 export default class VNode{
   constructor (tag, data, children, text, elm, context, componentOptions, asyncFactory) {
     this.tag = tag
     this.data = data
     this.children = children
     this.text = text
     this.elm = elm
     this.ns = undefined
     this.context = context
     this.functionalContext = undefined
     this.functionalOptions = undefined
     this.functionalScopedId = undefined
     this.key = data && data.key
     this.componentsOptions = componentOptions
     this.componentInstance = undefined
     this.parent = undefined
     this.raw = false
     this.isStatic = false
     this.isRootInsert = true
     this.isComment = false
     this.isCloned = false
     this.isOnce = false
     this.asyncFactory = asyncFactory
     this.asyncMeta = undefined
     this.isAsyncPlaceholder = false
   }

   get child () {
     return this.componentInstance
   }
 }
```

1、注释节点

```javascript
 /**
  * 注释节点
  */

export const createEmptyVNode = text => {
    const node = new VNode()
    node.text = text
    node.isComment = true
    return node
  }
```

2、文本节点

```javascript
/**
 * 文本节点
 */

 export function createTextNode (val) {
   return new VNode(undefined, undefined, undefined, String(val))
 }
```

3、克隆节点:由于静态节点和插槽节点创建后由于内容不会改变，因此尽量避免再次执行渲染函数生成这种节点的VNode，此时采用克隆节点进行复制可以节省系统性能。

```javascript
 /**
  * 克隆节点
  */

export function cloneVNode (vnode, deep) {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  )
  cloned.ns = vnode.ns
  cloned.isStatic = vnode.isStatic
  cloned.key = vnode.key
  cloned.isComment = vnode.isComment
  cloned.isCloned = true
  if (deep && vnode.children) {
    cloned.children = cloneVNodes(vnode.children)
  }
  return cloned
}
```

4.元素节点

```javascript
/**
 * 元素节点
 */

 {
   children: [VNode,VNode],   //  子节点列表
   context: {},   //当前组件的实例
   data: {},      //节点上的数据，如attr、class、style
   tag: "p",      //标签名
 }
```

5.组件节点

```javascript
/**
 * 组件节点
 */

{
  componentInstance: {},  //组件的实例
  componentOptions: {},   //组件节点的参数，如propsData，tag，children
  context: {},   //当前组件的实例
  data: {},      //节点上的数据，如attr、class、style
  tag: "vue-component-1-child",      //标签名
}
```

6.函数式节点

```javascript
/**
 * 函数式节点
 */

{
  functionalInstance: {},  //函数的实例
  functionalOptions: {},   //函数节点的参数
  context: {},   //当前组件的实例
  data: {},      //节点上的数据，如attr、class、style
  tag: "div",      //标签名
}
```

