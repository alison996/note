### Axios取消请求以及其原理（v0.26.1）
#### 1. 取消请求
```javascript
const axios = require('axios')

const instance = new axios.Axios({})
// 创建source，通过source.cancel()取消请求
const source = new axios.CancelToken.source()

instance.defaults.timeout = 10000

instance.interceptors.request.use(
  config => {
    config.cancelToken = source.token
    return config
  }
)

// 此处写了两个响应拦截器是测试用，
// 因为我在项目中看到他人
// 定义了一个全局的CancelToken.source。当请求失败
// 报错后执行source()，使config.token.reason.message = undefined
// dispatchRequest时会调用CancelToken.prototype.throwIfRequested方法，
// 直接抛出异常信息config.token.reason，不发送ajax请求
instance.interceptors.response.use(
  ({ data, status, statusText }) => {
    if (status < 200 || status > 399) {
      throw statusText
    }
    return data
  }
)
instance.interceptors.response.use(
  undefined,
  error => {
    // 取消请求
    source.cancel()
    throw error
  }
)
```

#### 2. 取消请求的原理
调用 **CancelToken.source.cancel方法** 时，将 **CancelToken实例** 上的 **promise** 属性的状态改为 **fulfilled**

`CancelToken.source方法`
```javascript
CancelToken.source = function source() {
  var cancel;
  // 用cancel保存executor中的方法c方法，执行cancel()时，会将token.promise的状态变为fulfilled
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};

```

`CancelToken构造函数`
```javascript
function CancelToken(executor) {
  // 此处省略executor须为function类型的校验

  // 用resolvePromise保存Promise中的resolve方法，在执行source.cancel()的时候调用
  var resolvePromise;
  this.promise = new Promise((resolve) => {
    resolvePromise = resolve;
  });

  var token = this;

  // promise的状态变为fulfilled时，遍历_listeners并执行里面的回调
  this.promise.then(function(cancel) {
    if (!token._listeners) return;
    var i;
    var l = token._listeners.length;
    for (i = 0; i < l; i++) {
      token._listeners[i](cancel);
    }
    token._listeners = null;
  });

  
  // 重写promise.then方法
  // 若已取消请求，会在token.subscribe方法中执行resolve(this.reason)
  this.promise.then = function(onfulfilled) {
    var _resolve;
    // eslint-disable-next-line func-names
    var promise = new Promise(function(resolve) {
      token.subscribe(resolve);
      _resolve = resolve;
    }).then(onfulfilled);

    // 取消订阅
    promise.cancel = function reject() {
      token.unsubscribe(_resolve);
    };

    return promise;
  };

  executor(function cancel(message) {
    if (token.reason) {
      // Cancellation has already been requested
      return;
    }

    token.reason = new CanceledError(message);
    // 改变token.promise的状态为fulfilled
    resolvePromise(token.reason);
  });
}
```

`CancelToken.prototype.subscribe方法`
```javascript
/**
 *  利用发布订阅者模式，收集回调（执行xhr.abort方法）
 */
CancelToken.prototype.subscribe = function subscribe(listener) {
  // 如果有reason，说明请求已取消，立即执行listener
  if (this.reason) {
    listener(this.reason);
    return;
  }

  if (this._listeners) {
    this._listeners.push(listener);
  } else {
    this._listeners = [listener];
  }
};
```

明白以上逻辑后，剩下要做的就是订阅 **xhr.abort方法** 。