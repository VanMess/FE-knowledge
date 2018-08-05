# iview 升级指南 —— MenuItem 篇

> iview 在今年 7 月 28 号发布了 3.0.0 版本，大版本升级往往意味着功能、接口的大变更。
> 虽然官网已经有长长的[更新日志](https://www.iviewui.com/docs/guide/update)，但看起来还是有些抽象了，
> 所以我决定做个新旧版本的比较，盘点新版本到底为我们带来了什么新特性。
>
> 本篇是系列文章的第三篇，重点并不在介绍 [MenuItem](https://www.iviewui.com/components/menu) 的功能特性，而在于对其代码的讨论； 对其设计的思考。
> 班门弄斧，见谅。

## 唯一新增的特性 —— 支持链接模式

循例是该先聊聊新特性的。Menu 有四个关联的组件，分别为：Menu、MenuItem、SubMenu、MenuGroup，
这些组件的新旧版本之间并没有太大差异，向后兼容的很好，理论上可以平滑升级。
新版本只有 MenuItem 增加了一个特性：**支持链接模式**，可以通过向组件传入 `to` 属性启用，效果与 [链接模式的 Button](http://www.thinkinfe.tech/iview-migration-button/) 完全一样，这里就不赘述了。

## 问题

MenuItem 是一个非常非常简单的组件，一开始觉得并没有太多好写的，细细看了代码...个人感觉问题不少，还是有必要单独写一篇文章聊聊的。

### 问题一： 代码重复

首先，依然是代码重复的问题，在 [`Button` 篇中](http://www.thinkinfe.tech/iview-migration-button/) 我们已经见识了一些无意义的重复，在 `MenuItem` 组件中也是不遑多让啊：

```html
<template>
    <a
        v-if="to"
        :href="linkUrl"
        :target="target"
        :class="classes"
        @click.exact="handleClickItem($event, false)"
        @click.ctrl="handleClickItem($event, true)"
        @click.meta="handleClickItem($event, true)"
        :style="itemStyle"><slot></slot></a>
    <li v-else :class="classes" @click.stop="handleClickItem" :style="itemStyle"><slot></slot></li>
</template>
```

这段模板有两处重复，一是标签，二是事件绑定。

#### 1. 重复的标签定义

模板中，通过判断 `to` 属性，确定需要渲染的标签类型，用于兼容新增的链接模式，这种写法很符合直觉，但有另一种更优雅的方案：[`is`](https://cn.vuejs.org/v2/api/#is)特性，同样的功能，用 `is` 实现：

```javascript
/* 模拟MenuItem组件 */
Vue.component("MenuItem", {
  name: "MenuItem",
  // 简化过的模板，干净无重复
  template: `<component :is="tagName" v-bind="tagProps"><slot></slot></component>`,
  props: {
    to: { type: String, required: false }
  },
  computed: {
    isLink() {
      const { to } = this;
      return !!to;
    },
    // 使用计算器属性，按需计算标签名
    // 这种方式可以承载更复杂的计算逻辑
    tagName() {
      const { isLink } = this;
      return isLink ? "a" : "li";
    },
    // 通过计算器+v-bind 语法，实现按标签类型传递不同属性
    // 这里把本来放在模板的运算，转嫁到计算器上
    tagProps() {
      const { isLink, to } = this;
      const baseProps = { class: "menu-item", style: { display: "block" } };
      if (isLink) {
        return Object.assign(baseProps, {
          href: to,
          target: "_blank"
        });
      }
      return baseProps;
    }
  }
});
```

示例中使用了 [computed 属性](https://cn.vuejs.org/v2/api/#computed)、[v-bind](https://cn.vuejs.org/v2/api/#v-bind)、[is](https://cn.vuejs.org/v2/api/#is) 三种特性，把本应在模板做的计算转移到计算属性；通过 `v-bind` 绑定复杂对象；通过 `is` 渲染不同的标签类型...达成与 iView 相同的功能，运行效果欢迎到 [在线 demo](https://jsfiddle.net/fanwenjie/eywraw8t/239268/) 体验。这种写法，有两个好处，一是减少模板上的重复；二是减少模板上的计算，转而在计算属性上实现，配合缓存效果，有一定的性能提升。

#### 2. 重复的事件绑定

另外一个问题，在 iView 的 MenuItem 中，`a` 标签会重复绑定三次相关的 click 回调，分别配以 `exact`、`ctrl`、`meta`，这种写法在 `Button` 组件也出现过，用以模拟 `a` 标签的不同点击效果，之前在 [`Button` 篇](http://www.thinkinfe.tech/iview-migration-button/#2)已做过深入讨论，这里不再赘述。

### 问题二：不符合 html 标准

现在，我们看看 Menu 与 MenuItem 的模板代码：

```html
<!-- Menu 组件模板部分源码 -->
<template>
    <ul :class="classes" :style="styles"><slot></slot></ul>
</template>
<!-- MenuItem 组件模板部分源码 -->
<template>
    <a v-if="to"><slot></slot></a>
    <li v-else :class="classes"><slot></slot></li>
</template>
```

如果没有 `to` 属性，MenuItem 渲染为 `li`，这没问题，但如果是链接模式，渲染结果就会是：

```html
<ul>
    <a></a>
    <a></a>
    <a></a>
    ...
</ul>
```

`ul` 中直接包含了 `a`！遥想我最初学习 html 的时候，就已经被一再警告 `ul` 就应该老老实实包着 `li`，确实也偶尔会看到其他一些框架漠视这条规则，没成想在 iView 这里也能遇到。
h5 包容性是很强，这段代码完全可以 work，没毛病，但没毛病不代表足够好，我们本可以做的更好，为什么不选择做的更好呢？
解法很简单，依然是 `is` 特性，只需多一层包裹，核心代码如下：

```html
<template>
<li class="menu-item">
  <component :is="tagName">
    <slot></slot>
  </component>
</li>
</template>

<script>
export default {
  computed: {
    tagName() {
      const { isLink } = this;
      return isLink ? "a" : "span";
    }
  }
};
</script>
```

### 问题三： 父子组件通信

这个问题有点复杂，要讲述清楚并不容易，还望读者朋友们能给多些耐心。

我注意到在 MenuItem 组件中有这样 [一行代码](https://github.com/VanMess/iview/blob/3.0.0/src/components/menu/menu-item.vue#L77)：`this.$on('on-update-active-name', (name) => {...}`，MenuItem 会在回调中给自身设置各种值。
事件绑定的代码用的多了，但这种组件自己侦听自己的方式却不多见，更奇怪的是 MenuItem 并没有 `$emit` 过 `on-update-active-name` 事件。
出于好奇，我仔细翻查源码，发现真正发出 `on-update-active-name` 事件的是父级 [Menu 组件](https://github.com/VanMess/iview/blob/3.0.0/src/components/menu/menu.vue#L76)！

一般情况下，Menu 与 MenuItem 是以父子关系成对出现的组件，比如：

```html
<template>
    <Menu active-name="1">
        <MenuItem name="1">内容管理</MenuItem>
        <MenuItem name="2">用户管理</MenuItem>
    </Menu>
</template>
```

上例中，改变 Menu 的`active-name`值后，Menu 会执行 `this.broadcast('MenuItem', 'on-update-active-name', this.currentActiveName);`，即调用`broadcast`函数，**向下广播** `on-update-active-name` 事件，注意我们的关键字：**向下广播**！
1.x 版本的 Vue 确实提供过两种传播事件的方法：`$dispatch`、`$broadcast`，其中 `$broadcast` 用于父组件向子组件 **传播** 事件，但到 2.x 时放弃了这种设计，[官网](https://cn.vuejs.org/v2/guide/migration.html#dispatch-%E5%92%8C-broadcast-%E6%9B%BF%E6%8D%A2) 提供的说法是这样的：

> 因为基于组件树结构的事件流方式实在是让人难以理解，并且在组件结构扩展的过程中会变得越来越脆弱。
> 这种事件方式确实不太好，我们也不希望在以后让开发者们太痛苦。并且 `$dispatch` 和 `$broadcast` 也没有解决兄弟组件间的通信问题。

我确信这是一个合理的设计优化 —— `$broadcast` 这种组件通讯方式会增加父子组件间的耦合性，无论是业务层面的开发，还是框架层面的开发，都应该摒弃这种设计模式。
但 iView 却大方复辟，不惜自行实现了 [一套 `$broadcast` 逻辑](https://github.com/VanMess/iview/blob/3.0.0/src/mixins/emitter.js#L1)，为什么？

我认为一种可信的说法是：这是不得已的妥协。
Menu 组件提供了 `active-name` 属性，用于指明当前处于激活态的菜单项，**但真正使用 `active-name` 属性的则是 MenuItem 组件**。那么 Menu 从用户拿到 `active-name` 后，如何传递到 MenuItem 组件呢？iView 选择了通过向下广播事件的方式，将值传递给 Menu 组件下的 MenuItem，合理有效，只是 `broadcast` 的复辟，让我觉得非常不舒服。

问题梳理清楚了，那么如何优化？

#### 1. 通过 Vuex 管理状态

最简单的方式，是遵循 [Vue 官网的建议](https://cn.vuejs.org/v2/guide/migration.html#dispatch-%E5%92%8C-broadcast-%E6%9B%BF%E6%8D%A2)，使用 Vuex 管理状态。这在日常业务开发中是相当有效的，但作为一个框架却万万使不得 —— 你总不能强行绑着另一个框架，要求用户必须同时使用吧？

作为一个变通，也可以设计一个全局状态变量，但这必然又会引发更多问题。

#### 2. 通过 JSX 实现

另一种方法是通过 JSX 方式，在渲染 MenuItem 前以 props 方式，将 `active-name` 给传过去：

```javascript
Vue.component("MenuItem", {
  render() {
    const {
      $slot: { default: children },
      activeName
    } = this;

    return (
      <ul>
        {children.map(node => {
          node.props = { activeName };
          return node;
        })}
      </ul>
    );
  }
});
```

如果我们只有 Menu、MenuItem，那上面的方式已经足够实现功能，也算是比较优雅，但如果把 SubMenu、MenuGroup 组件加入考虑范围，那么问题就会变得更复杂 —— `active-name` 需要从 Menu 跨过中间的 SubMenu、MenuGroup 传递到 MenuItem。这种跨组件的信息传递，在 JSX 环境下，我只想到两种解决方案：在 Menu 递归查找 MenuItem 组件；在 SubMenu、MenuGroup 中重复定义 props 的赋值逻辑。

最近新冒出来一个 UI 库 —— [`ant-design`](https://github.com/vueComponent/ant-design-vue)，它的 [`Menu`](https://github.com/vueComponent/ant-design-vue/blob/master/components/vc-menu/Menu.jsx) 正是基于 JSX 方式实现的，原谅我才疏学浅，看起来实现费劲吃力。

#### 3. 通过 [provide/inject](https://cn.vuejs.org/v2/api/#provide-inject) 实现

Vue 2.2.0 版本后提供了 [provide/inject](https://cn.vuejs.org/v2/api/#provide-inject) 特性，官网是这么介绍的：

> 这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，
> 并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

这真是一个大杀器 —— 祖先组件可以声明需要向所有后代传递的值；而后代组件，无论多深层次的后代，都可以按需订阅感兴趣的内容。我用这个特性做了个简单的 [demo](https://jsfiddle.net/fanwenjie/eywraw8t/239753/)，核心代码：

```javascript
Vue.component("MenuItem", {
  template: `<li :class="classes" class="menu-item"><slot></slot></li>`,
  // 在此声明“注入”activeName值
  inject: ["activeName"],
  props: {
    name: { type: String, required: true }
  },
  computed: {
    classes() {
      const { activeName, name } = this;
      return activeName === name ? "active" : "";
    }
  }
});

Vue.component("Menu", {
  template: `<ul class="menu"><slot></slot></ul>`,
  // 向所有后代组件传递此项
  provide() {
    return {
      activeName: this.activeName
    };
  },
  props: {
    activeName: { type: String, required: true }
  }
});
```

修改后的 Menu、MenuItem 依然可以保持父子关系，互相之间却不强耦合 —— 任何通过 `provide` 提供 `activeName` 属性的组件，都可以作为 MenuItem 的祖先。嵌套 Menu 也可以变得更简单些，我写了另外一个 [demo](https://jsfiddle.net/fanwenjie/987Lfspj/4/)，欢迎查阅，时间关系，不再赘述。

