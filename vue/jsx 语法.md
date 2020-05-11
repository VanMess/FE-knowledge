# Template 到 jsx 的语法转换

1. 属性绑定
    - `<div :name="name" />` => `<div name={this.name} />`
    - `<div v-bind:name="name" />` => `<div name={this.name} />`
    - `<div v-bind="options" />` => `<div {...this.options} />`
1. 内容
    - `<div>{{name}}</div>` => `<div>{this.name}</div>`
1. 事件绑定
    - on{event}
    - on={click:function}
1. 过滤器
1. slot
    - 具名 slot
    - 默认 slot
