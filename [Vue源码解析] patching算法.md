# [Vue源码解析] patching算法

pathching算法：通过对比新旧VNode的不同，然后找出需要更新的节点进行更新



操作：1、创建新增节点	2、删除废弃节点	3、修改需要更新的节点



创建节点的三种类型：1、元素节点（tag属性）：document.createElement	2、text属性（text属性和isComment属性为false）:document.createTextNode

3、注释节点（text属性和isComment属性为true）:document.createComment



删除节点：

1、删除多个节点：

```javascript
function removeVnodes (vnodes, startIdx, endIdx) {
  for (; startIdx <= endIdx; ++startIdx) {
    const ch = vnodes[startIdx]
    if (isDef(ch)) {
      removeNode(ch.elm)
    }
  }
}
```

2、删除单个节点

```javascript
const nodeOps = {
  removeChild (node, child) {
    node.removeChild(child)
  }
}

function removeNode (el) {
  const parent = nodeOps.parentNode(el)
  if (idDef(parent)) {
    nodeOps.removeChild(parent, el)
  }
}
```

更新节点：

1、静态节点不需要进行更新，所以更新时首先要判断新旧VNode的该节点是否是静态节点，至于判断依据，在模版解析为AST（抽象语法树）后，会遍历AST找出静态节点做上标记。

2、NewVNode上如果有text属性，直接调用node.TextContent方法进行文本更新。

3、NewVnode没有文本属性时，判断其是否有children属性，没有则其是空节点，直接清空dom上对应节点；如果不为空，则需对NewVNode和OldVMNode上的children节点进行对比才能做出更新。



更新子节点：

先说最垃圾的比对新旧VNode上子节点的方法：从新节点上按顺序取出一个子节点，来和OldVNode的所有children节点进行一一对比，找到且位置相同则更新，找到但位置不同则移动且更新（移动到未处理节点的前面），找不到则插入（插入到DOM中未处理节点前面），在处理过后的新旧VNodeChild上都标记为已处理，若遍历完newVNodeChildren后oldVNode还有child，说明这些节点是废弃的，则进行删除。



优化策略：

都说了上面是最垃圾的算法了，在实际情况中大多数VNode的节点位置其实根本没有发生变化，所以我们只需要对比相同索引位置的新旧VNodeChild就可以了。

方法：1、新未处理第一个比对旧未处理第一个

2、新未处理最后一个对比新未处理最后一个

3、新未处理第一个对比旧未处理最后一个

4、新未处理最后一个对比旧未处理第一个

在以上对比过后如果还没有找到NewVNodeChild在OldVNodeChildren中的位置，再采用循环的笨办法进行寻找。

在这里就要特别提一下key的作用了，在循环时添加key属性就可以建立起key值与数组索引的对应关系，也就是说不要要再去比对新旧VNodeChildren了，直接通过key：index的对应关系便可以快速地找到新旧子节点的位置，这也是Vue为什么推荐我们使用key的原因。



