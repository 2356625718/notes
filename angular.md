# angular

+ 创建组件: ng g component components/news(路径相对于app)

  此时在app.module.ts会引入这个组件并申明这个组件

+ 定义数据：module.ts中使用 public 变量名 = 值，在html文件中使用{{变量名}}使用,不加public默认是public

+ 声明属性的几种方式:1、public:类里面类外面都可以访问 2、protected：当前类和子类可以访问 3、private： 只有当前类可以访问

+ 在标签上绑定属性: [title]="变量名"

+ 渲染变量字符串: 父标签上添加[innerHTML]="变量名"

+ for循环渲染数组: *ngFor="let item of 数组名",循环时带索引值*ngFor="let item of arr;let key=index;"

+ 引入本地图片必须存放在assets文件夹下,通过asstes/图片名路径引入

+ 条件判断：*ngIf="arr.length > 0"

  [ngSwitch]="属性名"

  *ngSwitchCase="条件一"

  *ngSwitchCase="条件二"

  *ngSwitchDefault

+ 动态绑定类名:[ngClass]="{'red':true}"

+ 动态绑定style：[ngStyle]="{'background-color':'red'}"

+ 管道:对today日期字符串进行格式化：{today | date: "yyyy-MM-dd hh:mm:ss"}

+ 事件执行函数:(事件名)="函数名",例：(click)="getDate($event)"

+ 双向数据绑定: form表单中使用

  App.module.ts中引入: import {FormsModule} from '@angular/forms'

  Imports数组里添加FormsModule

  使用:[(ngModel)]="变量名"

+ 创建服务: ng g service services/storage

  在app.module.ts中导入 import {StorageService} from './Service/Storage.service'

  在providers数组中添加该服务

  在要使用的组件内导入该服务，在constructor构造函数中传参该服务便可通过该变量调用服务中的方法

+ 操作dom:

  原生方法:在ngAfterViewInit()生命周期函数内部进行获取和dom操作
  
  angulardom操作方法: @ViewChild(名称)变量名:ElementRef
  
  在获取的dom标签上增添名称:#名称
  
  变量名.nativeElement获取dom元素

+ 父子组件传值

  1、父传子

  父组件[data]="data"传值

  子组件导入Input方法,使用@Input() data: dataType接收数据

  2、子传父

  + 父组件通过ViewChild获取子组件实例。

  + 子组件引入Output和EventEmitter,向父组件广播数据

    ```javascript
    @Output() private out = new EventEmitter<String>()
    send () {
        this.out.emit("hello")
      }
    ```

    父组件通过该变量及事件对象e接收数据

    ```javascript
    <app-new-child [value]="value" #child (out)="rec($event)"></app-new-child>
    rec (e: any) {
        console.log(e)
      }
    ```

    

+ 生命周期函数

  1、ngOnChanges() 当父组件传递的数据发生变化时会触发

​       2、ngOnInit() 第一次数据绑定和传入数据时触发，常用作请求数据

​	   3、ngDoCheck() 检测数据的变化进行对应操作

​      4、ngAfterContentInit() 组件投影初始化完成

​       5、ngAfterContentChecked() 组件投影内容变更时调用

​       6、 ngAfterViewInit() 初始化完主视图及其子视图后调用

​		7、ngAfterViewChecked() 视图变更检测之后调用

​		8、ngOnDestroy() 组件销毁时调用

+ rxjs

  与Promise类似:

  ```javascript
  return new Observable(observer => {
        setTimeout(() => {
          //如果这里面是一个定时方法,rxjs会不断获取最新值，这是与Promise不同的地方
          let data = "rxjs"
          observer.next(data)
        }, 1000)
      })
  
  
  this.getRun().subscribe(data => {
        console.log(data)
      })
  ```

  pipe管道中可以使用map和filter对数据进行处理

+ 网络请求

  1、在app.module.ts中导入HttpClientModule模块并在imports中注入

  ```javascript
  import { HttpClientModule } from '@angular/common/http'
  
  imports: [
      BrowserModule,
      AppRoutingModule,
      FormsModule,
      HttpClientModule
    ],
  ```

  2、在引入的地方引入HttpClient和HttpHeaders并在构造函数声明HttpClient

  ```javascript
  import { HttpClient, HttpHeaders } from "@angular/common/http"
  
  constructor(public http: HttpClient) { }
  ```

  

  Get:

  ```javascript
  this.http.get("http://a.itying.com/api/productlist").subscribe(data => {
        console.log(data)
      })
  ```

  Post:

  ```javascript
  const httpOptions = {
        headers: new HttpHeaders({'Content-Type':'application/json'})
      }
      this.http.post("http://a.itying.com/api/productlist", {username: 'zy'}, httpOptions).subscribe(data => {
        console.log(data)
      })
  ```

  

+ 路由配置

  1、路由表:

  ```javascript
  const routes: Routes = [
    {path: 'a', component: PageaComponent},
    {path: 'b', component: PagebComponent},
    {path: '**', redirectTo: 'a'}
  ];
  ```

  2、跳转链接及显示位置

  ```javascript
  <div>
    <a [routerLink]="['/a']">a</a>
    <hr />
    <a [routerLink]="['/b']">b</a>
    <hr />
    <a [routerLink]="['/c']">c</a>
  </div>
  
  <router-outlet></router-outlet>
  ```

  3、get传值

  ```javascript
  <a [routerLink]="['/c']" [queryParams]="{name:'zy'}">c</a>
  ```

  或者获取route对象

  ```javascript
  import { ActivatedRoute } from '@angular/router'
  constructor(public route: ActivatedRoute) { }
  
    ngOnInit(): void {
      console.log(this.route)
      this.route.queryParams.subscribe(data => {
        console.log(data)
      })
    }
  
  ```

  动态路由:

  ```javascript
  {path: 'b/:id component: PagebComponent}
  ```

    跳转:

  ```javascript
  <a [routerLink]="['/c', 2]">c</a>
  ```

  动态路由获取传值

  ```javascript
  this.route.params.subscribe(data => {
    console.log(data)
  })
  ```

  4、js跳转

  动态路由跳转：

  ```javascript
  this.router.navicate([path, params])
  ```

  get传值跳转：

  ```javascript
  import { NavigationExtras, Router } from '@angular/router'
  //js跳转函数
    go () {
      let queryParams: NavigationExtras = {
        queryParams: {name: "zy"}
      }
      this.router.navigate(['/b', queryParams])
    }
  ```

  5、嵌套子路由:

  ```javascript
  children: {
    {path: '/admin', component: name}
  }
  
  
  跳转 /path/admin
  
  会在<router-link>处显示该子组件
  ```

  

