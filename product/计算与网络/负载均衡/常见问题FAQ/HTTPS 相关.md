<span id="1"></span>
**HTTP 相关问题**
- [HTTPS 支持的加密套件有哪些？](#1)
- [HTTPS 支持哪些版本的SSL/TLS安全协议？](#2)
- [HTTPS 监听使用什么端口？](#3)
- [为什么需要 HTTPS 双向认证？](#4)
- [为什么 HTTPS 协议实际产生的流量会比账单流量多一些？](#5)
- [添加 HTTPS 监听器后，负载均衡到后端云服务器间的请求是否依然通过 HTTP 协议传输？](#6)

**证书问题**
- [CLB 目前支持哪些类型的证书？](#7)
- [一个监听器可以绑定多少个 HTTPS 证书？](#8)
- [一个证书可以应用于多少个负载均衡器，多少个监听器？](#9)
- [如何上传证书？](#10)
- [证书区分地域吗？](#11)
- [证书需要上传到后端 CVM 吗？](#12)
- [证书过期后如何处理？](#13)
- [添加证书报错如何处理？](#14)


### HTTPS 支持的加密套件有哪些？
ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-CHACHA20-POLY1305:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128:AES256:AES:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK

[[回到顶部]](#1)

[](id:2)
### HTTPS 支持哪些版本的SSL/TLS安全协议？
负载均衡 HTTPS 目前支持的 ssl_protocols：TLSv1、TLSv1.1、TLSv1.2。

[[回到顶部]](#1)


[](id:3)
### HTTPS 监听使用什么端口？
不强制，建议使用443端口。

[[回到顶部]](#1)

[](id:4)
### 为什么需要 HTTPS 双向认证？
有些客户对数据安全要求较高，如涉及到金融服务的客户等。他们不仅需要在服务端进行 HTTPS 认证，在客户端也需要进行 HTTPS 认证，为了满足这些客户的需求，我们推出 HTTPS 双向认证功能。

[[回到顶部]](#1)

[](id:5)
### 为什么 HTTPS 协议实际产生的流量会比账单流量多一些？
如果用户使用 HTTPS 协议，将会使用一些流量用于协议握手，因此其实际产生的流量会比账单流量更多一些。

[[回到顶部]](#1)

[](id:6)
### 添加 HTTPS 监听器后，负载均衡到后端云服务器间的请求是否依然通过 HTTP 协议传输？
是的。添加 HTTPS 监听器后，客户端到负载均衡之间的请求将经过HTTPS协议加密，而负载均衡到后端云服务器依然通过HTTP协议传输，因此后端云服务器无需做SSL配置。

[[回到顶部]](#1)

[](id:7)
### CLB 目前支持哪些类型的证书？
目前支持服务器证书和 CA 证书的上传，服务器证书需要上传证书内容和私钥，CA 证书只需要上传证书内容；这两种类型的证书都只支持 PEM 编码格式的上传。

[[回到顶部]](#1)

[](id:8)
### 一个监听器可以绑定多少个 HTTPS 证书？
如果用户使用 HTTPS 单向认证，则一个监听只能绑定一个服务器证书；若用户使用 HTTPS 双向认证，则一个监听需要绑定一个服务器证书＋一个 CA 证书。

[[回到顶部]](#1)

[](id:9)
### 一个证书可以应用于多少个负载均衡器，多少个监听器？
一个证书可以应用于一个或多个负载均衡器，或多个监听器。

[[回到顶部]](#1)

[](id:10)
### 如何上传证书？
可以通过 API 或负载均衡控制台两种方式上传。

[[回到顶部]](#1)

[](id:11)
### 证书区分地域吗？
区分。考虑到安全和性能，目前用户的证书如需要在多个地域使用，就需要在多个地域上传。

[[回到顶部]](#1)

[](id:12)
### 证书需要上传到后端 CVM 吗？
不需要，负载均衡 HTTPS 提供证书管理系统管理和存储用户证书，证书不需要上传到后端 CVM，用户上传到证书管理系统的私钥都会加密存储。

[[回到顶部]](#1)

[](id:13)
### 证书过期后如何处理？
当前证书过期后，需要用户手动更新证书。

[[回到顶部]](#1)

[](id:14)
### 添加证书报错如何处理？
可能是私钥内容错误，需要用户替换为新的满足需求的证书。

[[回到顶部]](#1)
