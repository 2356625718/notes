[Vue源码解析] 解析器

解析器类型：HTML解析器、文本解析器、注释解析器

原理：将模版字符串按照一定的条件循环进行切割，对切割下来的内容进行类型判断，在生产对应类型的AST节点插入到父节点上，直到切割完毕最终生成一颗AST树。

逻辑概览：

```javascript
/**
 * 整体逻辑
 */

export function parseHTML (html, options) {
  while (html) {
    if (!lastTag || !isPlainTextElement(lastTag)) {
      let textEnd = html.indexOf('<')
      if (textEnd === 0) {
        //注释
        if (Comment.test(html)) {
          //注释的处理逻辑
          continue
        }

        //条件注释
        if (conditionalComment.test(html)) {
          //条件注释的处理逻辑
        }

        //DOCTYPE
        const doctypeMatch = html.match(doctype)
        if (doctypeMatch) {
          //DOCTYPE的处理逻辑
          continue
        }

        //结束标签
        const endTagMatch = html.match(endTag)
        if (endTagMatch) {
          //结束标签的处理逻辑
          continue
        }

        //开始标签
        const startTagMatch = parseTag()
        if (startTagMatch) {
          //开始标签的处理逻辑
          continue
        }

        //以上情况都不是则不是以<开头的模版字符串,找不到对应的AST节点
        let text, rest, next
        if (textEnd >= 0) {
          //解析文本
        }
        //找不到<符号，则该字符串全部为文本
        if (textEnd < 0) {
          text = html
          html = ''
        }

        if (options.chars && text) {
          options.chars(text)
        }
      } else {
        //父元素为script、style、textarea的处理逻辑
      }
    }
  }
}
```

就是通过正则表达式来判断字符串对应的AST节点类型，其中script、style和textarea标签是纯文本内容元素，当作文本节点处理。



在这里举个例子，比如元素标签,首先找到<符号，然后通过正则表达式判断其不是<--开头的注释节点，也不是</开头的结束标签，也不是没有>符号闭合的字符串和DOCTYPE标签，然后截取标签名，然后通过循环匹配正则表达式截取其属性，最后截取结束符号>或/>,如果是以/>结尾，则将他的自闭合属性设为true。

在字符串截取生成AST节点过程中，使用栈来维护DOM层级，开始标签进栈，知道结束标签才能出栈，通过在栈中的进出栈顺序便可以明确其DOM层级结构。

同时对于文本解析的时候还有一个需要注意的地方，在Vue中，存在带变量的文本，例如```姓名:{{name}}```，所以在解析文本的时候还需要使用正则表达式进行逻辑判断并分类处理，其代码如下：

```javascript
/**
 * 解析带变量的文本
 **/
function parseText(text) {
  const tagRE = /\{\{((?:.|\n)+?)\}\}/g
  if (!tagRE.test(text)) {
    return
  }

  const tokens = []
  let lastIndex = tagRE.lastIndex = 0
  let match, index
  while ((match = tagRE.exec(text))) {
    index = match.index
    if (index > lastIndex) {
      tokens.push(JSON.stringify(text.slice(lastIndex, index)))
    }

    tokens.push(`_s(${match[1].trim()})`)	//_s为toString函数的别名
    lastIndex = index + match[0].length
  }
  if (lastIndex < text.length) {
    tokens.push(JSON.stringify(text.slice(lastIndex)))
  }
  return tokens.join('+')
}

console.log(parseText("姓名:{{name}},年龄:{{age}}"))
```

结果如图：

![image-20201128131041271](https://tva1.sinaimg.cn/large/0081Kckwgy1gl4t5qvca5j30c801cgll.jpg)

