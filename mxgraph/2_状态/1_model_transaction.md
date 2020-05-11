# model 更新会话机制

## 1. 简介

1. 使用 model.beginUpdate 开启；使用 model.endUpdate 结束
2. 内部维护 updateLevel 变量，记录当前会话数
