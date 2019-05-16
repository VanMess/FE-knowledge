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
5. [TLS 握手优化详解](https://imququ.com/post/optimize-tls-handshake.html)

## quetions

1. 握手过程为什么需要三个随机数
   - 为了防止被中间拦截，后续重复请求。这种情况下，中间人虽然无法解码内容，但是后续还是可以不断重复这个请求
2. 什么叫 HTTPS 会话，周期？有两种会话复用技术
   - session_id: 支持 id 的服务器会在服务器保存 id 记录，后续通过 id 标识出会话信息
   - session_ticket: ticket 中包含加密过的会话信息，服务器收到 ticket 后可解密出会话信息，恢复会话，是一种 stateless 技术
3. 握手协商类型：
   - 第一次握手，4 次握手，其中并未加密
   - 基于 session_id 的握手，客户端发送 session_id，服务端通过 session_id 查找会话信息
   - 基于 session_ticket 的握手，服务端可通过解密 session_ticket 计算会话信息
4. 对于同一个域名的 https 连接，tcp 断开后是否需要重新建立？注意跟 http keep-alive 头的关系
5. 客户端如何验证证书合法性
6. 服务端要求提供客户端证书
7. SSL vs TLS？
8. TLS 只是一个握手协议，用于协商客户端、服务端在本次会话过程中使用的秘钥
9. TLS 协议中的信息类型包括：
   - client hello
   - server hello
   - Hello retry request —— 接收 client hello 成功，但缺少必要信息完成此次握手。协议要求，HRR 响应必须携带必要的扩展信息，向服务端说明本次连接还缺乏的信息；对于客户端而言，收到 HRR 会重新构建一份满足需求的 CH 请求，但是，如果新 CH 与旧 CH 内容完全一致，则应抛出异常；如果客户端已经响应了 HRR 后，再次收到 HRR，则应该立即停止此次握手。
10. TLS 中的 cookie，主要有两个作用：
    - 强制客户端指明可到达的网络地址，可用于有效预防 DoS 攻击
    - 用于支持服务端的无状态化
      服务端发送 HRR 响应时，可能会带上上次 CH 生成的 hash 信息
11. TLS 服务器有无状态？如何实现的？TLS 本质上是一个有状态协议，为什么有无状态实现？
12. OpenSSL 证书链？key + CSR = cert？
13. 什么是 master key
14. HelloRetryRequest 是干嘛用的？

## 注意点

1. TLS 不支持重连，所以链接建立后，服务端如果再次受到 client hello 信息，需要返回 `unexpected_message` 异常
2. 协商过程，服务器可能要求客户端多次发送 client hello，
3. TLS 包含三部分协议：handshake、record、alert

## 历史

一开始的时候确实是使用 SSL 协议，后来由于需要更通用、标准的加解密技术，所以 IETF 设计了 TLS 协议。两个协议做的事情都差不多，都是用来实现客户端与服务端之间的握手——它们都不做真实的加解密操作，加解密是交给后面的加密套件做的
