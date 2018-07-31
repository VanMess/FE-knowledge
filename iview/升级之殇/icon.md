# iview 升级指南 —— Icon 篇

> iview 在今年 7 月 28 号发布了 3.0.0 版本，大版本升级往往意味着功能、接口的大变更。
> 虽然官网已经有长长的[更新日志](https://www.iviewui.com/docs/guide/update)，但看起来还是有些抽象了，
> 所以我决定做个新旧版本的比较，盘点新版本到底为我们带来了什么新特性。
>
> 这是系列文章的第一篇，讲的是最简单的组件 —— Icon，希望能给大家带来帮助

## 结论

新版本 Icon 有如下变化点：

1.  新版本的 Icon 组件支持更多图标类型
2.  新旧版本的图标名有些差异，升级时务必注意
3.  Icon 组件支持自定义图标，可通过 `custom` 属性传递类名
4.  Button、Avatar、Rate 组件也支持自定义图标，可通过 `custom-icon` 属性传递类名

有时间的朋友，也欢迎看看下面的详细解读：

## 基础升级

新版 [Icon 组件](https://www.iviewui.com/components/icon) 最大的变化，是升级图标库 [ionicons](https://ionicons.com/) 到 3.x 版本，可用的图标类型从 730 增加至 866。奇怪的是，目前[ionicons](https://ionicons.com/)提供的版本已经是 4.2.5 iview 却只用了其 3.x 版本。

升级后的图标库，功能更强大了，但却为旧版本升级带来了一个坑：

![官网alert](../../assets/icon.md/2018-07-29-17-05-30.png)

具体是那些图标名称发生变化了呢？官网没有明说，[ionicons](https://ionicons.com/)也没有明说，找来找去也没找着可信的说明，建议大家在升级的时候仔细测试所有 Icon 调用。

## 支持自定义图标

除了[ionicons](https://ionicons.com/)库的变化之外，新版 Icon 还支持 **通过 `custom` 传入图标 class 名，实现自定义图标功能**，举个例子：

```html
<Icon custom="fa fa-user" />
<!-- 等同于： -->
<i class="ivu-icon fa fa-user">
```

这真是一个很方便的功能，因为 iview 提供的图标是不可能覆盖所有应用场景的，实际开发中一般都会自行引入其他图标库，在旧版本中引入的图标库与 iview 之间是割裂的，没法复用 icon 的行为逻辑，比如 `Button` 中图标的 loading 效果。
在新版本中终于可以大胆使用自定义图标了，比如 [下面的例子](https://jsfiddle.net/1gxuwney/7/)，我在 `Button` 组件中使用 font-awesome 的 `fa-user` 图标，但在 loading 态中，还是会保留原来的转菊花效果。

```html
<div id="app">
  <i-button custom-icon="fa fa-user">Custom icon</i-button>
  <i-button custom-icon="fa fa-user" :loading="true">
    Loading effect
  </i-button>
</div>
```

尴尬的是，目前仅有 [`Button`](https://www.iviewui.com/components/button)、[`Avatar`](https://www.iviewui.com/components/avatar)、[`Rate`](https://www.iviewui.com/components/rate) 三个组件支持 `customIcon` 属性，其他组件，诸如 `Tab`、`Input`、`Alert` 等尚不支持，官方也没有给出明确的计划，所以也不好揣测。

## 代码

新旧版本 Icon 组件代码差别不大，我将差异点抽出来：

```html
<script>
    export default {
        props: {
            ...
            custom: {
                type: String,
                default: ''
            }
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-${this.type}`]: this.type !== '',
                        [`${this.custom}`]: this.custom !== '',
                    }
                ];
            }
            ...
        }
    };
</script>
```

可以看到，区别有两点，一是支持 `custom` 属性；二是基于 `type`、`custom` 两个 props 计算 `classes` 值。Icon 组件很简单，这里我唯一觉得有问题的地方是： 没有对 type 进行必要的校验！
既然 type 属性只能传入 ionicons 支持的图标，为什么不做个 in 校验呢？为了性能？新版的 ionicons 有 866 图标，确实可能会影响一丢丢性能，但至少可以在 `process.env.NODE_ENV ==='development'` 环境下做校验呀，多多少少也是可以挡住一些问题。
