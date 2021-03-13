# express

+ 静态资源配置: app.use(express.static('./static'))

+ post数据解析

  App.use(bodyParser.urlencoded({extended: false}))

  app.use(bodyParser.json())

  Req.body获取数据

+ cookie处理：cookie-parser

  ```javascript
  const cookieParser = require('cookie-parser')
  app.use(cookieParser())
  //设置cookie
  res.cookie(key, val, {maxAge: 过期时间})
  //获取cookie
  req.cookies.key
  
  
  //cookie设置的参数
  maxAge:过期时间
  signed:boolean 加密
  expries: Date 过期时间
  httpOnly: boolean 为true时只能后台访问
  path: string 指定可以访问cookie的路径
  domain：string 二级域名可以共享cookie
  sucure: boolean cookie在http协议中无效、在https中才有效
  
  //cookie的加密
  app.use(cookieParser(加密字符串))
  //设置cookie
  res.cookie(key, value, {maxAge: 1000 * 60 * 60 * 24, signed: true})
  //获取cookie
  req.signedCookies
  ```

+ session处理: express-session

  原理：浏览器请求，生成session键值对,返回键给浏览器，下次请求携带键来找值

  ```javascript
  const session = require('express-session')
  
  app.use(session({
    secret: string //签名，
    name: string //修改session对应的cookie名称
    resave: boolean //强制保存,默认为true
    saveUninitialized: boolean //强制将为初始化的session存储,默认为true
    cookie: {
    maxAge: 1000 //过期时间
    secure: boolean //true时只有https才能访问
  }
    rolling: boolean //默认为false，每次请求重置cookie过期时间
    
  }))
  
//设置
  req.session.key = val
  //销毁session
  req.session.cookie.maxAge = 0		//会销毁所有session
  req.session.key = ''		//销毁指定session
  req.session.destroy()		//销毁所有session
  
  //session存储到数据库
  
  const mongoStore = require('connect-mongo')(session)
  
  app.use({
    secret: string,
    name: string,
    resave: true,
    saveUninitialized: boolean,
    cookie: {
      maxAge: 1000 * 60 * 60 * 24
    },
    rolling: false,
    store: new mongoStore{
      url: "mongodb://127.0.0.1:27017/session",
      touchAfter: 24 * 3600 //24小时内只更新一次session
    }
  })
  ```
  
  

+ 生成项目模版

  ```javascript
  npm i express-generator -g		//下载生成器
  
  //生成项目模版
  express demo名称
  ```

  

+ multer上传文件模块处理multipart/form

  ```javascript
  <form action="http://127.0.0.1:3000/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="image" value="" />
        <button type="submit">提交</button>
  </form>
    //服务端
  const path = require('path')
  const multer = require('multer')
  const tools = {
    multer () {
      let storage = multer.diskStorage({
        destination: function (req, file, cb) {
          cb(null, './static')
        },
        filename: function (req, file, cb) {
          console.log(file)
          let filename = new Date().getTime() + path.extname(file.originalname)
          cb(null, filename)
        }
      })
       
      let upload = multer({ storage: storage })
      return upload
    }
  }
  
  module.exports = tools;
  
  //app.js
  app.post('/upload', tools.multer().single('image'), (req, res) => {
    console.log(req.file)
    res.end("ok")
  })
  ```

  

+ 按目录结构生成文件夹: mkdirp

  ```javascript
  const mkdirp = require('mdirp')
  
  mkdirp('/static/upload')
  .then(res => {
    
  })
  ```

  

+ 格式化日期字符串: silly-datetime

  ```javascript
  const sd = require('selly-datetime')
  
  sd.format(new Date(), 'YYYY-MM-DD hh:mm:ss')
  ```

  