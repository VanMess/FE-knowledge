# Symbol

## 作为属性名使用

Symbol 可作为属性名使用，特点是不可更改，也就避免了拼写错误

Symbol 是 ES6 中新的原始类型，用于创建必须通过 Symbol 才能引用的属性，比较难以覆盖改写

计算属性名形如：

```javascript
const obj = {
  [propName]: "test"
};
```

Symbol 作为属性名时，最大的意义是保持属性名的不可变。

许多对象属性是不可变更的，比如：

```javascript
class Person {}

Person[Symbol.hasInstance]; // 不可变更
```

但是可以通过 `Object.defineProperty` 函数重新赋值

```javascript
class Person {}

Person[Symbol.hasInstance]; // 不可变更

Object.defineProperty(Person, Symbol.hasInstance, {
  value: function() {
    return false;
  }
});
```

ES6 将很多行为，通过 Symbol 属性开放，被称为 well-know 属性，比如 hasInstance 对应于 instanceof 计算符

通过修改对象的 well-know 属性，可以实现操作符、函数行为重载的效果

拥有 length、数值型属性名属性的对象是很特殊的，比如在 concat 下可以被认为是数组，而进行合并

## JS 元编程接口

- Symbol 中的 well-know 属性
- Proxy
- Object.defineProperty => 不是单纯的赋值，而是属性描述符
- Object.keys/Object.getOwnPropertyNames/Object.getOwnPropertySymbols

## well-known 属性的难点

1.  toPromitive 的用法？
2.  按照书中的描述，合适执行强制类型转换？
