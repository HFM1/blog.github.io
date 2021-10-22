### Promise 期约

#### 出现的原因（需求）
  javascript 以往的异步编程模式实现不理想，只支持自定义回调函数来表明异步操作完成。串联多个异步操作是一个常见的操作，通常需要深度嵌套的回调函数（回调地狱）来解决。

  ```javascript
    function double(value, success, fail) {
      setTimeout(() => {
        try {
          if(typeof value !== 'number') {
            throw 'Must provide number as value argument'
          }
          success(2 * value)
        } catch(e) {
          fail(e)
        }
      })
    }
    const success = (value) => {
      console.log(value)
    }
    const fail = (value) => {
      console.log(value)
    }
    double(2, success, fail)
  ```


#### 定义
  期约是对尚不存在结果的一个替身，描述的是一个异步程序执行的机制。

#### 发展过程
  - 早期的期约(jQuery 的Deferred API的形式) --> CommonJS（Promises/A规范）--> ECMAScript 6(Promises/A+)
  - 所有现代浏览器都支持ES6期约, 很多其他浏览器API（如fetch()和Battery Status API）也以期约为基础。



#### jQuery Deferred API(早期期约机制)
  > ajax操作的链式写法
  ```javascript
  $.ajax("test.html")
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
  ```
  > 类似 promise.all
  ```javascript
  $.when($.ajax("test1.html"), $.ajax("test2.html"))
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
  ```
  > 类似 new Promise
  ```javascript
  var wait = function(dtd){
　　　　var dtd = $.Deferred(); //在函数内部，新建一个Deferred对象
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　　　};
　　　　setTimeout(tasks,5000);

　　　　return dtd.promise(); // 返回promise对象
　　};
　　$.when(wait())
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
  ```


