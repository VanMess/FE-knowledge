# Vue Mount 过程笔记

## src/core/instance/lifecycle.js

1. 真正的 mount 方法定义在改文件中
2. 几点更新的核心函数是 `Vue.prototype._update`
3. 节点更新会有调用 patch 函数，不同运行环境的 patch 函数不同

## render

1. 真正执行 render 的函数定义在： src/core/instance/render.js
2. render 过程有一些兜底措施：
   1. 开发模式下发生异常时，调用用户提供的 renderError 函数
   2. 返回上一次 render 的 vnode 值，避免出现渲染空白
3. transformModel => 用于转换 model 为 props
4. 重点还是在于 `__patch__` 函数

## 何时初始化子组件？

1. 代码生成的 render 函数执行时
2. render 函数中以 \_c 方式创建子组件
3. `_c` 函数会调用 `core/create-element.js` 中的 \_createElement 函数
4. 重点在于 patch 方法

如果 is 值为 false 时，渲染空组件

先创建 vnode，在创建组件实例？

## slot 组件会渲染成啥？

所有事件回调会解析为 on 对象的属性

根组件的初始化方法与非跟组件有区别

跟组件是在组件 new 的时候开始执行 init，非根组件则是在根组件创建 vnode 的过程中逐步初始化、添加的

第一次调用 init 是在 new vue 实例的时候。后续组件的 init，则在 path 的 createComponent 函数时调用

patch 入口第一个调用的函数，是： _createElement
