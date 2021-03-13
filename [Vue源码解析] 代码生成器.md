# [Vue源码解析] 代码生成器

代码生成器：将AST节点转换为代码字符串，代码字符串就是我们交给渲染函数的内容。



代码字符串中具有生成VNode的方法，我们先罗列出这三种方法：

1、createElement，创建元素VNode，别名_c,参数1：tag名称，参数2:包含属性的对象，参数三：子节点列表

2、createTextVNode, 创建文本VNode，别名_v，参数：文本

3、createEmptyVNode, 创建注释VNode，别名_e，参数：文本



假如有一个模版为:

```html
<div id="el">
  <div>
    <p>
      Hello {{name}}
    </p>
  </div>
</div>
```



先将其解析为AST树（由于太长就不写了），然后通过代码生成器将其根节点生成代码字符串：

```javascript
with (this) {
  return _c(
  	'div',
    {attrs:{"id":"el"}},
    [_c(['div',[_c('p',[_v("Hello"+_s(name))])]])]
  )
}
```

然后就可以将它交给渲染函数生成VNode了



代码生成器的原理：

生成元素节点代码字符串：

```javascript
/**
 * 生成元素节点代码字符串
 */

function genElement (el, state) {
  //如果el.plain是true，则说明该节点没有属性
  const data = el.plain ? undefined : genData(el, state)

  const children = genChildren(el, state)
  code = `_c('${el.tag}'${
    data ? `,${data}` : ''
  }${
    children ? `,${children}` : ''
  })`
}
```

生成data：

```javascript
//生成data

function genData (el:ASTElement, state: CodegenState): string {
  let data = '{'
  if (el.key) {
    data += `key:${el.key}`
  }
  if (el.ref) {
    data += `key:${el.ref}`
  }
  //。。。。。。el的属性还有很多种情况
  data = data.replace(/,$/,'') + '}'
  return data
}
```

生成子节点列表字符串

```javascript
//生成子节点列表字符串

function genChildren (el, state) {
  const children = el.children
  if (children.length) {
    return `[${children.map(c => genNode(c,state)).join(',')}]`
  }
}

function genNode (node, state) {
  if (node.type === 1) {
    return genElement(node, state)
  } if (node.type === 3 && node.isComment) {
    return genComment(node)
  } else {
    return genNode(node)
  }
}
```

生成文本节点代码字符串：

```javascript
/**
 * 生成文本节点代码字符串
 */

function genText (text) {
  return `_v($(text.type === 2
    ? text.expression
    : JSON.stringify(text.text)   //
    ))`
}
```

注释节点代码字符串：

```javascript
/**
 * 注释节点代码字符串
 */

function genComment (comment) {
  return `_e(${JSON.stringify(comment.text)})`
}
```



