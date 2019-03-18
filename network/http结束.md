在一些情况下，浏览器会判定 response 结束：

1. content-length 指明的值已经接收完了，此时即使 tcp 连接没有断开，浏览器也会判定 response 结束了
2. content-length 指明的值还没接收完，TCP 连接未中断时，浏览器会一直等待
3. content-length 指明的值还没接收完，但 TCP 连接已经断开，浏览器会认为响应内容有误，报错：ERR_CONTENT_LENGTH_MISMATCH
