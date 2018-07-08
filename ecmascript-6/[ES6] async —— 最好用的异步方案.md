# [ES6] async/await —— 最好用的异步方案

> 1.  async 是什么？这里主要强调用法，从一个个的例子感受何为 async
> 1.  async 实现原理？
>     - 基于生成器 + promise
>     - 只是一个语法糖
> 1.  实践指南

JS 运行时被设计为 **单线程** 结构，因此，在诞生之初，设计者就有意将各类资源操作定义为 **异步** 操作，使得程序免于被耗时操作所阻塞。

## `async` 用法解析

`async/await` 是成对的语法结构， async 用于声明一个函数是异步的；await 则用于声明关键字后面的函数调用是异步的。注意 async 函数并不要求必须包含 await 语句，但 await 语句只能出现在 async 函数中。举个简单的例子：

```javascript
async function foo() {
  await bar();
}
```

这是一个完整的 `async/await` 函数，但与普通的调用语句并无差别，并没有表现出 async 的真正威力。现在，我们来实现一套账号注册逻辑，分为几个步骤：

1.  验证 email 是否已经被注册
2.  创建账号记录
3.  写日志
4.  返回账号记录

如果以 `Promise` 方式实现，代码大致为：

```javascript
function reg(email, password) {
  let profile = null;
  checkIfExist(email)
    .then(isExist => {
      if (isExist) {
        throw new Error(`email: ${email} you trying to reg has been signed`);
      } else {
        return addUser(email, password);
      }
    })
    .then(p => {
      profile = p;
      return createLog("reg", email, password);
    })
    .then(log => sendNotify(log))
    .then(() => profile);
}
```

上面的代码是可以实现需求，Promise 的特点是将异步操作变成一串函数调用，在过去确实解决了 **回调地狱** 的问题，但却引入了一些问题，
比如如果没有在调用链中 `catch` 异常，则异常会被掩埋，开发者很难准确获得异常信息，虽然后来出现了 `window.onunhandledrejection` 方案，但就开发而言还是略麻烦。
另外就是状态的传递，上面例子中的 `profile` 就是为了在不同 promise 之间传递状态，而不得不添加的变量，这已经违背了纯函数的定义了，很不优雅。

而以 `async/await` 方式实现，代码则优雅许多：

```javascript
async function reg(email, password) {
  try {
    const isExist = await checkIfExist(email);
    if (isExist) {
      throw new Error(`email: ${email} you trying to reg has been signed`);
    } else {
      const profile = await addUser(email, password);
      const log = await createLog("reg", email, password);
      await sendNotify(log);
      return profile;
    }
  } catch (e) {
    createLog("error", e);
  }
}
```

这正是 `async/await` 的意义 —— 以一种接近于 **同步** 调用的语法结构，更好的解决 **异步** 调用代码结构上的复杂性。同步结构更符合人类的思维逻辑，这让我们更容易做好代码的设计、管理。

async and catch
普通 catch =》 嵌套 catch =》 这已经是语言层面的问题，超过了异步处理讨论的范围。

值得注意的是，`await` 后面调用的并不要求必须是异步函数！
如果调用的函数返回一个 `Promise` 对象，则 `async` 函数会在 `Promise` 对象 `fullfill` 后执行后续语句；
如果调用的是一个普通函数， `await` 会将函数结果以异步回调后续的处理流程，行为上与 `Promise.resolve` 非常相似。

`async` 的行为与 `Promise` 有诸多关联

从形式上看跟同步操作代码非常相像
举个最简单的例子：

， `bar` 可以是一个异步操作，也可以不是
从语法结构的角度来说， `async` 只有

ES7 在语法上支持 `async` 结构，用法非常简单，所有类型的函数定义语法都支持 `async` 声明：

```javascript
async function asyncFunc() {...}

// 箭头函数
const asyncFunc = async () => {...};

// 类方法
class Foo {
  async bar() {...}
}
```