#### ECMAScript 6(Promises/A+) 基础
  1. 基础用法
  ```javascript
  let p = new Promise((resolve, reject) => {
    const value = Math.random()
    if(value - 0.5 > 0.001) {
      resolve(value)
    } else {
      reject(value)
    }
  });
  p.then(value => {
    console.log('value', value)
  }).catch(err => {
    console.log('err', err)
  })
  ```

  2. 期约状态机
    期约是一个有状态的对象，有三种状态： 待定(pending)、兑现(fulfilled、resolved)、拒绝(rejected)
    待定(pending)是初始状态，在待定状态下，期约可以落定为兑现(fulfilled、resolved)状态或者拒绝(rejected)状态。 无论落定为哪种状态都是不可逆的。只要从待定转换为兑现或拒绝，期约的状态就不再改变。期约的状态是私有的，不能直接通过JavaScript检测到。
    ![avatar](./img/promise.png)
  
  3. Promise.resolve()
    期约并非一开始就必须处于待定状态，然后通过执行器函数才能转换为落定状态。通过调用Promise.resolve()静态方法，可以实例化一个解决的期约。
  ```javascript
    let p1 = new Promise((resolve, reject) => resolve(1))
    let p2 = Promise.resolve(2)
  ```
  4. Promise.reject()
    与Promise.resolve()类似，Promise.reject()会实例化一个拒绝的期约并抛出一个异步错误（这个错误不能通过try/catch捕获，而只能通过拒绝处理程序捕获）。
  ```javascript
    let p1 = new Promise((resolve, reject) => reject(1))
    let p2 = Promise.reject(2)
  ```
  5. Promise 的错误捕获(Promise.prototype.catch)
  ```javascript
    // 抛出并捕获了错误
    try {
      throw new Error('foo')
    } catch(e) {
      console.log(e);
    }
    // Error: foo
    // at <anonymous>:2:13
    // 抛出错误并没有捕获到
    // 同步代码之所以没有捕获到期约抛出的错误，是因为没有通过异步模式捕获错误。
    // 期约的异步特性： 是同步对象，也是异步执行的媒介
    try {
      Promise.reject(new Error('foo'))
    } catch(e) {
      console.log(e)
    }
    // Uncaught (in promise) Error: foo
    // at <anonymous>:2:22
  ```

    拒绝期约的错误并没有抛到执行同步代码的线程里，而是通过浏览器异步消息队列来处理的。
    因此，try/catch块并不能捕获该错误。代码一旦开始以异步模式执行，则唯一与之交互的方式就是使用异步结构。
  promise 捕获错误的方式
  ```javascript
    Promise.reject(new Error('foo')).then(res => {
      console.log(res)
    }).catch(err => {
      console.log(err)
    })
  ```
  6. 期约实例方法
  (1) promise.prototype.then()接受两个参数 onResolve 处理程序和 onReject 处理程序, 返回一个新的期约实例。
  ```javascript
    let p = new Promise((resolve, reject) => {
      setTimeout(resolve, 2000);
    })
    p.then(() => { console.log('resolve') }, () => { console.log('reject')})
    let p1 = new Promise((resolve, reject) => {
      setTimeout(resolve, 2000);
    })
    // onResolve 不传参数，最好传 null
    p1.then(null, () => { console.log('reject')})
  ```
  (2) Promise.prototype.catch()用于给期约添加拒绝处理程序，这是一个语法糖，调用它相当于调用Promise.prototype.then(null, () => { console.log('reject')})
  ```javascript
  let p = new Promise((resolve, reject) => {
    reject('err')
  })
  p.catch(() => { console.log('rejected')});
  p.then(null, () => { console.log('rejected')})

  ```
  (3) Promise.prototype.finally()方法用于给期约添加onFinally处理程序，这个处理程序在期约转换为解决或拒绝状态时都会执行。这个方法可以避免onResolved和onRejected处理程序中出现冗余代码。但onFinally处理程序没有办法知道期约的状态是解决还是拒绝，所以这个方法主要用于添加清理代码。
  ```javascript
  let p = new Promise((resolve, reject) => {
    resolve(1)
  })
  // 效果一样
  p.then(() => {
    console.log('resolve')
    console.log('finally 程序')
    loding = false
  }, () => {
    console.log('reject')
    console.log('finally 程序')
    loding = false
  })
  p.finally(() => {
    console.log('finally 程序')
    loding = false
  })
  ```
  7. 当期约进入落定状态时，与该状态相关的处理程序会被排期，不立即执行。邻近处理程序的执行顺序按照程序添加顺序依次执行。
  ```javascript
  let p = new Promise((resolve, reject) => {
    console.log('1.begin resolve')
    resolve('promise')
    console.log('2.after resolve')
  })
  p.then((res) => {
    console.log('3. promise then')
    console.log('res ', res)
  })
  p.then((res) => {
    console.log('4. promise then')
    console.log('res ', res)
  })
  p.then((res) => {
    console.log('5. promise then')
    console.log('res ', res)
  })
  // 运行结果
  1.begin resolve
  2.after resolve
  3. promise then
  res  promise
  4. promise then
  res  promise
  5. promise then
  res  promise
  ```
  8. Promise.all()静态方法创建的期约会在一组期约全部解决之后再解决。这个静态方法接收一个可迭代对象，返回一个新期约。如果有期约拒绝，则第一个拒绝的期约会将自己的理由作为合成期约的拒绝理由。之后再拒绝的期约不会影响最终期约的拒绝理由。
  ```javascript
    let p = Promise.all([
      Promise.resolve(1),
      Promise.resolve(2),
      Promise.reject(3),
      Promise.reject(4),
    ])
    p.then(res => {
      console.log('res ', res)
    }).catch(err => {
      console.log('err ', err)
    })
  ```
  9. Promise.race()静态方法返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像。这个方法接收一个可迭代对象，返回一个新期约。Promise.race()不会对解决或拒绝的期约区别对待。无论是解决还是拒绝，只要是第一个落定的期约，Promise.race()就会包装其解决值或拒绝理由并返回新期约。
  ```javascript
    let p1 = Promise.race([
      Promise.resolve(1),
      Promise.resolve(2),
    ])
    p1.then(res => {
      console.log('res ', res)
    }).catch(err => {
      console.log('err ', err)
    })
    let p2 = Promise.race([
      Promise.reject(1),
      Promise.resolve(2),
    ])
    p2.then(res => {
      console.log('res ', res)
    }).catch(err => {
      console.log('err ', err)
    })
  ```

