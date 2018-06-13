# Autocomplete bug 排查记

> iview 升级 2.14.2 后，[AutoComplete](https://www.iviewui.com/components/auto-complete) 组件报了一个莫名其妙的bug。
> 在此记录调试过程

## 环境配置

默认情况下，我们引入iview库的方式为： `import iView from 'iview';`。这里为了方便调试，修改为 `import iView from 'iview/src/index';`。

修改引用后，会报错： 