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
