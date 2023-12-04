# 3、Promise规范及应用

## Promise的术语

1. `promise`是一个有then方法的对象或者是函数，行为遵循本规范
2. `thenable`是一个有then方法的对象或者是函数
3. `value` 是promise状态成功时的值，也就是`resolve`的參数，表示结果的数据
4. `reason`是promise状态失败时的值，也就是r`eject`的参数，表示拒绝的原因
5. `exception`是一个使用throw抛出的异常值



## Promise的使用

### 异步逻辑

![image-20231025011752448](http://img.roydust.top/img/image-20231025011752448.png)

###  为什么会有微任务？

调用栈并发量大的时候，微任务可以解决异步时机不可控的问题。



### 异步

- 事件回调
- Ajax 请求
- Node API
- setTimeout 等等



#### callback

```js
function foo(fn) {
  setTimeout(() => {
    console.log("foo");
    // 假如，把这个foo字符串看成是一个复杂逻辑的计算结果
    fn();
  }, 5000);
}

foo((res) => {
  console.log("res", res);
  console.log("bar");
});
```

##### 回调地狱

![image-20231025013143010](http://img.roydust.top/img/image-20231025013143010.png)



#### Promise

![image-20231025013859609](http://img.roydust.top/img/image-20231025013859609.png)

let SomeThing， 我返回了一个promise；

1. Promise 是一个构造函数
2. Promise 接收一个函数，这个函数的参数，是（resolve, reject），也要求是函数。
3. Promise 返回的对象，包含一个 then 函数，then 函数接收两个參数，这两个参数，一般也是函数。
4. 我们再使用 new 关键字调用 Promise 构造函数时，在结束时
   - 如果正确执行，调用 resolve 方法，将结果放在 resolve 的参数中执行，这个结果可以在后面then 中的第一个函数参数（onfulfilled） 中拿到：
   - 如果错误执行，调用 reject 方法，将错误信息放在 reject 的参数中执行，这个结果可以在后面then 中的第二个函数参数（onrejected）中拿到：



## Promise的实现

### 初探Promise

#### Promise的状态

1. `pending`
   - 初始的状态，可改变
   - 一个promise在`resolve`/`reject`前都属于这种状态
   - 我们可以通过使用`resolve`方法或者`reject`方法，让这个promise，变成`fulfilled`/`rejected`状态
2. `fulfilled`
   - 不可变状态
   - 在`resolve`之后，变成这个状态，拥有一个`value`
3. `rejected`
   - 不可变状态
   - 在`reject`之后，变成这个状态，拥有一个`reason`



#### then 函数

1. 参数：
   - `onFulfilled`必须是函数类型，如果不是应该被忽略
   - `onRejected`必须是函数类型，如果不是应该被忽略
2. onFulfilled / onRejected 的特征
   - 在 promise 变成`fulfilled` / `rejected`状态的时候，应该调用 `onFulfilled` / `onRejected`
   - 在 promise  变成 `fulfilled` / `rejected`状态之前，不应该被调用；
   - 只能被调用一次。



### 完善Pormise

#### 第一版

**要有改变状态和返回不同值的方法**

```js
function L0Promise(executor) {
  this.status = "pending";
  this.value = undefined;
  this.reason = undefined;
  // 我们可以通过使用`resolve`方法或者`reject`方法，让这个promise，变成`fulfilled`/`rejected`状态
  let resolve = (value) => {
    if (this.status === "pending") {
      this.status = "fulfilled";
      this.value = value;
    }
  };
  let reject = (reason) => {
    if (this.status === "pending") {
      this.status = "rejected";
      this.reason = reason;
    }
  };
  executor(resolve, reject);
}

L0Promise.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled =
    typeof onFulfilled === "function" ? onFulfilled : (value) => value;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (reason) => {
          throw reason;
        };
  if (this.status === "fulfilled") {
    onFulfilled(this.value);
  }
  if (this.status === "rejected") {
    onRejected(this.reason);
  }
};

let promise = new L0Promise((resolve, reject) => {
  resolve("data");
  // 异步无效
  // setTimeout(() => {
  //   console.log("data");
  // }, 5000);
});

promise.then((data) => {
  console.log(data);
});
```



问题在于，当我 resolve 的时候，onfulfilled 函数，已经执行过了，
所以，我们需要在一个合适的时间去执行 onfulfilled.
换句话说，我们需要在一个合适的时间，去通知 onfulfilled 执行，
--发布订阅。

#### 第二版

**onfulfilled 和 onrejected 应该是微任务**，我们暂时用 setTimeout 来代替。

```js
function L1Promise(executor) {
  this.status = "pending";
  this.value = undefined;
  this.reason = undefined;
  this.onFulfilledArray = [];
  this.onRejectedArray = [];
  // 为什么一定要用数组
  // then方法可以多次被调用
  // promise的状态变成`fulfilled`/`rejected`后，所有的`onFulfilled`/`onRejected`
  // 都按照 then 的顺序来执行，也就是按照注册顺序执行

  // 我们可以通过使用`resolve`方法或者`reject`方法，让这个promise，变成`fulfilled`/`rejected`状态
  let resolve = (value) => {
    setTimeout(() => {
      if (this.status === "pending") {
        this.status = "fulfilled";
        this.value = value;
        this.onFulfilledArray.forEach((fn) => fn(value));
      }
    });
  };
  let reject = (reason) => {
    setTimeout(() => {
      if (this.status === "pending") {
        this.status = "rejected";
        this.reason = reason;
        this.onRejectedArray.forEach((fn) => fn(reason));
      }
    });
  };
  executor(resolve, reject);
}

L1Promise.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled =
    typeof onFulfilled === "function" ? onFulfilled : (value) => value;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (reason) => {
          throw reason;
        };
  if (this.status === "fulfilled") {
    onFulfilled(this.value);
  }
  if (this.status === "rejected") {
    onRejected(this.reason);
  }
  if (this.status === "pending") {
    this.onFulfilledArray.push(onFulfilled);
    this.onRejectedArray.push(onRejected);
  }
};

let promise = new L1Promise((resolve, reject) => {
  // resolve("data");
  // 异步无效
  setTimeout(() => {
    // console.log("data");
    resolve("data");
  }, 500);
});

promise.then((data) => {
  console.log(data);
});

```

[问题：为什么一定要用数组]

- then方法可以多次被调用
- promise的状态变成`fulfilled`/`rejected`后，所有的`onFulfilled`/`onRejected`都按照 then 的顺序来执行，也就是按照注册顺序执行

**但是这一版不能链式调用**



#### 第三版

**then应该返回一个promise**

- onFulfilled / onRejected 执行的结果为x，调用 resolvePromise；
- 如果 onFulfilled / onRejected 执行时拋出异常，promise2 需要被 reject；
- 如果 onFulfilled / onRejected 不是一个函数，promise2 以 promise1 的 value/reason 触发rejected/fulfilled

```js
function L2Promise(executor) {
  this.status = "pending";
  this.value = undefined;
  this.reason = undefined;
  this.onFulfilledArray = [];
  this.onRejectedArray = [];

  let resolve = (value) => {
    setTimeout(() => {
      if (this.status === "pending") {
        this.status = "fulfilled";
        this.value = value;
        this.onFulfilledArray.forEach((fn) => fn(value));
      }
    });
  };
  let reject = (reason) => {
    setTimeout(() => {
      if (this.status === "pending") {
        this.status = "rejected";
        this.reason = reason;
        this.onRejectedArray.forEach((fn) => fn(reason));
      }
    });
  };
  executor(resolve, reject);
}

L2Promise.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled =
    typeof onFulfilled === "function" ? onFulfilled : (value) => value;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (reason) => {
          throw reason;
        };

  let promise2; //作为then函数的返回值
  if (this.status === "fulfilled") {
    return new L2Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          let result = onfulfilled(this.value);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
    });
  }
  if (this.status === "rejected") {
    setTimeout(() => {
      try {
        let result = onrejected(this.value);
        resolve(result);
      } catch (error) {
        reject(error);
      }
    });
  }
  if (this.status === "pending") {
    return (promise2 = new L2Promise((resolve, reject) => {
      this.onFulfilledArray.push(() => {
        try {
          let result = onFulfilled(this.value);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      this.onRejectedArray.push(() => {
        try {
          let result = onRejected(this.reason);
          resolve(result);
        } catch (error) { 
          reject(error);
        }
      });
    }));
  }
};

let promise = new L2Promise((resolve, reject) => {
  // resolve("data");
  // 异步无效
  setTimeout(() => {
    // console.log("data");
    resolve("data");
  }, 500);
});

promise.then((data) => {
  console.log(data);
});
```

但是传入的promise的参数不能为另外一个promise

#### 第四版

将resolve改为resolvePromise

**如何实现resolvePromise**

- 如果 promise2 和 x 相等，那么 reject error;
- 如果 x 是一个 promise
  - 如果 x 是一个 pending 状态，那么 promise 必须要在 pending, 直到 x 变成 fulfilled or rejected
  - 如果 x 被 fulfilled， fulfill promise with the same value
  - 如果 x 被 rejected， reject promise with the same reason
- 如果 x 是一个 object 或者 function
  - let thenable = x.then
  - 如果 x.then 这一步出错，那么 reject promise with e as the reason
  - 如果 then 是一个函数，then.call(x, resolvePromiseFn, rejectPromise) resolvePromiseFn 的 入参是 y, 执行 resolvePromise(promise2, y, resolve, reject); rejectPromise 的 入参是 r, reject promise with r. 如果 resolvePromise 和 rejectPromise 都调用了，那么第一个调用优先，后面的调用忽略。 如果调用then抛出异常e 如果 resolvePromise 或 rejectPromise 已经被调用，那么忽略 则，reject promise with e as the reason 如果 then 不是一个function. fulfill promise with x.

```js
function L3Promise(executor) {
  this.status = "pending";
  this.value = undefined;
  this.reason = undefined;
  this.onFulfilledArray = [];
  this.onRejectedArray = [];

  let resolve = (value) => {
    setTimeout(() => {
      if (this.status === "pending") {
        this.status = "fulfilled";
        this.value = value;
        this.onFulfilledArray.forEach((fn) => fn(value));
      }
    });
  };
  let reject = (reason) => {
    setTimeout(() => {
      if (this.status === "pending") {
        this.status = "rejected";
        this.reason = reason;
        this.onRejectedArray.forEach((fn) => fn(reason));
      }
    });
  };
  try {
    executor(resolve, reject);
  } catch (error) {
    reject(error);
  }
}

// resolvePromise 就是一个纯规范理论

const resolvePromise = (promise2, result, resolve, reject) => {
  // 死循环，结果和then的返回promise相等
  if (result === promise2) {
    reject(new TypeError("error due to circular reference"));
  }

  // 是否已经执行过 onfulfilled 或者 onrejected
  let consumed = false;
  let thenable;

  if (result instanceof LPromise) {
    if (result.status === "pending") {
      result.then(function (data) {
        //promsie状态改变后，resolvePromise返回结果
        resolvePromise(promise2, data, resolve, reject);
      }, reject);
    } else {
      //promise状态已改变
      result.then(resolve, reject);
    }
    return;
  }

  let isComplexResult = (target) =>
    (typeof target === "function" || typeof target === "object") &&
    target !== null;

  // 如果返回的是疑似 Promise 类型
  if (isComplexResult(result)) {
    try {
      thenable = result.then;
      // 如果返回的是 Promise 类型，具有 then 方法
      if (typeof thenable === "function") {
        thenable.call(
          result,
          function (data) {
            //保证执行一次 避免死循环
            if (consumed) {
              return;
            }
            consumed = true;

            return resolvePromise(promise2, data, resolve, reject);
          },
          function (error) {
            //保证执行一次
            if (consumed) {
              return;
            }
            consumed = true;

            return reject(error);
          }
        );
      } else {
        resolve(result);
      }
    } catch (e) {
      if (consumed) {
        return;
      }
      consumed = true;
      return reject(e);
    }
  } else {
    //基本数据类型
    resolve(result);
  }
};

L3Promise.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled =
    typeof onFulfilled === "function" ? onFulfilled : (value) => value;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (reason) => {
          throw reason;
        };

  let promise2; //作为then函数的返回值
  if (this.status === "fulfilled") {
    return new L3Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          let result = onfulfilled(this.value);
          resolvePromise(promise2, result, resolve, reject);
        } catch (error) {
          reject(error);
        }
      });
    });
  }
  if (this.status === "rejected") {
    setTimeout(() => {
      try {
        let result = onrejected(this.value);
        resolvePromise(promise2, result, resolve, reject);
      } catch (error) {
        reject(error);
      }
    });
  }
  if (this.status === "pending") {
    return (promise2 = new L3Promise((resolve, reject) => {
      this.onFulfilledArray.push(() => {
        try {
          let result = onFulfilled(this.value);
          resolvePromise(promise2, result, resolve, reject);
        } catch (error) {
          reject(error);
        }
      });
      this.onRejectedArray.push(() => {
        try {
          let result = onRejected(this.reason);
          resolvePromise(promise2, result, resolve, reject);
        } catch (error) {
          reject(error);
        }
      });
    }));
  }
};

let promise = new L3Promise((resolve, reject) => {
  // resolve("data");
  // 异步无效
  setTimeout(() => {
    // console.log("data");
    resolve("data");
  }, 500);
});

promise.then((data) => {
  console.log(data);
});

```



### Promise问题

```js

// 1. 如何并发执行 promise？

const promiseArrGenerator = (num) =>
  new Array(num).fill(0).map(
    (item, index) => () =>
      new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve(index);
        });
      })
  );

const promiseArr = promiseArrGenerator(100);

Promise.all(promiseArr.map((item) => item())).then((res) => console.log(res));

// 2. promiseChain ， 顺序执行这些 promise
const promiseChain = (proArr) => {
  proArr
    .reduce(
      (proChain, pro) =>
        proChain.then((res) => {
          console.log(res);
          return pro(res);
        }),
      Promise.resolve(-1)
    )
    .then((res) => console.log(res));
};

promiseChain(promiseArr);

// 3. pipe去保证并发量的处理

const promisePipe = (proArr, concurrent) => {
  if (concurrent > proArr.length) {
    return Promise.then(proArr.map((fn) => fn())).then((resArr) =>
      console.log(resArr)
    );
  }

  let _arr = [...proArr];

  for (let i = 0; i < concurrent; i++) {
    let fn = _arr.shift();
    run(fn);
  }

  function run(fn) {
    fn().then((res) => {
      console.log(res);
      if (_arr.length) run(_arr.shift());
    });
  }
};

promisePipe(promiseArr, 10);

```