#### 期约连锁(期约串联)
  之所以可以串联，是因为期约实例的方法(then()、catch、finally)都会返回一个新的期约对象，而这个新期约又有自己的实例方法。
  ```javascript
  // 不灵活
  let delayedResolve = function(str) {
    return new Promise((resolve, reject) => {
      console.log(str)
      setTimeout(resolve, 1000)
    })
  }
  delayedResolve('p1 执行').then(() => delayedResolve('p2 执行')).then(() => delayedResolve('p3 执行')).then(() => delayedResolve('p4 执行'))
  // 运行结果
  // p1 执行
  // p2 执行
  // p3 执行
  // p4 执行
  const addTwo = (x) => {
    return x + 2
  }
  const addThree = (x) => {
    return x + 3
  }
  const addFifth = (x) => {
    return x + 5
  }
  const funcs = [addTwo, addThree, addFifth]
  let p = Promise.resolve(0)
  let index = 1
  for(let func of funcs) {
    p = p.then(func)
    index += 1
  }
  // 串联
  const funcs = [delayedResolve, delayedResolve, delayedResolve, delayedResolve]
  let p = Promise.resolve()
  for(let i = 0; i < funcs.length; i++) {
    const func = funcs[i];
    p = p.then(() => func(`p${i + 1} 执行`))
    console.log('p', p)
  }
  ```
  

#### 期约图
  因为一个期约可以有任意多个处理程序，所以期约连锁可以构建有向非循环图的结构。这样，每个期约都是图中的一个节点，而使用实例方法添加的处理程序则是有向顶点。因为图中的每个节点都会等待前一个节点落定，所以图的方向就是期约的解决或拒绝顺序。
下面的例子展示了一种期约有向图，也就是二叉树：
  ```javascript
    //     A
    //    / \
    //   B   C
    //  / \ / \
    // D  E F  G
  let A = new Promise((resolve, reject) => {
    console.log('A')
    resolve();
  })
  let B = A.then(() => console.log('B'))
  let C = A.then(() => console.log('C'))
  B.then(() => console.log('D'))
  B.then(() => console.log('E'))
  C.then(() => console.log('F'))
  C.then(() => console.log('G'))
  // 运行结果（是对二叉树的层序遍历）
  // A
  // B
  // C
  // D
  // E
  // F
  // G
  ```
  
#### 期约拓展(期约取消和进度追踪)
  1. 中断期约
  注意这里是中断而不是终止，因为 Promise 无法终止，这个中断的意思是：在合适的时候，把 pending 状态的 promise 给 reject 掉。例如一个常见的应用场景就是希望给网络请求设置超时时间，一旦超时就就中断
  ```javascript
  function abortWrapper(p1) {
    let abort
    let p2 = new Promise((resolve, reject) => (abort = reject))
    let p = Promise.race([p1, p2])
    p.abort = abort
    return p
  }
  const req = abortWrapper(request)
  req.then(res => console.log(res)).catch(e => console.log(e))
  setTimeout(() => req.abort('用户手动终止请求'), 2000) // 这里可以是用户主动点击
  ```
  2. 进度追踪
  ```javascript
  class TrackablePromise extends Promise {
    constructor(executor) {
      const  notifyHandlers = []
      super((resolve, reject) => {
        return executor(resolve, reject, (status) => {
          notifyHandlers.map((handler) => {
            handler(status)
          })
        })
      })
      this.notifyHandlers = notifyHandlers;
    }
    notify(notifyHandler) {
      this.notifyHandlers.push(notifyHandler)
      return this
    }
  }

  let p = new TrackablePromise((resolve, reject, notify) => {
    function countDown(x) {
      if(x > 0) {
        notify(`${20*x}%remaining`)
        setTimeout(() => {
          countDown(x - 1)
        }, 1000)
      } else {
        resolve()
      }
    }
    countDown(5)
  })
  p.notify((x) => setTimeout(console.log, 0, 'process:', x))
  p.then(() => setTimeout(console.log, 0, 'done'))
  ```


