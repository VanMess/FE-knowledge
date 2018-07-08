# [ES6] async/await 语法笔记

## 特点：

1.  原理上依赖于 promise + 生成器 —— 生成器用于流程控制，promise 实现交互
2.  async 函数会逐条执行语句，当遇到 await 调用时，则挂起当前的函数调用，直到 await 的值 resolve 后，才继续执行。函数挂起后会保留当前的执行上下文，下次执行时恢复，流程上完全依赖于生成器规则
3.  async 函数中并不要求必须包含 await 语句，无论函数的内容如何，最终返回值都会被包装为 promise 对象
4.  await 关键字后面的语句都会被转换为 `thenable` 对象，即使是 `await 1;`
5.  async 函数中不要求必须包含 await 语句，await 语句则必须出现在 async 函数中
6.  对于串联的依赖，写不同的 await 语句，前后串起来即可；对于并联的异步，则可以用 promise.all 函数；也可以用 promise.any，只要 await 语句返回的是一个 thenable 对象即可。
7.  一个 async 函数中可以包含任意多的 await 语句，顺序执行，到 await 语句就暂停，直到 await 语句 resolve
8.  假设 `foo` 是一个返回 `promise` 对象的函数，则 `const result = foo()` 中的 result 是一个 promise 对象； `const result = await foo()` 中的 result 则是 foo 函数 resolve 的值。
9.  await 更像是一个异步求值语句 => 异步获取 promise resolve 的值
10. async 可以用在任何类型的函数定义语句中，包括：function 关键字前、箭头函数、类方法
11. babel 已经有很好的支持，只要 `import '@babel\polyfill'` 即可
12. async 可以配合 try-catch 语句使用，行为上与 promise 调用链最后的 catch 语句相似
13. catch 中还可能继续触发异常，因此，必要时可以在 catch 中继续写 try-catch => 语法上已经决定了只能这么做了，这是语言层面的问题

## 与其他异步方案的对比

1.  回调地狱的问题：
    - 回调层次可能很深
    - 并联请求很难监控
1.  promise 确实解决了回调地狱的诸多问题，但同时也有其他问题：
    - 多个 then 之间，状态难以传递
    - 你可以捕获异常，但你捕获不到堆栈
1.  async 的陷阱
    - [没有相互以来的异步调用，却有先后等待的次序](https://medium.freecodecamp.org/avoiding-the-async-await-hell-c77a0fb71c4c)

## 资料

https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9
https://medium.com/@rafaelvidaurre/truly-understanding-async-await-491dd580500e
https://codeburst.io/javascript-es-2017-learn-async-await-by-example-48acc58bad65
https://juejin.im/entry/59f6fb1f6fb9a045030f5e28
