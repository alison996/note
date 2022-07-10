###  一、react生命周期

#### 1.不推荐使用的生命周期函数

​	1）componentWillMount()

​	2）componentWillUpdate()

​	3）componentWillReceiveProps()

​	这三个生命周期函数并无多大意义，且后期实现异步渲染后可能存在问题，故废弃。

#### 2.新的生命周期函数

​	1）getDerivedStateFromProps()	// 获取衍生的state，无法通过setState()修改

​	2）getSnapshotBeforeUpdate()	// 获取更新前的快照，可应用于滚动条停留场景

### 二、react 请求代理

#### 1.新建setupProxy.js文件

#### 2.配置setupProxy.js文件

```javascript
const proxy = require('http-proxy-middleware')

module.exports = function(app){
	app.use(
		proxy('/api',{
			target:	'http://localhost:5000',	// 目标服务器地址、端口
			changeOrigin: true,					//	改变请求服务器地址，避免目标服务器端做过多校验
             secure: false, // 是否验证证书
             ws: true, // 启用websocket
			pathRewrite:{					// 路径重写
				'^api': ''
			}
		})
	)
}
```

### 三、react组件间通信

#### 1.父子组件

​	1）父传子：props

​	2）子传父：调用props中父组件传入的函数

#### 2.兄弟组件

​	消息订阅-发布机制：

​		1）安装pubsub-js：yarn add pubsub-js

​		2）发布消息

```javascript
import PubSub from 'pubsub-js'
// const PubSub = require('pubsub-js')	 // CommonJS写法

PubSub.publish(消息名称,参数)		// 发布消息

var token = PubSub.subscribe(消息名称,函数名称)		// 订阅消息
PubSub.unsubscribe(token)	//取消订阅
```

### 四、react路由

#### 1、SPA（Single Page Application）的理解

​	1）整个应用只有 ==一个完整的页面==

​	2）点击页面中的链接 ==不会刷新== 页面，只会做页面的 ==局部刷新== 

​	3）数据需要通过`ajax`或`fetch`（低版本浏览器不支持）获取，在前端异步展现

#### 2、react-router-dom的理解

​	1）react的一个插件库

​	2）专门用来实现一个SPA应用

​	3）基于react的项目基本都会用到

#### 3、react-router-dom相关API

```react
import React, { Component }  from 'react'
// 引入路由标签
import { HashRouter, BrowserRouter, Link, NavLink, Switch, Route } from 'react-router-dom'
// 引入页面组件
import { Comps } from './Comps'

export default class Demo extends Component {
    render () {
        {/* hash路由组件 */}
        <HashRouter>
       	{/* <BrowserRouter>	只访问前端资源，不会访问后端资源 */}
            <Link to="/comp">跳转</Link>
        	{/* 注册路由 */}
        	<Switch>
        		<Route path="/comp" component={Comps}></Route>
        	</Switch>
        {/* <BrowserRouter> */}
        </HashRouter>
    }
}
```

​	1）路由组件与一般组件的区别

​			①写法不同

​			②接收的props不同，路由组件==默认有history、location、match属性==

​	2）NavLink与Link的区别：NavLink会根据当前路由地址==高亮==

#### 4、路由组件传参

​	1）传递params参数：

```react
// 传递
<Link to=`/xxx/${val}`>传递参数</Link>
// 路由注册
<Route path="/xxx/:val"></Route>
// 接收
const { val } = this.props.match
```

​	2）传递search参数（需自己转换为JSON）：

```react
import qs from 'querystring'
// 传递
<Link to="/xxx/?val1=xxx&val2=xxx">传递参数</Link>
// 接收
const { search } = this.props.location
// 用qs解析
const { val1, val2 } = qs.parse(search.slice(1))
```

​	3）传递state参数：

```react
// 传递
<Link to={{pathname:'/xxx',state:{val1:xxx}}}>传递参数</Link>
// 接收
const { val1 } = this.props.location.state
```

#### 5、编程式路由

​	1）this.props.history.push(path,state)

​	2）this.parop.history.replace(path)

​	3）this.parop.history.goBack()

​	4）this.parop.history.goForward()