#### async/await异步函数
  1. async 函数就是 Generator 函数的语法糖。异步编程的最高境界，就是根本不用关心它是不是异步。

  2. 同 Generator 函数一样，async 函数返回一个 Promise 对象，可以使用 then 方法添加回调函数。当函数执行的时候，一旦遇到 await 就会先返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。

  Generator 函数
  ```javascript
  let delay = function (time){
    return new Promise(function (resolve, reject){
      setTimeout(() => {
        console.log(time)
        resolve(time)
      }, time)
    })
  }
  var gen = function* (){
    var f1 = yield delay(1000);
    console.log('f1', f1)
    var f2 = yield delay(2000);
    console.log('f2', f2)
  }
  let g = gen()
  g.next()
  ```
  async/await 函数
  ```javascript
  var gen = async function (){
    var f1 = await delay(1000);
    console.log('f1', f1)
    var f2 = await delay(2000);
    console.log('f2', f2)
  }
  // let g = gen()
  // g.next()
  ```

  3. await 捕获错误
  ```javascript
  let testError = function() {
    return new Promise((resolve, reject) => {
      reject('error')
    })
  }
  async function test() {
    try {
      await testError();
    } catch (err) {
      console.log(err);
    }
  }
  ```
  4. await关键字会暂停执行异步函数后面的代码，让出JavaScript运行时的执行线程。这个行为与生成器函数中的yield关键字是一样的。await关键字同样是尝试“解包”对象的值，然后将这个值传给表达式，再异步恢复异步函数的执行。await关键字期待（但实际上并不要求）一个实现thenable接口的对象，但常规的值也可以。如果是实现thenable接口的对象，则这个对象可以由await来“解包”。如果不是，则这个值就被当作已经解决的期约。即使await后面跟着一个立即可用的值，函数的其余部分也会被异步求值。
  ```javascript
  const delay = function(num) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(num)
      }, 1000)
    })
  }

  async function foo() {
    console.log(2);
    console.log(await delay(8));
    console.log(9);
  }
  async function bar() {
    console.log(4);
    console.log(await 6);
    console.log(7);
  }
  console.log(1);
  foo();
  console.log(3);
  bar();
  console.log(5);
  //（1）打印1；
  //（2）调用异步函数foo()；
  //（3）（在foo()中）打印2；
  //（4）（在foo()中）await关键字暂停执行，向消息队列中添加一个期约在落定之后执行的任务；
  //（5）foo()退出；
  //（6）打印3；
  //（7）调用异步函数bar()；
  //（8）（在bar()中）打印4；
  //（9）（在bar()中）await关键字暂停执行，为立即可用的值6向消息队列中添加一个任务；
  //（10）bar()退出；
  //（11）打印5；
  //（12）顶级线程执行完毕；
  //（13）JavaScript运行时从消息队列中取出解决await期约的处理程序，并将解决的值8提供给它；
  //（14）JavaScript运行时向消息队列中添加一个恢复执行foo()函数的任务；
  //（15）JavaScript运行时从消息队列中取出恢复执行bar()的任务及值6；
  //（16）（在bar()中）恢复执行，await取得值6；
  //（17）（在bar()中）打印6；
  //（18）（在bar()中）打印7；
  //（19）bar()返回；
  //（20）异步任务完成，JavaScript从消息队列中取出恢复执行foo()的任务及值8；
  //（21）（在foo()中）打印8；
  //（22）（在foo()中）打印9；
  //（23）foo()返回。
  ```

  5. await期约串联
  ```javascript
  const addTwo = (x) => {
    return x + 2
  }
  const addThree = (x) => {
    return x + 3
  }
  const addFifth = (x) => {
    return x + 5
  }
  const funcs = [addTwo, addThree, addFifth]
  async function addTen(x) {
    for(const fn of funcs) {
      x = await fn(x)
    }
    return x
  }
  addTen(9).then(console.log)
  ```

  实现sleep
  ```javascript
  async function sleep(delay) {
    return new Promise((resolve, reject) => {
      setTimeout(resolve, delay)
    })
  }
  async function foo() {
    const t0 = Date.now()
    await sleep(1500)
    console.log(Date.now() - t0)
  }
  foo()
  ```




