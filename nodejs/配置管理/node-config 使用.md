# [node-config](https://github.com/lorenwest/node-config/wiki/Configuration-Files) 应用笔记

## 碎碎念笔记

1.  可通过 `process.env["NODE_CONFIG_DIR"]` 配置配置文件夹的路径，[默认是 `process.cwd`](https://github.com/lorenwest/node-config/blob/master/lib/config.js#L668)
2.  有三种类型的配置变量：
    1.  `development` —— 通过 `NODE_ENV`、`NODE_CONFIG_ENV` 指定
    2.  `hostname` —— 通过 `$HOST`、`$HOSTNAME` 变量，或 `os.hostname()` 指定
    3.  `instance` —— 通过 `NODE_APP_INSTANCE` 指定实例名，实例名并不要求必须是数字。[详细解释](https://github.com/lorenwest/node-config/wiki/Configuration-Files#multi-instance-deployments)
3.  敏感信息方案 —— [local 文件](https://github.com/lorenwest/node-config/wiki/Configuration-Files#local-files)
4.  环境变量是最高的
5.  支持多种格式的文件，并且可混合使用
6.  支持异步获取的值，但必须使用 `require('config/defer').deferConfig` 函数
7.  支持配置复杂的值，比如 `stream`、`Promise` ，可通过 `require('config/raw').raw` 实现
8.  `config.get('key')` 中的 key 为 undefined 时会报异常
9.  `process.env.NODE_ENV` 并不在 config 的管理范畴
10. config 生产的内容默认是 immutable 的，可以通过 [`ALLOW_CONFIG_MUTATIONS`](https://github.com/lorenwest/node-config/wiki/Environment-Variables#allow_config_mutations) 环境变量变更
11. 可以通过 `config/custom-environment-variables.json` 文件定义一些需要从环境变量中获取的配置值，这些配置同样适用于属性值的覆盖机制，这是最高级别的覆盖
12. 有两种方法获取配置值：get 函数、对象语法，get 函数在获取不存在的值时，会报异常，对象语法不会。无论哪种语法，都需注意避免使用保留字：get/has/util，否则会覆盖原来的功能函数

NODE_APP_INSTANCE 参数配合 pm2： 

```javascript
module.exports = {
  apps: [
    // First application
    {
      name: "instance-1",
      script: "test-instance.js",
      env: {
        NODE_ENV: "development",
        NODE_APP_INSTANCE: 1
      }
    },

    // Second application
    {
      name: "instance-2",
      script: "test-instance.js",
      env: {
        NODE_ENV: "development",
        NODE_APP_INSTANCE: 2
      }
    }
  ]
};
```

如果要按 CPU 数量分配实例:

```javascript
const os = require("os");

const apps = os.cpus().map(function(o, i) {
  return {
    name: "instance-" + i,
    script: "test-instance.js",
    env: {
      NODE_ENV: "development",
      NODE_APP_INSTANCE: i
    }
  };
});

module.exports = {
  apps: apps
};
```

但这种做法其实很不实用，没办法应用 [`cluster`](https://nodejs.org/api/cluster.html#cluster_how_it_works) 模式的各种优点

immutable 特性是基于以下代码实现：

```javascript
Object.defineProperty(object, property, {
  value: typeof value === "undefined" ? object[property] : value,
  writable: false,
  configurable: false
});
```

所以在尝试写的时候，会报异常

## 官方提供的集成方案：

1. [加密](https://github.com/lorenwest/node-config/wiki/Securing-Production-Config-Files)
2. [webpack](https://github.com/lorenwest/node-config/wiki/Webpack-Usage)，有四种方法

## 问题

1.  如何启动多实例机制 —— 通过 `NODE_APP_INSTANCE` 变量指定，跟 cluster 没有很好的合作模式
2.  如何管理敏感信息？
3.  数组型配置信息的 immutable

## 总结

1.  node-config 是一个功能完备的配置管理库，提供了各种级别的配置文件体系
2.  支持配置合并
3.  支持按 node_env 变量切换不同的配置
4.  支持按 host 变量切换不同配置
5.  支持按实例切换配置
6.  支持异步配置
7.  支持多种类型的配置文件：json、json5、hjson、yaml、js、ts

## 竞品

1.  [node-convict](https://github.com/mozilla/node-convict)
1.  [nconf](https://github.com/indexzero/nconf)
1.  [rc](https://github.com/dominictarr/rc)
1.  [node-figc](https://github.com/substack/node-figc)
