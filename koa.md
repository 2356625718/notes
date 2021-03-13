# koa

```javascript
const koa = require('koa')
const router = require('koa-router')()
const bodyParser = require('koa-bodyparser')

const app = new koa()

app.use(bodyParser())
router
.get('/', async (ctx) => {
  console.log(ctx)
  ctx.body = "index"
})
.get('/admin/:id', async (ctx) => {
  console.log(ctx.query)
  console.log(ctx.quertString)
  console.log(ctx.url)
  console.log(ctx.params)
  ctx.body = "admin"
})
.post('/login', async ctx => {
  console.log(ctx.request.body)
  ctx.body = "成功"
})

app
.use(router.routes())
.use(router.allowedMethods())
.listen(3000)
```

+ post请求参数解析中间件： koa-bodyparser



+ 静态资源配置中间件：koa-static

+ 设置cookie:

  ```javascript
  ctx.cookies.set(key, val, options)
  ctx.cookies.get(key)
  //koa中无法设置中文的cookie，需要转码
  let val = new Buffer("中文").toString("base64")
  ctx.cookies.set(key, val, options)
  //转换回来
  let data = new Buffer(ctx.cookies.get(key), 'base64').toString()
  ```

+ 设置session：koa-session

  ```javascript
  const CONFIG = {
    key: 'session', /** (string) cookie key (default is koa.sess) */
    /** (number || 'session') maxAge in ms (default is 1 days) */
    /** 'session' will result in a cookie that expires when session/browser is closed */
    /** Warning: If a session cookie is stolen, this cookie will never expire */
    maxAge: 86400000,
    autoCommit: true, /** (boolean) automatically commit headers (default true) */
    overwrite: true, /** (boolean) can overwrite or not (default true) */
    httpOnly: false, /** (boolean) httpOnly or not (default true) */
    signed: false, /** (boolean) signed or not (default true) */
    rolling: false, /** (boolean) Force a session identifier cookie to be set on every response. The expiration is reset to the original maxAge, resetting the expiration countdown. (default is false) */
    renew: false, /** (boolean) renew session when session is nearly expired, so we can always keep user logged in. (default is false)*/
    secure: false, /** (boolean) secure cookie*/
    sameSite: null, /** (string) session cookie sameSite options (default null, don't set it) */
  };
   
  app.use(session(CONFIG, app));
  ```

  

+ koa脚手架:koa-generator

  ```
  koa demo
  cd demo
  npm i
  npm start
  ```

  