# eggjs

+ 创建项目

  ```javascript
  npm i egg-init -g
  egg-init demo --type=simple
  ```

  

+ 目录结构

  ```javascript
  app目录下
  controller: 控制器
  extend: 扩展
  middleware: 中间件
  public: 静态资源
  service: model模型，和数据库打交道
  view: 视图、模版
  router.js: 路由模块化
  ```

  

+ 获取get传值

  ```javascript
  this.ctx.query
  ```

  

+ 请求数据

  ```
  this.ctx.curl(url)
  ```

  

+ eggjs内置对象

  ```javascript
  Application
  Context
  Request
  Response
  Helper
  可以在extend文件夹下写对应的js文件对相应对象进行扩展，注意helper最后会挂在到ctx上
  ```

  

+ 中间件

  ```javascript
  module.exports = (options, app) => {
    //options为传入的参数
    console.log(options)
    return async (ctx, next) => {
      console.log(new Date())
      next()
    }
  }
  
  //注册及传递参数
  config.middleware = ['printDate'];
  
    config.printDate = {
      msg: "传入参数"
    }
  ```

+ 更改启动端口

  ```javascript
    config.cluster = {
      listen: {
        path: '',
        port: 3000,
        hostname: '127.0.0.1'
      }
    }
  ```

  

+ post参数传递：

  egg有csrf安全机制，会在请求时在客户端存储为csrf的cookie，传送来的post数据中需要有key为_csrf、值为csrf的属性

  post数据存储在ctx.request.body中。

+ cookie

  this.ctx.cookies.set() //设置cookie

  This.ctx.cookies.get() //获取cookie

  this.ctx.cookies.set(key, null) //清除cookie

  cookie的加密选项为encrypt为true时可以设置中文cookie

  ```javascript
  常用选项
  this.ctx.cookies.set(key, val, {
    maxAge: 1000 * 60 * 60 * 24,
    httpOnly: true,		//只能http访问
    signed: true,			//用户不可更改
    encrypt: true,    //加密且能使用中文cookie
  })
  ```

  

+ session

  ```javascript
  this.ctx.session.key = {...}  //设置session
  this.ctx.session.key //获取session
  设置session的参数
  config.session = {
                          key : 'session_id',
                          maxAge: 1000 * 60 * 60 * 24,
                          httpOnly: true,
                          encrypt: true,
                          renew: true  //每次都重新更新session过期时间
                         }
  ```
  
  