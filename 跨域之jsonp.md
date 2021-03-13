1. 跨域问题：由于同源策略，跨协议、主机号、端口号其中之一便会产生跨域问题，导致请求无法发送。

2. 解决方法：主流的前端解决方法为jsonp，其他如iframe或flash（这个好像要淘汰了）未去了解。而且做项目的时候跨域问题好像一般都是后端解决，主要jsonp也就解决get请求跨域，遇见post就无能为力了。

3. jsonp原理：运用了script标签可以跨域请求的特性，动态创建script标签发起请求，通过回调函数获取返回的参数。

4. 代码演示：

   ```javascript
   function ajax(obj){
     var defaults = {
       type:'get',
       async:true,   
       dataType:"jsonp",   
       jsonp:"callback", //默认的回调函数名称
       data:{},
       success:function(data){
         console.log(data);
       }
     }
   
     //传入参数替换默认参数
     for(var key in obj){
       defaults[key] = obj[key];
     }
   
     var cbName;
   
     if(defaults.jsonp){
       cbName = defaults.jsonp;
     }
   
     //调用回调函数,data值为返回参数
     window[cbName] = function(data){
       defaults.success(data)
     }
   
     //参数字符串拼接
     var param = '';
     for(var attr in defaults.data){
       param = attr + '=' + defaults[attr] + '&'
     }
   
     if(param){
       param = param.substring(0,param.length-1);
       param = '&' + param
     }
   
     //动态创建script标签
     var script = document.createElement('script');
     script.src = defaults.url + '?' + "callback=" + defaults.jsonp + param;
     //传给后端回调函数名称和参数
     var head = document.getElementsByTagName('head')[0];
     head.appendChild(script);
   }
   ```

   发送请求

   ```javascript
   <script>
       ajax(
         {
    url:"http://127.0.0.1:4000/test"
         }
       )
   </script>
   ```

   后端简陋代码（偷懒不想解析参数了，用的默认回调函数名称)

   ```javascript
   const express = require("express");
   
   const app = express();
   
   app.get('/test',(req,res)=>{
     var data = {
       name:"我胡汉三又回来啦",
       age:18
     }
     res.send("callback("+
     JSON.stringify(data) +
       ")");
   });
   
   app.listen(4000,()=>{
     console.log("服务器已开启")
   });
   ```

   返回成功

   ![image-20201109161818432](https://tva1.sinaimg.cn/large/0081Kckwgy1gkizt45no4j30vz0u0wiu.jpg)

   好了好了，又是一篇水水的博客完工啦。

