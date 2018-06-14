# 分享一个 iView 调试案例

1.  问题的表征，同时提供示例
2.  查找原因的过程，包括：
    - 配置便利的 debugger 环境
    - 异常位置的参数 exception
    - 解决方案 1： 用户侧不要使用 null 做 key
    - 解决方案 2： 直接过滤掉 tag 为 undefined 的 node
3.  深究
    - 为什么会传这样一个 node？
    - auto-complete、 i-select 组件所生成的 slot.default 的差异，4 vs 5? 在传入 i-select 时，凭空多了一个
    - 可能性：
      1.  iview 插入了一个空节点；
      2.  vue 生成了一个空节点
    - 针对第一种情况，在 i-select 的 data 函数中，打印 len ，发现没有出现变化
    - 针对第二种情况，查看 auto-complete 生成的 render 函数，发现多一个字符串节点
    - vue 解析 dom 的算法，与浏览器解析算法一直，会在节点末尾解析多一个空格
    - 解决方案：
      1.  注意编写格式
      2.  在 i-select 中过滤掉空节点
4.  针对前端开源库的调试经验
    - 在开发环境中使用源码

> 当开源库报错时，该怎么调试，找到问题的核心？
> 本文通过分享一个 iView 组件的调试过程，阐释
> 但不同框架、库有不同的实现，在现下的前端环境很难提取一些通用的处理方法，

## 版本升级触发的 bug

最近抽空对项目用的 iView 版本做升级，从 2.6.0 升到 2.14.2，原以为不会有太大问题，但在 [AutoComplete](https://www.iviewui.com/components/auto-complete) 组件上报了一些错误：

![异常信息截图](../assets/iview-ac-debug/bug-report.png)

分析调用的上下文代码后，发现在将 `AutoComplete` 组件的值设置为 `null` 时，会触发这个异常，我简化了调用的上下文代码，做了一个 [Demo](https://codepen.io/vanmess/pen/ZRJGma)，建议点进去看看，直观感受下， **记住：请打开控制台看报错信息**。

从异常信息看，问题的核心是 `i-select` 组件发生 `TypeError: Cannot read property 'propsData' of undefined`，也就是有一些 `undefined` 值在预期之外地流入 `i-select` 逻辑中。

## 从源码 Debug

首先，我们来看看，抛出异常的代码，如下：

```javascript
var applyProp = function(node, propName, value) {
  (0, _newArrowCheck3.default)(undefined, undefined);

  return (0, _extends4.default)({}, node, {
    componentOptions: (0, _extends4.default)({}, node.componentOptions, {
      propsData: (0, _extends4.default)(
        {},
        // 这一句抛出了 undefined 异常
        node.componentOptions.propsData,
        (0, _defineProperty3.default)({}, propName, value)
      )
    })
  });
}.bind(undefined);
```

默认情况下，我们引入 iview 库的方式为： `import iView from 'iview';`，入口是 `iview/dist/iview.js` 文件，这是经过编译的版本，
上面这一段正是编译后的代码。编译后的代码，是面向机器而不是人的，所以在语义、结构等方面有些许损耗，不利于调试，因此，我们需要配备一个更友好的环境。

### 配置调试环境

#### 1. 在开发环境中使用源码

通常，引入 iview 库的方式如下：

```javascript
// main.js
import Vue from "vue";
import iview from "iview";

Vue.use(iview);
```

这里做一个修改，从 iview 源码直接：

```javascript
// main.js
import Vue from "vue";
import iview from "iview/src/index";

Vue.use(iview);
```

#### 2. 使用 babel-loader 加载

引入源码后， 编译环境可能会报一个错：

![webpack异常信息截图](../assets/iview-ac-debug/error-where-compile.png)

这是 [babel-loader](https://github.com/babel/babel-loader) 的配置造成。通常情况下，为了提升编译速度，会通过配置 `babel-loader` 忽略 `node_modules` 路径，
此时是 webpack 会使用其他 loader 加载 `iview/src/index.js`，此时就有可能报错。
因此，修改 `babel-loader`配置将 iview 包含进来即可，形如：

```javascript
{
  test: /\.js$/,
  loader: 'babel-loader',
  include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client'), resolve('node_modules/iview/src')]
}
```

#### 3. 修改 export 方式

修改配置后，重新运行 webpack，又报了一个错：

![导出包异常报警](../assets/iview-ac-debug/iview-src-export-warn.png)

进入 `iview/src/index.js` 文件，发现导出语句是这样的：

```javascript
module.exports.default = module.exports = API; // eslint-disable-line no-undef
```

不知道是为了兼容旧版本，还是为了兼容其他环境，iview 在这里选择 `exports` 形式的包导出方法，额，这里先不深究，将代码直接改为：

```javascript
// module.exports.default = module.exports = API; // eslint-disable-line no-undef
export default API;
```

> 到这里，理论上程序已经可以正常运行了，如果还有遇到其他问题，欢迎在文章下面留言

### 重新审查异常信息

[vue-cli](https://github.com/vuejs/vue-cli) 初始化的，

默认情况下，我们引入 iview 库的方式为： `import iView from 'iview';`。这里为了方便调试，修改为 `import iView from 'iview/src/index';`。

修改引用后，会报错：

修改 babel-loader 配置为：

```javascript
{
  test: /\.js$/,
  loader: 'babel-loader',
  include: [
    utils.resolve('src'),
    utils.resolve('test'),
    utils.resolve('node_modules/iview/src')
  ]
}
```

重新运行 `npm start`

iview 导出报的语句 `module.exports.default = module.exports = API;`

计算 options 的逻辑有问题

当把值设置为 `null`，会导致生成多一个 `node`:

```
{tag: undefined, data: undefined, children: undefined, text: " ", elm: undefined, …}
```

解决方案：

1.  应用层面，不要设置值为 null
2.  在 `select.vue` 组件中做过滤
