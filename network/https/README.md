# https root

## 握手过程

1. 客户端发送 client hello，包含如下信息：
   1. 客户端支持的 [cipher suites](https://www.ibm.com/support/knowledgecenter/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10700_.htm)
   2. 客户端支持的 supported_groups
   3. 客户端支持的加密算法列表
   4. pre_shared_key？
1. 服务端返回 server hello 包，如果上一步选择的 PSK，则返回 PSK，指明已经选定的 key；否则，返回证书及证书认证信息

第一第二步，协商确定双方支持的参数集

## reference

1. [How SSL and TLS provide identification, authentication, confidentiality, and integrity](https://www.ibm.com/support/knowledgecenter/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10670_.htm)
2. [TLS](https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security)
3. [RFC 8446](https://tools.ietf.org/html/rfc8446)
4. [SSL vs. TLS - What's the Difference?](https://www.globalsign.com/en/blog/ssl-vs-tls-difference/)

## quetions

1. 握手过程为什么需要三个随机数
   - 为了防止被中间拦截，后续重复请求。这种情况下，中间人虽然无法解码内容，但是后续还是可以不断重复这个请求
2. 什么叫 HTTPS 会话，声明周期？
3. 对于同一个域名的 https 连接，tcp 断开后是否需要重新建立？注意跟 http keep-alive 头的关系
4. 客户端如何验证证书合法性
5. 服务端要求提供客户端证书
6. SSL vs TLS？

## 注意点

1. TLS 不支持重连，所以链接建立后，服务端如果再次受到 client hello 信息，需要返回 `unexpected_message` 异常
2. 协商过程，服务器可能要求客户端多次发送 client hello，

## 历史

一开始的时候确实是使用 SSL 协议，后来由于需要更通用、标准的加解密技术，所以 IETF 设计了 TLS 协议。两个协议做的事情都差不多，都是用来实现客户端与服务端之间的握手——它们都不做真实的加解密操作，加解密是交给后面的加密套件做的
