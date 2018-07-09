# [ES6] async/await 应用指南

## async/await 是什么

async/await 是 ES7 引入的新的异步代码 [规范](https://www.ecma-international.org/ecma-262/8.0/#sec-async-function-definitions)，它提供了一种新的编写异步代码的方式，这种方式在语法层面提供了一种形式上非常接近于同步代码的异步非阻塞代码风格，在此之前我们使用的多是异步回调、 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 模式。
从实现上来看 async/await 是在 [生成器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)、Promise 基础上构建出来的新语法：以 **生成器** 实现流程控制，以 Promise 实现异步控制。
Node 自 [v8.0.0](https://nodejs.org/en/blog/release/v8.0.0/) 起已经完全支持 async/await 语法，[babel](https://babeljs.io/docs/en/babel-plugin-syntax-async-functions) 也已经完全支持 async/await 语法的转译。

下面，我们以一个一个实例的方式，由浅入深介绍 async/await 语法的使用。

## 一个简单的实例

我们来实现一个获取登录用户信息的函数，逻辑如下：

1.  获取用户登录态
2.  如果用户已经登录，返回对应的用户信息
3.  如果用户未登录，跳转到登录页

### 以回调方式实现

**回调** 在最初版本的 JS 就已经出现，可谓历史悠久，到现在也还保持着相当的活力。
如果以回调方式实现上述需求，代码大概如下：

```javascript
function getProfile(cb) {
  isUserLogined(req.session, (err, isLogined) => {
    if (err) {
      cb(err);
    } else if (isLogined) {
      getUser(req.session, (err, profile) => {
        if (err) {
          cb(err);
        } else {
          cb(null, profile);
        }
      });
    } else {
      cb(null, false);
    }
  });
}
```

感受到臭味了吗？这里我们还只是实现了两层的异步调用，代码中就已经有许多问题，比如重复的 `if(err)` 语句；比如层层嵌套的函数。
另外，如果在层层回调函数中出现异常，调试起来是非常让人奔溃的 —— 由于 `try-catch` 无法捕获异步的异常，我们只能不断不断的写 `debugger` 去追踪，简直步步惊心。
这种层层嵌套导致的代码臭味，被称为 [**回调地狱**](http://callbackhell.com/)，在过去是困惑社区的一个大问题。

### 以 Promise 方式实现

`Promise` 模式最早只是社区出现的一套解决方案，但凭借其优雅的链式调用语句，得到越来越多人的青睐，最终被列为 ES6 的正式规范。
上面的需求，如果以 Promise 模式实现：

```javascript
function getProfile() {
  return isUserLogined(req.session)
    .then(isLogined => {
      if (isLogined) {
        return getUser(req.session);
      }
      return false;
    })
    .catch(err => {
      console.log(err);
    });
}
```

ok，这减少了些模板代码，也有了一致的异常 catch 方案。但这里面也有其他的一些坑，比如，如果我们要 `resolve` 两个不同 Promise 的值？假设上面的例子中，我们还需要返回用户的日志记录：

```javascript
function getProfile() {
  return isUserLogined(req.session)
    .then(isLogined => {
      if (isLogined) {
        return getUser(req.session).then(profile => {
          return getLog(profile).then(logs => Promise.resolve(profile, logs));
        });
      }
      return false;
    })
    .catch(err => {
      console.log(err);
    });
}
```

上面的代码在 `getUser.then` 中嵌套了一层 `getLog.then` ，这在代码上破坏了 Promise 的链式调用法则，而且，`getUser.then` 函数中发生的异常是无法被外层的 `catch` 函数捕获的，这破坏了异常处理的一致性。

Promise 的另一个问题，是在 `catch` 函数中的异常堆栈不够完整，导致难以追寻真正发生错误的位置。比如以下代码中：

```javascript
function asyncCall(){
    return asyncFunc()
      .then(()=>asyncFunc())
      .then(()=>asyncFunc())
      .then(()=>asyncFunc())
      .then(()=>throw new Error('oops'));
}

asyncCall()
  .catch((e)=>{
    console.log(e);
    // 输出：
    // Error: oops↵    at asyncFunc.then.then.then.then (<anonymous>:6:22)
  });
```

由于抛出异常的语句是在一个匿名函数中，运行时会认为错误发生的位置是 `asyncFunc.then.then.then.then`，假如代码中大量使用了 `asyncFunc` 函数，那么上面的报错信息就很难帮助我们准确定位错误发生的位置。
我们当然可以给每个 `then` 的回调函数赋予一个有意义的名词，但这又丧失了箭头函数、匿名函数的简洁。

### 以 async/await 方式实现

最后，终于轮到我们这次的主题 —— async/await 方式的异步代码，虽然这是一个 ES7 规范，但配合强大的 babel，现在已经可以大胆使用。
以上需求的实现代码：

```javascript
async function getProfile() {
  const isLogined = await isUserLogined(req.session);
  if (isLogined) {
    return await getUser(req.session);
  }
  return false;
}
```

代码比上面两种风格要简单了许多，形式上就是同步操作流程，与我们的需求描述也非常非常的接近。

async 关键字用于声明一个函数是异步的，可以出现在任何函数声明语句中，包括：普通函数、箭头函数、类函数。普通函数的 `constructor` 是 [`Function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)， 而被 async 关键字修饰的函数则是 [`AsyncFunction`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction) 类型的：

```javascript
Object.getPrototypeOf(function() {}).constructor;
// output
// AsyncFunction() { [native code] }

Object.getPrototypeOf(async function() {}).constructor;
// output
// Function() { [native code] }
```

await 关键字只能在 async 函数中使用，用于声明一个异步调用，比如上面例子中的 `const isLogined = await isUserLogined(req.session);`，当 async 风格的 `getProfile` 函数执行到该语句时，会挂起当前函数，将后续语句加入到 `event loop` 循环中，这一点与 [**生成器**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 执行特性相同。
直到 `isUserLogined` 函数 `resovle` 后，才继续执行后面的语句。

我们可以在 async 函数中编写任意数量的 await 语句，async 函数的执行会一直处在 **执行-挂起-执行** 的循环中，这种特性得到了语言层面的支持，并不需要我们为此编写多余的代码，这就为复杂的异步场景提供便捷的实现方案，比如：

```javascript
async function asyncCall() {
  const v1 = await asyncFunc();
  const v2 = await asyncFunc(v1);
  const v3 = await asyncFunc(v2);
  return v3;
}
```

到这里，我们已经简单了解了 async/await 的用法，这种同步风格的异步处理方案，相比而言会更容易维护。

## async 中的异常处理

上面我们提到，在 Promise 模式中，`catch` 函数难以获得完整的异常信息，导致在 Promise 下做调试变得困难重重，那在 async/await 中呢？
我们来看一段代码：

```javascript
async function asyncCall() {
  try {
    await asyncFunc();
    throw new Error("oops");
  } catch (e) {
    console.log(e);
    // output
    // Error: oops  at asyncCall (<anonymous>:4:11)
  }
}
```

相比 Promise 模式，上面代码中异常发生的位置是 `asyncCall` 函数！相对而言，容易定位了许多。

## 并联的 await

async/await 语法确实很简单好用，但却容易用岔了。以下面代码为例：

```javascript
async function retriveProfile(email) {
  const user = await getUser(email);
  const roles = await getRoles(user);
  const level = await getLevel(user);
  return [user, roles, level];
}
```

上面代码实现了获取用户基本信息，然后通过基本信息获取用户角色、级别信息的功能，其中 `getRoles` 与 `getLevel` 两者之间并无依赖，是两个并联的异步操作。
但代码中 `getLevel` 却需要等待 `getRoles` resolve 之后才能执行。并不是所有人都会犯这种错误，而是同步风格很容易诱惑我们忽略掉真正的异步调用次序，而陷入过于简化的同步思维中。写这一段的目的正是为了警醒大家，async 只是形式上的同步，根本上还是异步的，请注意不要让使用者把时间浪费在无谓的等待上。
上面的逻辑，用一种稍微 **绕** 一些的方式来实现，就可以避免这种性能损耗：

```javascript
async function retriveProfile(email) {
  const user = await getUser(email);
  const p1 = getRoles(user);
  const p2 = getLevel(user);
  const roles = await p1;
  const level = await p2;
  return [user, roles, level];
}
```

注意，代码中的 `getRoles` 、`getLevel` 函数都没有跟在 await 关键字之后，而是把函数返回的 Promise 存放在变量 `p1`、`p2` 中，后续才对 `p1`、`p2` 执行 await 声明， `getRoles` 、`getLevel` 就能同时执行，不需等待另一方的完成。

这个问题在循环场景下特别容易发生，假设我们需要获取一批图片的大小信息：

```javascript
async function retriveSize(imgs) {
  const result = [];
  for (const img of imgs) {
    result.push(await getSize(img));
  }
}
```

代码中的每次 `getSize` 调用都需要等待上一次调用完成，同样是一种性能浪费。同样的功能，用这样的方式会更合适：

```javascript
async function retriveSize(imgs) {
  return Promise.all(imgs.map(img => getSize(img)));
}
```

这实际上已经回退到了 Promise 模式，所以为了写出良好的 async/await 代码，建议还是认真学习学习 Promise 模式
