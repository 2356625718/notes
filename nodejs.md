# nodejs

+ res

  ```javascript
  res.writeHead()  //设置状态码和请求头
  res.write()  		 //返回信息
res.end() 			 //结束回复
  ```
  
  

+ req

  ```javascript
  req.url		//请求路径
  ```

  

+ http

  ```javascript
  http.createServer((req, res) => {
    
  }).listen(port)		//创建服务器
  ```

  

+ url

  ```javascript
  url.parse(url, true)		//解析资源通配符，并将query解析为对象
  ```

  

+ commonjs

  ```javascript
  exports.fn = function () {}		//暴露多个方法
  module.exports = {
    
  }  //暴露模块，所有要暴露的写在里面
  ```

  

+ fs

  ```javascript
  /**
   * 检测是文件还是目录
   */
  
  const fs = require("fs");
  
  fs.stat('./test.js', (err, data) => {
    if (err) {
      console.error(err)
    }
    console.log(`是文件: ${data.isDirectpry}`)
    console.log(`是目录: ${data.isFile()}`)
  })
  
  /**
   * 创建目录,已存在则返回错误信息
   */
  
   fs.mkdir('./static', (err) => {
     if (err) {
       console.error(err)
       return
     }
     console.log("创建成功")
   })
  
  /**
   * 创建文件
   */
  
   fs.writeFile('./static/index.html', '<div>hello 罗雅</div>' , (err) => {
     if (err) {
       console.error(err)
       return
     }
     console.log("创建成功")
   })
  
  /**
   * 追加文件，不存在时创建
   */
  
  fs.appendFile('./static/index.css', 'h2{color:red}\n', err => {
    if (err) {
      console.error(err)
      return
    }
    console.log('追加成功')
  })
  
  /**
   * 读取文件
   */
  
   fs.readFile('./static/index.css', (err, data) => {
     if (err) {
       console.error(err)
       return
     }
     console.log(data)
     console.log(data.toString())
   })
  
   /**
    * 读取目录,返回当前目录下文件和文件夹名称的数组
    */
  
    fs.readdir('./', (err, data) => {
      if (err) {
        console.error(err)
        return
      }
      console.log(data)
    })
  
    /**
     * 文件重命名和文件移动
     */
  
     fs.rename('./a.text', './static/index.css', (err) => {
       if (err) {
         console.error(err)
         return
       }
       console.log('移动文件成功')
     })
  
  /**
   * 删除文件
   */
  
   fs.unlink('./static/index.html', (err) => {
     if (err) {
       console.error(err)
       return
     }
     console.log("删除成功")
   })
  
  /**
   * 删除目录,需要目录为空
   */
  
   fs.rmdir('./static', err => {
     if (err) {
       console.error(err)
       return
     }
     console.log("删除目录成功")
   })
  
  /**
    * 写入文件流，推荐,没有文件时会创建文件
    */
  
    const writer = fs.createWriteStream('./b.txt')
    writer.write('hehe')
    writer.end()
    writer.on('finished', () => {
      console.log("写入成功")
    })
  
  
  **
   * 读取并创建文件写入,pipe()
   */
  fs.mkdir('./static', (err) => {
    if (err) {
      console.error(err)
    }
  })
  
  const reader = fs.createReadStream('./a.jpg')
  
  const writer = fs.createWriteStream('./static/b.jpg')
  
  reader.pipe(writer)
  ```

  ```javascript
  
  
  ```
  
  