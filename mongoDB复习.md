# mongoDB复习

1、创建使用数据库： use name

2、查看数据库： show databases

3、查看表：show collections

4、创建表：db.表名.insert（{})

5、查询表中数据：db.表名.find()

6、删除数据库：db.dropDatabase()

7、大于：gt，小于：lt，大于等于：gte，小于等于：lte

```javascript
例如查询年龄大于23小于25岁的学生
db.stduent.find({"age":{$gt: 23, $lt: 25}})
```

8、可以使用正则表达式进行查找：/reg/

9、升序降序：db.表名.find().sort({键名:1||-1}),1为升序，-1为降序

10、限制条数：db.表名.find().limit(10)

11、跳过指定条数：db.表名.find().skip(10)

12、统计数量：db.表名.find().count()

13、一次插入多条数据:

```javascript
for (var i = 0; i <= 99; i++) {
  db.user.insert({"name":"hah" + i, "age": i})
};		//;号表示结束
```



14、or查询：

```ja
db.user.find({$or:["age":16,"age": 13]})
```

15、查询一条数据:

```javascript
db.findOne()
```

16、修改增添属性：

```javascript
db.user.update({"name": "张三"}, {$set: {"name": "王武", "sex": "男"}}, {multi: true} )//第三个参数表示一次修改多个
```

17、删除数据：

```javascript
db.user.remove({})	//删除所有
db.user.remove({"age": {$gte: 20}})
db.user.remove({"age": {$gte: 20}}, {justOne: true}) //只删除一条

```

18、索引操作

```javascript
db.tableName.ensureIndex({属性名:1 || -1})		//1升序2降序
db.tableName.getIndexs()	//获取所有索引
db.tableName.dropIndex({属性名: 1 || -1})  //删除索引
```

19、创建管理员

```javascript
use admin //使用系统用户库

db.createUser({user: '用户名', pwd: '密码', roles: [{role: 'root', db: 'admin'}]})

用户角色有超级用户角色:root, 数据库用户角色: read、readWriter, 数据库管理角色: dbAdmin, dbOwner
```

20、管道操作

db.tableName.aggregate([])

```javascript
管道操作符:
$project 	//等同于select
$match		//等同于where
$limit 		//等同于limit
$skip			
$sort			//等同于order
$group 		//等同于group
$lookup		//等同于join
{$lookup: {
  from: "表名",		//关联表
  localField: "属性名",	//当前表的关联属性
  foreignFiled: "属性名",	//关联表的关联属性
  as: "items"	// 关联生成的属性
}}


常用表达式操作符:
$addToSet 	//指定字段去重
$max		//最大值
$min		//最小值
$sum		//和
$avg		//平均值
$gt			//大于
$lt			//小于
```

21、nodejs操作mongodb

```javascript
const { MongoClient } = require('mongodb')

const url = "mongodb://127.0.0.1:27017"

const client = new MongoClient(url, { useUnifiedTopology: true })

client.connect(err => {
  if (err) {
    console.error(err)
    return
  }
  console.log("连接成功")
  const db = client.db(dbName)
  db.collection('zy').find().toArray((err, data) => {
    if (err) {
      console.error(err)
      return
    }
    console.log(data)
  })

  db.collection('zy').insertOne({name: "哈哈", age: 20}, (err) => {
    if (err) {
      console.error(err)
      return
    }
  })
})
```

