# redux, thunk,saga学习

## redux

+ 概念：js容器，进行全局的状态管理

+ 特点：1、单一数据源

     		2、state是只读的
	
     	 		3、使用纯函数进行修改（reducer）

  action：在应用内对store进行操作的桥梁，是js对象，通过store.dispatch调用，通常通过action创建函数使用。

  reducer：根据接收到的action对store中的state进行操作，必须有return值。

  store：将action和reducer连接在一起

  ​			职责：1、维持应用的state

  ​                        2、通过getState()获取state

  ​                        3、通过dispatch（action）传递action

  ​                        4、通过subscribe（）来注册监听。

  ​                        5、通过subscribe（）的返回值来取消监听。

  ## react-redux

  + 结合react的redux

  + Provider：包裹根组件，以store属性作为props传递给所有组件

  + connect：调用此方法包裹的组件才能拿到store

    调用方法connect(mapStateToProps, mapDispatchToProps)()

+ combineReducers:将reducer合并为一个reducer，如

  ```javascript
  import { combineReducers } from 'redux'
  
  const reducer = combineReducers({
    home: homeReducer,
    age: ageReducer,
  })
  ```

  

## redux-thunk

+ thunk中间件使我们能够编写异步action creator，dispatch调用时参数可以不只是一个对象还可以是函数（传参dispatch异步任务内部调用返回函数内部dispatch的对象）

  ```javascript
  applyMiddleware(thunk)
  
  //异步任务内部dispatch
  const fetchData = (url) => {
    return dispatch => {
      fetch(url)
      .then(res => {
        res.json
        .then(data => {
          dispatch({type:"fetch_data", ...data})
        })
      })
    }
  }
  
  //实际生效的是异步任务内的dispatch
  const mapDispatchToProps = dispatch => {
    return {
      asyncAction: () => {
        dispatch(fetchData(url))
      }
    }
  }
  ```

## redux-saga

+ 可以访问完整的redux state也可以dispatch redux action

  middleware的API：

  1、createSagaMiddleware（），返回sagaMiddleware

  2、sagaMiddleware.run（defsaga），运行自定义的saga

+ saga辅助函数

  1、takeEvery: 所有触发的action都执行回调generator函数

  2、takeLatest：执行未执行完成的最后一次

  3、throttle：节流，在指定时间内只执行第一次和第二次。

+ effect创建器

  1、select：获取reducer的返回值，可以通过state参数进行过滤

  2、call：调用函数

  3、take：阻塞，直到收到指定的action

  4、put：同dispatch

