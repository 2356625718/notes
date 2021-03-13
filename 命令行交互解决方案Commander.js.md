# 命令行交互解决方案Commander.js

1、安装```yarn add commander```

2、在index.ts根据官方配置添加如下代码：

```typescript
commander
  .version("6.2.1")
  .option("-p, --peppers", "Add peppers")
  .option("-P, --pineapple", "Add pineapple")
  .option("-b, --bbq-sauce", "Add bbq sauce")
  .option("-c --cheese [type]", "Add the specified type of cheese [marble]","marble")
  .parse(process.argv)
```

此时process会报错提示无此类型

3、```yarn add @types/node```安装Node.js的类型信息

4、此时就可以```yarn ts-node index.ts -h```获取我们新添加的命令行交互选项了

# 给命令行添加色彩colors

1、```yarn add colors```

2、导入和使用

```typescript
import colors from 'colors'
import commander from "commander"
const command = commander
            .version("0.1.0")
            .option("-c --city [string]", "Add city name")
            .parse(process.argv)

if (!command.city) {
  command.outputHelp(colors.red)		//命令行颜色
  process.exit()
}
```

