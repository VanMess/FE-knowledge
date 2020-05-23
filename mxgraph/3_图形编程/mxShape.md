1. mxShape 是所有图形元素的父类
2. ?子类必须继承 createSvg 方法
3. 创建图元时，调用 `create` 方法
4. 有两种方法自定义图元，一是继承 mxShape；二是通过 xml 定义 mxStenlic
5. mxCellRender ，针对单一图元的渲染器，由上层组件 mxGraphView 实现调度
6. mxCellRender 创建并使用 graph 的环境变量设置 shape
7. container ?

mxEvent.CHANGE -> mxGraph.graphModelChangeListener

## mxCellRender

initializeShape vs createShape vs createIndicatorShape ?

## mxGraphModel

mxGraphModel
mxRootChange: root 节点变更信息？
mxChildChange
mxTerminalChange
mxValueChange
mxStyleChange
mxGeometryChange
mxCollapseChange
mxVisibleChange
mxCellAttributeChange

都是树形结构，用于存储图形文档的状态

## mxCell

一颗独立的树
初始化的时候会生成一个默认父节点，似乎是 group？

mxGraph 1:1 mxGraphModel 1:1 mxCell 1:n mxCell
直接修改 mxCell 值不会触发变更
树形结构，祖先节点、叶子节点分别支持哪些类型的 mxCell；mxCell 中支持更多 mxCell 作为子节点，即使这个 cell 是个 vertex
    似乎并不会限制说，edge或vertex就不能有子节点
    mxCell 状态：单纯的容器、vertex、edge
    vertext: group、html内容、内置图形对象、画笔对象
为什么 mxModel 默认要构造 root[defaultParent] 两级节点？能否在root上增加更多容器？有啥作用？
    root 下一个mxCell就是一个layer
relative 的作用？
