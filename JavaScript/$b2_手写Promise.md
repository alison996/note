#### 手写Promise
##### 1. 个人理解
感觉`手写Promise`，实际上是对`发布订阅模式`及`闭包`的一种应用。

##### 2. 话不多说，直接撸
按照文档注释，与原生一一对比。场景可能没有覆盖全面，大致理解手写Promise的思想、原理即可。另`Promise.allSettle`与`Promise.any`方法没有写，原因是用的比较少...不过其原理大同小异，hah...
```javascript
  const CachePremitivePromise = Promise;  // 在此保存一下原生Promise的引用， 方便后面与手写的放一起比较

  (function() {

    /**
      * 定义自定义promise属性, 属性可写但不可枚举
      */
    function defineNotEnumrableProperty(obj, key, value) {
      Object.defineProperty(obj, key, {
        value,
        enumerable: false,
        writable: true
      })
    }

    // 保存每个promise实例的回调,须事先调用generateCacheCbksZone生成
    const cacheCbksMap = new Map();

    /**
      * 生成cache实例回调的对象
      * 在new阶段调用
      */
    function generateCacheCbksZone(inst) {
      if (!cacheCbksMap.has(inst)) {
        cacheCbksMap.set(inst, {
          resolvedCbks: [],
          rejectedCbks: []
        })
      }
    }

    /**
      * 销毁cache实例回调的对象
      * 在rejected或resolved回调执行完毕后调用
      */
    function destoryCacheCbksZone(inst) {
      if (cacheCbksMap.has(inst)) {
        cacheCbksMap.delete(inst);
      }
    }

    /**
      * 定义STATE常量
      */
    const STATE = {
      /* promise的状态 */
      PENDING:  'pending',
      RESOLVED: 'resolved',
      REJECTED: 'rejected'
    };

    function Promise(excutor) {
      /* 为了尽量还原Promise，不在此处定义保存回调的数组，而是保存到cacheCbksMap中，key为当前Promise的实例，即this */
      // this.resolvedCbks = []; // 保存成功回调
      // this.rejectedCbks = []; // 保存失败回调

      generateCacheCbksZone(this);
      defineNotEnumrableProperty(this, '[[PromiseState]]', STATE.PENDING);
      defineNotEnumrableProperty(this, '[[PromiseResult]]', undefined);

      function resolve(result) {
        if (this['[[PromiseState]]'] === STATE.PENDING) {
          // 状态只允许修改一次
          this['[[PromiseState]]'] = STATE.RESOLVED;
          this['[[PromiseResult]]'] = result;

          // setTimeout(() => this.resolvedCbks.forEach(fn => result = fn(result)))
          setTimeout(() => {
            cacheCbksMap.get(this).resolvedCbks.forEach(fn => result = fn(result))

            // 销毁保存回调的数组
            destoryCacheCbksZone(this);
          })
        }
      }

      function reject(reason) {
        if (this['[[PromiseState]]'] === STATE.PENDING) {
          // 状态只允许修改一次
          this['[[PromiseState]]'] = STATE.REJECTED;
          this['[[PromiseResult]]'] = reason;
          setTimeout(() => {
            const rejectedCbks = cacheCbksMap.get(this).rejectedCbks;

            // if (!this.rejectedCbks.length) {
            if (!rejectedCbks.length) {
              // 若不处理失败结果，则抛出错误
              throw new Error(reason);
            }

            // this.rejectedCbks.forEach(fn => fn(reason))
            rejectedCbks.forEach(fn => {
              fn(reason);
            })

            // 销毁保存回调的数组
            destoryCacheCbksZone(this);
          })
        }
      }

      try {
        excutor(resolve.bind(this), reject.bind(this)); // 默认传 resolve reject 方法
      } catch(e) {
        reject.call(this, e);
      }
    }

    /**
      * Promise的resolve方法，返回一个状态为resolved的对象
      */
    Promise.resolve = function _resolve(result) {
      return new Promise(resolve => resolve(result))
    }

    /**
      * Promise的reject方法，返回一个状态为rejected的对象
      */
    Promise.reject = function _reject(result) {
      return new Promise((resolve, reject) => reject(result))
    }

    /**
      * Promise的all方法，传入promise实例的集合
      */
    Promise.all = function _all(pArray) {
      return new Promise((resolve, reject) => {
        const resArray = [];        // 按顺序保存res
        let completedCount = 0;     // 记录resolved的promise数量

        pArray.forEach((p, i) => {
          p.then(
            result => {
              resArray[i] = result;
              completedCount += 1;  // 完成数量+1

              if (resArray.length === completedCount) {

                /**
                  * completedCount等于resArray的长度时
                  * 说明所有promise已为resolved状态
                  * 将返回promise实例的状态改为resolved，result结果为resArray
                  */
                resolve(resArray);
              }
            },
            reason => reject(reason)  // 只要有一个异常，则将返回的promise实例状态改为rejected，失败原因为异常信息
          )
        })
      })
    }

    /**
      * Promise的race方法，传入promise实例的集合
      */
    Promise.race = function _race(pArray) {
      return new Promise((resolve, reject) => {
        // 一旦有结果立马根据结果类型resolve或reject
        pArray.forEach((p, i) => p.then(result => resolve(result), reason => reject(reason)))
      })
    }

    const CachePromise = Promise;
    {
      function Promise() {

        defineNotEnumrableProperty(this, 'then', function _then() {
          const [onResolved, onRejected] = arguments;

          return new CachePromise((resolve, reject) => {
            const { resolvedCbks, rejectedCbks } = cacheCbksMap.get(this);
            if (onResolved) {

              // this.resolvedCbks.push(onResolved, resolve);
              resolvedCbks.push(onResolved, resolve);
            }

            if (onRejected) {
              // 有onRejected，则在当前promise实例中处理失败结果

              // this.rejectedCbks.push(onRejected, () => resolve());
              rejectedCbks.push(onRejected, () => resolve());
            } else {
              // 无onRejected，则返回一个rejected的Promise实例，在返回的实例上处理失败结果，若不处理则报错

              // this.rejectedCbks.push(reject);
              rejectedCbks.push(reject);
            }
          })
        })

        defineNotEnumrableProperty(this, 'catch', function _catch() {
          const onCatch = arguments[0];
          const self = this;

          return new CachePromise((resolve, reject) => {
            // if (onCatch) {
            if (onCatch && cacheCbksMap.has(self)) {
              // 若有onCatch，则放到rejectedCbks中，处理失败结果后返回一个resolved的Promise实例

              // this.rejectedCbks.push(onCatch, () => resolve());
              cacheCbksMap.get(self).rejectedCbks.push(onCatch, () => resolve());
            }
          })
        })

        defineNotEnumrableProperty(this, 'finally', function _finally() {
          const onFinally = arguments[0];
          const self = this;

          return new CachePromise((resolve, reject) => {
            // if (onFinally) {
            if (onFinally && cacheCbksMap.has(self)) {
              // 若有onFinally，则push到resolvedCbks，rejectedCbks中，最后执行

              // this.resolvedCbks.push(() => onFinally(), () => resolve());
              // this.rejectedCbks.push(() => onFinally(), reject);

              const { resolvedCbks, rejectedCbks } = cacheCbksMap.get(self);
              resolvedCbks.push(() => onFinally(), () => resolve());
              rejectedCbks.push(() => onFinally(), reject);
            }
          })
        })
      }

      // 此处这样写是为了让改变自定义Promise的隐式原型
      CachePromise.prototype = new Promise();
      // 改变一下constructor，尽量对齐原生的Promise
      defineNotEnumrableProperty(CachePromise.prototype, 'constructor', Promise);
    }
    // 还原自定义的Promise
    Promise = CachePromise;

    /**
      * @description Promise.resolve 测试
      */
    // {
    //   const p = Promise.resolve('自定义success');
    //   console.log('自定义resolve', p);
    // }

    // console.log('------------------------------');

    // {
    //   const p = CachePremitivePromise.resolve('原生success');
    //   console.log('原生resolve', p);
    // }

    /**
      * @description Promise.reject 测试
      */
    // {
    //   const p = Promise.reject('自定义rejected')
    //   .then(
    //     res => console.log('自定义res-1111', res),
    //     // err => console.log('自定义err-1111', err)
    //   )
    //   .then(
    //     res => console.log('自定义res-2222', res),
    //     // err => console.log('自定义err-2222', err)
    //   );
    //   console.log('自定义reject', p);
    // }

    // console.log('------------------------------');

    // {
    //   const p = CachePremitivePromise.reject('原生rejected')
    //   .then(
    //     res => console.log('原生res-1111', res),
    //     // err => console.log('原生err-1111', err)
    //   )
    //   .then(
    //     res => console.log('原生res-2222', res),
    //     // err => console.log('原生err-2222', err)
    //   );
    //   console.log('原生reject', p);
    // }

    /**
      * @description Promise.all 测试
      */
    // {
    //   const p1 = new Promise((resolve, reject) => {
    //     // setTimeout(resolve, 3000, '自定义 resolve p1');
    //     setTimeout(reject, 3000, '自定义 reject p1');
    //   });
    //   const p2 = new Promise((resolve, reject) => {
    //     setTimeout(resolve, 2000, '自定义 p2');
    //   });
    //   const p3 = new Promise((resolve, reject) => {
    //     setTimeout(resolve, 1000, '自定义 p3');
    //   });

    //   const pa = Promise.all([p1, p2, p3])
    //   .then(
    //     res => console.log('自定义 all resolved', res),
    //     err => console.log('自定义 all rejected', err),
    //   )
    //   // .catch(
    //   //   err => console.log('自定义 all catch', err)
    //   // )

    //   console.log('自定义all', pa);
    // }

    // console.log('------------------------------');

    // {
    //   const p1 = new CachePremitivePromise((resolve, reject) => {
    //     // setTimeout(resolve, 3000, '原生 resolve p1');
    //     setTimeout(reject, 3000, '原生 reject p1');
    //   });
    //   const p2 = new CachePremitivePromise((resolve, reject) => {
    //     setTimeout(resolve, 2000, '原生 p2');
    //   });
    //   const p3 = new CachePremitivePromise((resolve, reject) => {
    //     setTimeout(resolve, 1000, '原生 p3');
    //   });

    //   const pa = CachePremitivePromise.all([p1, p2, p3])
    //   .then(
    //     res => console.log('原生 all resolved', res),
    //     // err => console.log('原生 all rejected', err),
    //   )
    //   // .catch(
    //   //   err => console.log('原生 all catch', err)
    //   // )

    //   console.log('原生 all', pa);
    // }

    /**
      * @description Promise.race 测试
      */
    // {
    //   const p1 = new Promise(resolve => setTimeout(resolve, 3000, '自定义 p1'));
    //   const p2 = new Promise(resolve => setTimeout(resolve, 2000, '自定义 p2'));
    //   const p3 = new Promise((resolve, reject) => {
    //     setTimeout(resolve, 1000, '自定义 p3');
    //     // setTimeout(reject, 1000, '自定义 p3');
    //   });

    //   const pa = Promise.race([p1, p2, p3])
    //   // .then(
    //   //   res => console.log('自定义 race resolved', res),
    //   //   err => console.log('自定义 race rejected', err)
    //   // )
    //   // .catch(
    //   //   err => console.log('自定义 all catch', err)
    //   // )

    //   console.log('自定义 race', pa);
    // }

    // console.log('------------------------------');

    // {
    //   const p1 = new CachePremitivePromise(resolve => setTimeout(resolve, 3000, '原生 p1'));
    //   const p2 = new CachePremitivePromise(resolve => setTimeout(resolve, 2000, '原生 p2'));
    //   const p3 = new CachePremitivePromise((resolve, reject) => {
    //     setTimeout(resolve, 1000, '原生 p3');
    //     // setTimeout(reject, 1000, '原生 p3');
    //   });

    //   const pa = CachePremitivePromise.race([p1, p2, p3])
    //   // .then(
    //   //   res => console.log('原生 race resolved', res),
    //   //   err => console.log('原生 race rejected', err)
    //   // )
    //   // .catch(
    //   //   err => console.log('原生 all catch', err)
    //   // )

    //   console.log('原生 race', pa);
    // }

    /**
      * @description Promise.prototype.then 测试
      */
    // {
    //   const p = Promise.resolve('自定义')
    //   // .then(
    //   //   res => console.log('自定义 then-111', res)
    //   // )
    //   // .then(
    //   //   res => console.log('自定义 then-222', res)
    //   // );
    //   console.log('自定义 then', p);
    // }

    // console.log('------------------------------');

    // {
    //   const p = CachePremitivePromise.resolve('原生')
    //   // .then(
    //   //   res => console.log('原生 then-111', res)
    //   // )
    //   // .then(
    //   //   res => console.log('原生 then-222', res)
    //   // );
    //   console.log('原生 then', p);
    // }


    /**
      * @description Promise.prototype.catch 测试
      */
    // {
    //   // const p = Promise.reject('自定义 rejected')
    //   const p = new Promise(() => {
    //     // setTimeout(() => {
    //       throw new Error('自定义错误');
    //     // }, 3000);
    //   })
    //   .then(
    //     res => console.log('自定义 res-1111', res),
    //     // err => console.log('自定义 err-1111', err)
    //   )
    //   .catch(
    //     err => console.log('自定义catch', err),
    //   );
    //   console.log('自定义 catch', p);
    // }

    // console.log('------------------------------');

    // {
    //   // const p = CachePremitivePromise.reject('原生 rejected')
    //   const p = new CachePremitivePromise(() => {
    //     // setTimeout(() => {
    //       throw new Error('原生错误');
    //     // }, 3000);
    //   })
    //   .then(
    //     res => console.log('原生 res-1111', res),
    //     // err => console.log('原生 err-1111', err)
    //   )
    //   .catch(
    //     err => console.log('原生catch', err),
    //   );
    //   console.log('原生 catch', p);
    // }


    /**
      * @description Promise.prototype.finally 测试
      */
    {
      // const p = Promise.reject('自定义 rejected')
      const p = Promise.resolve('自定义 resolved')
      .then(
        res => console.log('自定义 res-1111', res),
        err => console.log('自定义 err-1111', err)
      )
      .finally(
        /* finally的回调没有参数，此处是我为了测试故意这样写的 */
        err => console.log('自定义 finally', err),
      );
      console.log('自定义 finally', p);
    }

    {
      // const p = CachePremitivePromise.reject('原生 rejected')
      const p = Promise.resolve('原生 resolved')
      .then(
        res => console.log('原生 res-1111', res),
        err => console.log('原生 err-1111', err)
      )
      .finally(
        /* finally的回调没有参数，此处是我为了测试故意这样写的 */
        err => console.log('原生 finally', err),
      );
      console.log('原生 finally', p);
    }
  })()
```