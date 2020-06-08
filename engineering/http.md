# HTTP学习笔记

### HTTP历史版本

* HTTP/0.9 只支持GET方法，不支持请求头
* HTTP/1.0 基本成型，支持富文本、Header、状态码、缓存等
* HTTP/1.1 主流。支持连接复用、分块发送
* SPDY  HTTP/2的前身
* QUIC  第三代协议，基于UDP实现TCP+HTTP/2并优化
* HTTP/2 第二代协议，多路复用、头部压缩、服务器推送等
* QUIC 更名为HTTP/3



> QUIC （Quick UDP Internet Connection），基于UDP的高效和快速，整合了TCP、TLS和HTTP/2的优点，并加以优化。
>
> 运行在QUIC上的HTTP （HTTP over QUIC）则称为HTTP/3 。



### QUIC的优势

相比于HTTP/2，QUIC能够做到零RTT建立连接，也就是客户端发给服务端第一个包里面就含有数据了，而HTTP/2则需要2到3RTT才能发送数据。

https://zhuanlan.zhihu.com/p/143464334



​	