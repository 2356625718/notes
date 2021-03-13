# TypeScript项目初始化

1. git init初始化git本地仓库，创建配置.gitignore文件忽视某些文件,推荐配置：

   ```javascript
   # Numerous always-ignore extensions  
   *.bak  
   *.patch  
   *.diff  
   *.err  
   
   # temp file for git conflict merging  
   *.orig  
   *.log  
   *.rej  
   *.swo  
   *.swp  
   *.zip  
   *.vi  
   *~  
   *.sass-cache  
   *.tmp.html  
   *.dump  
   
   # OS or Editor folders  
   .DS_Store  
   ._*  
   .cache  
   .project  
   .settings  
   .tmproj  
   *.esproj  
   *.sublime-project  
   *.sublime-workspace  
   nbproject  
   thumbs.db  
   *.iml  
   
   # Folders to ignore  
   .hg  
   .svn  
   .CVS  
   .idea  
   node_modules/  
   jscoverage_lib/  
   bower_components/  
   dist/  
   ```

2. 在项目文件夹内```yarn add ts-node typescript --dev```安装ts-node和typescript

3. ```yarn tsc --init```初始化，会在项目根目录生成tsconfig.json文件，这是typeScript的配置文件

4. ```yarn add tslint --dev```安装tslint*typeScript代码的样式风格检查工具,```yarn tslint --init```初始化生成tslint.json文件

5. ```yarn add husky```安装哈士奇在package.json文件中devDependencies后添加配置

   ```json
   "husky": {
   
   ​    "hooks": {
   
   ​      "pre-commit": "yarn tslint -c tslint.json './src/*.ts"
   
   ​    }
   
     }
   ```

   

   src目录下的ts文件在每次提交前都执行一次tslint的检查命令

 