#### promise源码
  1. 浏览器兼容
  ![avatar](./img/promiseUse.png)
  2. MyPromise
  
  ```javascript
  const isFunction = (value) => typeof value === 'function'
  const PENDING = 'pending'
  const RESOLVED = 'fulFilled'
  const REJECTED = 'rejected'
  class MyPromise {
      constructor(executor) {
          this.status = PENDING
          this.value = undefined
          this.reason = undefined
          this.onResolvedCallbacks = []
          this.onRejectedCallbacks = []
          // 成功
          let resolve = (value) => {
              // pending最屏蔽的，resolve和reject只能调用一个，不能同时调用，这就是pending的作用
              if (this.status == PENDING) {
                  this.status = RESOLVED
                  this.value = value
                  // 发布执行函数
                  this.onResolvedCallbacks.forEach(fn => fn())
              }
          }
          // 失败
          let reject = (reason) => {
              if (this.status == PENDING) {
                  this.status = REJECTED
                  this.reason = reason
                  this.onRejectedCallbacks.forEach(fn => fn())
              }
          }
          try {
              // 执行函数
              executor(resolve, reject)
          } catch (err) {
              // 失败则直接执行reject函数
              reject(err)
          }
      }
      then(onFulFilled, onRejected) {
          // onfulfilled, onrejected 都是可选参数
          onFulFilled = isFunction(onFulFilled) ? onFulFilled : data => data
          onRejected = isFunction(onRejected) ? onRejected : err => {
              throw err
          }
          let promise2 = new MyPromise((resolve, reject) => {
              // 箭头函数，无论this一直是指向最外层的对象
              // 同步
              if (this.status == RESOLVED) {
                  setTimeout(() => {
                      try {
                          let x = onFulFilled(this.value)
                          // 添加一个resolvePromise（）的方法来判断x跟promise2的状态，决定promise2是走成功还是失败
                          resolvePromise(promise2, x, resolve, reject)
                      } catch (err) { // 中间任何一个环节报错都要走reject()
                          reject(err)
                      }
                  }, 0) // 同步无法使用promise2，所以借用setiTimeout异步的方式
                  // MDN 0>=4ms
              }
              if (this.status == REJECTED) {
                  setTimeout(() => {
                      try {
                          let x = onRejected(this.reason)
                          resolvePromise(promise2, x, resolve, reject)
                      } catch (err) { // 中间任何一个环节报错都要走reject()
                          reject(err)
                      }
                  }, 0) // 同步无法使用promise2，所以借用setiTimeout异步的方式
              }
              // 异步
              if (this.status == PENDING) {
                  // 在pending状态的时候先订阅
                  this.onResolvedCallbacks.push(() => {
                      // todo
                      setTimeout(() => {
                          try {
                              let x = onFulFilled(this.value)
                              resolvePromise(promise2, x, resolve, reject)
                          } catch (err) { // 中间任何一个环节报错都要走reject()
                              reject(err)
                          }
                      }, 0) // 同步无法使用promise2，所以借用setiTimeout异步的方式
                  })
                  this.onRejectedCallbacks.push(() => {
                      // todo
                      setTimeout(() => {
                          try {
                              let x = onRejected(this.reason)
                              resolvePromise(promise2, x, resolve, reject)
                          } catch (err) { // 中间任何一个环节报错都要走reject()
                              reject(err)
                          }
                      }, 0) // 同步无法使用promise2，所以借用setiTimeout异步的方式
                  })
              }
          })
          return promise2
      }
      catch(onReJected) {
          // 返回一个没有第一个参数的then方法
          return this.then(undefined, onReJected)
      }
  }
  ```



#### 参考
  [JavaScript高级程序设计（第4版）](https://book.douban.com/subject/35175321/)
  [async 函数的含义和用法](https://www.ruanyifeng.com/blog/2015/05/async.html)
  [promise源码](https://www.cnblogs.com/xinggood/p/11836096.html)
  



