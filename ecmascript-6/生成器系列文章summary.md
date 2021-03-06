# 【深入 ES6】生成器入门

- [x] 1.  基本特性
  - 不会立即执行函数体的内容
  - 返回 [Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 对象
  - 执行 => 返回中间值 => 挂起 => 执行 ==> 结束
  - return 语句
  - throw 语句
  - 传值
- [ ] 2.  生成器并联
- [ ] 3.  异步 vs 生成器
- [ ] 4.  原理： 执行上下文挂起
- [ ] 5.  应用场景： 1.  未知的迭代； 2.异步操作

## 基本特性

生成器是一种特殊函数，调用时并不会执行函数体的内容，而是将函数体包装为 [Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 对象。
[Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 向外提供了一系列交互机制，实现生成器函数 **内外环境的数据交换** 。

### 如何定义生成器

语法层面，生成器函数有两个特殊点：

1.  必须在 `function` 关键字后加 `*`
2.  可在函数体内部使用 `yield` 关键字

举个例子：

```javascript
function* genGreeting() {
  yield "hello";
  yield "world";
}
```

yield 关键字功能上来说，只是声明应用进入 **暂停** 态，不一定非得返回值

### 运行特性

执行层面，生成器函数的运行遵循如下流程：

```bash
执行 => 返回中间值 => 挂起 => 执行 ==> 结束
```

Generator 内部就是一个特殊的状态机，包含如下状态： suspended、closed

return ： 提前进入 closed 状态
throw : 提前进入 closed 状态

运行过程中会保存执行上下文信息

## 运行原理

调用生成器函数，生成 `Generator` 时，并不会开始执行函数代码，此时 `Generator` 是 **挂起状态**；
调用 `Generator` 对象的 `next` 函数，对象进入 **运行态**；
函数执行完毕，或使用者提前调用 `return` 或 `throw` 函数后，对象进入 **关闭态**。

## 生成器串联

生成器串联是指，在生成器的内部调用生成器，举个例子：

```javascript
function* genWords() {
  yield "hello";
  yield "world";
}

function* regen() {
  yield "I say: ";
  yield* genWords();
}
```

## [Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 对象

`Generator` 是一个实现了 [Iteration 协议](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterable_protocol) 及 [Iterator 协议](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterator_protocol) 的对象。
