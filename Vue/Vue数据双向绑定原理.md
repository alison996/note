
#### Vue数据双向绑定原理 ( Vue.version = 2.6.14 )
<font color="red">分析主要代码</font>

##### 1. 监测Object的变化
在`vue.commone.dev.js`文件中，Vue在`defineReactive$$1`方法中监听对象的属性变化。

```javascript
/**
 * defineReactive$$1
 */
function defineReactive$$1 (
  obj,
  key,
  val,监测
  customSetter,
  shallow
) {

  // 创建Dep实例
  const dep = new Dep();

  // 获取属性默认的 getter/setter ，防止重新定义getter/setter 的时候覆盖默认行为
  const descriptor = Object.getOwnPropertyDescriptor(obj, key);
  const getter = descriptor && descriptor.get;
  const setter = descriptor && descriptor.set;

  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key];
  }

  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function() {
      const value = getter ? getter.call(obj) : val;
      if (Dep.target) {

        /**
         * 1. 调用Dep.prototype.depend
         * 2. 在Dep.prototype.depend中调用Watcher.prototype.addDep
         * 3. Watcher.prototype.addDep中将Dep实例保存到Watcher实例的newDepIds、newDeps属性上，再将Watcher实例保存到Dep实例的subs属性上
         */
        dep.depend();
      }
      return value;
    },
    set: function(newVal) {
      const value = getter ? getter.call(obj) : val;

      // 设置新的值
      if (setter) {
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }

      /**
       * 1. 调用Dep.prototype.notify
       * 2. 在Dep.prototype.notify中遍历Dep实例上的subs属性 ( getter中保存的Watcher实例 )
       * 3. 调用Watcher.prototype.update更新视图 ( 具体update中做了什么，下次单独分析 )
       */
      dep.notify(); 
    }
  })
}
```
<font color="green">整体流程：</font>

① 通过`observer`，使用`Object.defineProperty`让`data`的获取、变化能够被监测

② 获取`data`时，触发`getter`， 将`依赖`添加到`watcher`中，将`wacher`添加到`依赖`的`subs ( 订阅？ )` 中

③ 更新`data`时，触发`setter`，通知`依赖`遍历`subs`，并调用`watcher`的`update`方法更新视图 ( 其实最终调用的是`Vue实例`的`_render`方法 )

<font color="orange">疑问：</font>

① `watcher`是在何时创建的？

  在`initComputed`阶段创建`watcher`，并且`watcher的vm属性`指向`Vue实例`，将`watcher`保存到`Vue实例的_watchers`中

<font color="red">不足：</font>
- 向对象中添加或删除属性时，无法监测到变化
- 无法监测数组的变化

##### 2. 监测Array的变化
定义`拦截器`，在调用`修改`数组的方法时，触发拦截器。
```javascript
const arrayProto = Array.prototype;
const arrayMethods = Object.create(arrayProto);

// 能够改变数组自身的7个方法
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
];

// 遍历methodsToPatch并修改arrayMethods上对应名称的方法
methodsToPatch.forEach(method => {
  Object.defineProperty(
    arrayMethods,
    method,
    {
      value: function() {

        /**
         * clone参数
         */
        const args = [];
        let len = arguments.length;
        while (len--) {
          args[len] = arguments[len];
        }

        // 调用原生的方法
        const result = arrayProto[method].apply(this, args);

        /**
         * 调用 push/unshift/splice时若添加/更新 了新元素
         * 则调用observe观察该元素
         */
        const ob = this.__ob__;
        let inserted;
        switch (method) {
          case 'push':
          case 'unshift':
            inserted = args;
            break;
          case 'splice':
            inserted = args.slice(2);
            break;
          default:
            break;
        }
        if (inserted) {
          ob.observeArray(inserted);
        }

        // 通知依赖更新
        ob.dep.notify();

        return result;
      }
    }
  )
})
```
<font color="green">整体流程：</font>

① 给`Array.prototype`上能`改变数组本身的方法`添加`拦截器`

② 在`Observer的构造器中`将`Array实例的__proto__`指向`arrayMethods`

③ 调用这些方法时，通知依赖更新

<font color="red">不足：</font>
- 通过`索引`修改数组时，无法监测到数组的变化

##### 3. Vue对以上不足的解决方案
增加了两个全局API：`Vue.set`和`Vue.delete`。