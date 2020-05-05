# Template 到 jsx 的语法转换

1. 属性绑定
   - `<div :name="name" />` => `<div name={this.name} />`
   - `<div v-bind:name="name" />` => `<div name={this.name} />`
   - `<div v-bind="options" />` => `<div {...this.options} />`
1. 内容
   - `<div>{{name}}</div>` => `<div>{this.name}</div>`
