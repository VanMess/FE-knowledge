# [node-config](https://github.com/lorenwest/node-config/wiki/Configuration-Files) 应用全解析

## agenda

1.  层叠配置结构：
    1.  default 文件
    2.  environment 变量
    3.  host 变量
    4.  instance 变量
    5.  local
    6.  环境变量
2.  基于文件
    1.  支持多种文件类型
    2.  我们需要关注的就是写各种文件 + 用不同的环境变量去启动
3.  在前端使用 node-config
4.  官方声称是 immutable，但实际测试，数组是可以变更的

## 导言