​	5）this.parop.history.go(n)

​	注：**<u>一般组件用withRouter处理一下才可使用history对象</u>**

#### 6、BrowserRouter与HashRouter的区别

​	1）底层原理不同：

​		BrowserRouter使用的是==H5的history API==，<u>不兼容IE9及以下版本</u>

​		HashRouter使用的是URL的哈希值

​	2）url表现形式不一样：

​		BrowserRouter路径中没有#

​		HashRouter路径中有#

​	3）刷新后对路由state参数的影响：

​		BrowserRouter没有任何影响，因为==state保存在history对象中==

​		HashRouter刷新后会导致路由state参数的丢失

​	注：**<u>HashRouter可以用于解决一些路径错误相关的问题</u>**

### 五、redux

#### 1、redux的三个核心概念

​	1）action：

​		①动作对象

​		②包含2个属性

​			type：标识属性，值为字符串、唯一、必需

​			data：数据属性，值为任意类型，可选

​	2）reducer：

​		①用于初始化、加工状态

​		②加工时根据旧的state和action，产生新的state的==纯函数==

​	3）store：

​		①将state、action、reducer联系在一起的对象

#### 2、创建redux

​	1）控制台中运行yarn add redux，安装redux

​	2）创建reducer

```javascript
// 初始化数据
const initState = 0

export default function demoReducer(preState=initState,action) {

  // 从action对象中获取：type、data
  const { type, data } = action

  // 根据type用switch进行对应处理
  switch (type) {
    case 'value':
      return data;
    default:
      return preState
  }

}
```

​	3）创建action（用于统一生成action对象）

```javascript
// 引入store
import store from './store'

// 同步action，返回值为action类型对象
export const demoAction = data => ({type:'value',data})
// 异步action，返回值为函数类型(因为函数里面能开启异步任务)
export const demoAsyncAction = data => {
    return (dispatch) => {
        setTimeout(() => {
            dispatch(demoAction(1))
        },1000)
    }
}
```

​	4）创建store

​			项目中创建store.js文件

```javascript
// 引入createStore，专门用于创建store
import { createStore, applyMiddleware } from 'redux'
// 引入redux-thunk，用于支持异步action
import thunk from 'redux-thunk'
// 引入reducer
import demoReducer from './demoReducer'

// 暴露store
/* 普通store
* export default store = createStore(demoReducer)
*/
// 可使用异步action的store
export default store = createStore(demoReducer,applyMiddleware(thunk))
```

​	4）组件中操作store

```react
// 引入store
import store from './store'
// 引入action
import { demoAction, demoAsyncAction } from './action'

// 获取state
store.getState()
// 通知reducer
/*
* 自已定义action对象
* store.dispatch({type: 'value',data: 1})
*/
// 使用同步action
store.dispatch(demoAction(1))
// 使用异步action

// 在index.js中监听state的改变，并渲染
store.subscribe(() => {
    ReactDom.render(
    	<BrowserRouter>
        	<App />
        </BrowserRouter>,
        document.getElementById('root')
    )
})

// 在组件中监听state的改变,重新渲染（代码冗余）
/*componentDidMount(){
    store.subscribe(() => {
        this.setState({})
    })
}*/
```

### 六、react-redux

#### 1、react-redux思想：

​	只能在容器组件中使用store相关API，UI组件只负责页面的呈现

#### 2、连接容器组件

```react
// 引入UI组件
import DemoUI from './demoUI'
// 引入connnect用于连接UI组件与容器组件
import { connect } from 'react-redux'
/*
* store由容器组件的父组件传入
* <DemoContainer store={store} />
*/

// mapStateToProps函数的返回值作为状态传递给UI组件，默认传入store中的state
function mapStateToProps(state){
    return { state }
}

// mapDispatchToProps函数的返回值作为操作状态的方法，默认传入dispatch方法
function mapDispatchToProps(dispatch){
    return {
        func: (value) => {
            dispatch({type: 'demo', data: value})
        }
    }
}

// 使用connect()()创建并暴露容器组件
export default connect(mapStateToProps,mapDispatchToProps)(DemoUI)
```

