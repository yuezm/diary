```
class Promise {
  constructor(execute) {
    this.status = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledCallback = [];
    this.onRejectedCallback = [];

    try {
      execute(this.resolvePromise.bind(this), this.rejectPromise.bind(this));
    } catch (error) {
      this.rejectPromise(error);
    }
  }

  /** resolve,Promise execute 第一参数
   *  pending => fulfilled 时需要判断 value是否是promise,及thenable对象，如果是，则状态需要与它保持一致，否则进行状态扭转
   * !!! 请注意，这里 resolveFulfilledValue 第三个参数为 resolve,而非resolePromise,因为进入 resolveFulfilledValue 函数执行完成后，
   * 不会在存在 thenable对象(即使存在then，then的值也非函数,不需要考虑)不需要再次判断，否则如果then为属性时，会陷入无限循环判断
   *
   * !!! 为什么不在 resolvePromise 进行 then获取和判断，因为测试用例对 x.then 获取次数进行了限制，获取1次为 Function，大于1次为null（坑的很）
   * @param {*} value
   */
  resolvePromise(value) {
    if (this.status === 'pending') {
      if (value !== null && typeof value === 'object' || typeof value === 'function') {
        resolveFulfilledValue(this, value, this.resolve.bind(this), this.rejectPromise.bind(this));
      } else {
        // 如果不是promise，thenable对象，则resolve
        this.resolve(value);
      }
    }
  }

  // 从 resolvePromise 抽离出来，单独称为修改 fulfilled 状态方法
  resolve(value) {
    this.status = 'fulfilled';
    this.value = value;

    setTimeout(() => {
      this.onFulfilledCallback.forEach(fn => fn(value));
    }, 0);
  }

  /**
   * reject,Promise execute 第二参数
   * @param {*} reason
   */
  rejectPromise(reason) {
    if (this.status === 'pending') {
      this.status = 'rejected';
      this.reason = reason;

      setTimeout(() => {
        this.onRejectedCallback.forEach(fn => fn(reason));
      }, 0);
    }
  }


  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : function (value) {
      return value
    };
    onRejected = typeof onRejected === 'function' ? onRejected : function (reason) {
      throw reason
    };

    const _this = this,
      status = _this.status;
    let promise2;

    if (status === 'fulfilled') {
      promise2 = new Promise((resolve, reject) => {
        setTimeout(() => {
          try {
            // 处理 onFulfilled 返回值, 判断返回值是否我promise,thenable对象
            resolveFulfilledValue(promise2, onFulfilled(_this.value), resolve, reject);
          } catch (error) {
            reject(error);
          }
        }, 0);
      });

    } else if (status === 'rejected') {
      promise2 = new Promise((resolve, reject) => {
        setTimeout(() => {
          try {
            // 处理 onRejected 返回值
            resolveFulfilledValue(promise2, onRejected(_this.reason), resolve, reject);
          } catch (error) {
            reject(error);
          }
        }, 0);
      });
    } else {
      promise2 = new Promise((resolve, reject) => {
        _this.onFulfilledCallback.push(value => {
          try {
            resolveFulfilledValue(promise2, onFulfilled(value), resolve, reject);
          } catch (error) {
            reject(error);
          }
        });

        _this.onRejectedCallback.push(reason => {
          try {
            resolveFulfilledValue(promise2, onRejected(reason), resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      });
    }

    return promise2;
  }
}

function resolveFulfilledValue(promise, x, resolve, reject) {
  if (promise === x) {
    reject(TypeError());
    return;
  }

  const _type = typeof x;

  if ((_type === 'function' || _type === 'object') && x !== null) {
    let called = false; // 保证resolve,reject只执行一次
    try {
      // 务必保证 then 只获取一次
      const then = x.then;
      if (typeof then === 'function') {
        then.call(
          x,
          value => {
            if (called) return;
            called = true;
            resolveFulfilledValue(promise, value, resolve, reject);
          },
          reason => {
            if (called) return;
            called = true;
            reject(reason);
          }
        );
      } else {
        if (called) return;
        called = true;
        resolve(x);
      }
    } catch (error) {
      if (called) return;
      called = true;
      reject(error);
    }
  } else {
    resolve(x);
  }
}

module.exports = Promise;

// 测试用例
Promise.deferred = function () {
  let dfd = {};
  dfd.promise = new Promise(function (resolve, reject) {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd
};
```
