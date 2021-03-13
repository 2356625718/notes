# nextjs

+ 服务端渲染的实现

  React-dom/server包中有renderToString（带有额外属性data-react-id，第一个dom会有data-checksum）和renderToStaticMark Mark（不会有额外的属性）将react component转换成html字符串。

​       data-checksum现在已经取消，因为其改变会导致该dom及其子树重新渲染

​		这两个方法不能够编译js脚本

+ 自定义头部

  ```javascript
  import Head from "next/head"
  
  const app = () => {
    return (
      <Head>
      	<title>首页</title>
        <meta charset="UTF-8" />
      </Head>
    )
  }
  ```

  

+ 生命周期:getInitialProps在props初始化时执行，在服务端运行，没有跨源限制

  ```javascript
  static async getInitialProps () {
    return {
      film: {}  //film将作为props数据传入组件
    }
  }
  ```

+ 页面导航式跳转

  ```javascript
  import Link from 'next/link'
  
  <Link href="/else" >其他页面</Link>
  ```

  

+ 编程式挑战

  ```javascript
  import Router from 'next/router'
  
  Router.push(url)
  ```

  

+ 预加载页面

  1、link标签上prefetch属性

  2、withRouter中使用router.prefetch（url）

  ```javascript
  import withRouter from 'next/router'
  
  export default withRouter(({children, router}) => (
  <div>{router.prefetch(url)}</div>
  )
  })
  ```

  

+ 路由常见事件

  routerChangeStart(url)-路由跳转开始

  routerChangeComplete(url)-路由跳转完成

  routerChangeError(url)-路由跳转失败

  beforeHistoryChange(url)-浏览器历史改变

```javascript
Router.events.on('routerChangeStart', (url) => {
  if (url === '/') {
    location.href = '/index'
  }
})
```

+ 打包部署

+ 第一种方式：不使用服务端进行扩展

  1、npm run build

  2、运行到80端口: npm start -p 80

+ 运行express服务器为例：

  ```javascript
  const express = require('express')
  const next = require('next')
  
  const dev = process.env.NODE_ENV !== 'production'
  const app = next({dev})
  
  const handle = app.getRequestHandler()
  
  app.prepare().then(() => {
    const server = express()
    server.get('*', (req, res) => {
      console.log(req)
      return handle(req, res)
    }).listen(3000)
  })
  
  下载cross-env修改环境变量
   "start_e": "cross-env NODE_ENV=production node server.js"  //script中添加
  ```

  