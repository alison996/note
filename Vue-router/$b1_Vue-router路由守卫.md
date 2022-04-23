#### Vue-router路由守卫 ( v4.x )
对比之前版本，需要使用next的地方，现在`默认返回undefined`，进入下一个路由。避免了`未正确使用next而导致的一些错误` ( 比如：未调用next )。
##### 1. 全局前置守卫
`router.beforeEach([to][, from][, next])`
```javascript
router.beforeEach((to, from, next) => {
  // ...

  /* 1. 取消导航 */
  return false;

  /* 2. 进入下一个路由守卫 */
  return true;
  return undefined;

  /* 3. 跳转至其他路由地址 */
  return [url];

  // next 不做赘述
})
```

##### 2. 全局解析守卫
`router.beforeResolve([to][, from][, next])` ( 同全局前置守卫 )

##### 3. 全局后置守卫
`router.afterEach([to][, from][, failure])`

##### 4. 路由独享守卫
`beforeEnter([to][, from][, next])` ( query / params / hash 改变时不会触发 )
```javascript
const routes = [
  {
    path: '/demo1',
    component: Demo1,
    beforeEnter: (to, from) => {
      return false; // 取消导航
    }
  },
  {
    path: '/demo2',
    component: Demo2,
    beforeEnter: [fnc1, fnc2] // 接收一个函数数组
]
```

##### 5. 组件前置路由守卫
`beforeRouteEnter` ( 在vm实例创建之前调用， 可通过next传入一个回调函数，默认参数是vm实例 )
```javascript
const ComponentOptions = {
  beforeRouteEnter(to, from, next) {
    next(vm => {});
  }
}
```

##### 6. 组件路由更新守卫
`beforeRouteUpdate` ( 重用组件时调用 )
```javascript
const ComponentOptions = {
  beforeRouteUpdate(to, from) {
    // ...
  }
}
```

##### 7. 组件后置路由守卫
`beforeRouteLeave([to][, from][, next])` ( 重用组件时调用 )
```javascript
const ComponentOptions = {
  beforeRouteLeave(to, from) {
    // ...

    /* 取消导航 */
    return false
  }
}
```

##### 8. 调用顺序
`进入组件`
    beforeEach 
->  beforeRouterEnter
->  beforeResolve
->  afterEach
->  beforeCreate ( 组件lifecycle )

`离开组件`
    beforeRouterLeave
->  beforeEach
->  beforeResolve
->  afterEach
->  beforeDestory ( 组件lifecycle )
