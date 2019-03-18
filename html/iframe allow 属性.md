# iframe allow 属性

总所周知，iframe 用于实现嵌套 html 页面，但并不是简单嵌套，其背后需要遵循一些规则，用以保全安全性。
笔者在工作中就遇到过这样的问题，页面 A 嵌套了页面 B，页面 B 中调用了浏览器的摄像头接口[getUserMedia](https://devdocs.io/dom/mediadevices/getusermedia)，但此时在 chrome 下，页面 B 读取摄像头失败，报错 `DOMExceptionString`。

规范文档: https://w3c.github.io/webappsec-feature-policy/#feature-policy-http-header-field

这是因为摄像头接口受到了 [`Feature Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Feature_Policy#Browser_compatibility) 规则的限制，这是一个实验特性，有一定的兼容性问题。在 MDN 上的解释：

> Feature Policy allows web developers to selectively enable, disable, and modify the behavior of certain features and APIs in the browser. It is similar to Content Security Policy but controls features instead of security behavior.

也就是一种用于控制浏览器特性、API 行为的策略方法，当需要合并第三方网页时，这种特性能有效保证安全性。

FP 支持声明那些源(origin)可以使用那些功能

## 通过 http header 指定

有两种方法指定页面 FP，一是通过 http 头： Feature-Policy，定义格式：

```
Feature-Policy: <directive> <allowlist>
```

directive 指定 FP 策略名，包含`camera`等；allowlist 指定 directive 限定的域名，有几个特殊关键字：\*、self 等

## 通过 iframe allow 属性指定

规则与 header 相似，不过只控制 iframe 嵌套的 document，有两点需要注意：

1. allow 有继承规则，**父级优先级应该更高？** https://developer.mozilla.org/en-US/docs/Web/HTTP/Feature_Policy/Using_Feature_Policy#Inheritance_of_policy_for_embedded_content
2. allow 一经指定即生效，后期变化不会影响行为
