---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 配置HTTPS服务

为了配置 HTTPS 服务, `ssl` 参数必须被添加在 `server` 环境的 `listen` 指令下, 并且配置好服务器证书和私钥文件的路径:

```text
server {
    listen                443 ssl;
    server_name           www.example.com;
    ssl_certificate       /path/to/www.example.com.crt;
    ssl_certificate_key   /path/to/www.example.com.key;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers           HIGH:!aNULL:!MD5;
}
```

服务器证书是公共对象, 它会被发送给每个连接这台服务器的客户端. 私钥是私有对象, 它应该被保存在文件内, 并严格限制访问, 当然, 它必须能够被Nginx的主进程读取. 私钥也可以被保存在服务器证书的文件内:

```text
ssl_certificate     www.example.com.cert;
ssl_certificate_key www.example.com.cert;
```

在这种情况下, 文件的读取权限应该被限制. 尽管证书和私钥都被保存在一个文件内, 但只有证书会被发送至客户端.

ssl\_protocls 和 ssl\_ciphers 可以控制连接必须使用指定版本和加密算法的SSL/TLS协议. 默认情况下, Nginx使用 `TLSv1 TLSv1.1 TLSv1.2` 版本和 `ssl_ciphers HIGH:!aNULL:!MD5` 的加密算法, 因此不必显式声明它们. 但要注意, 这些指令的默认值在别的Nginx版本下变化了几次.

## HTTPS服务最优化

ssl 操作会消耗额外的CPU资源. 在多进程系统中, 将会运行多个工作进程, 进程数量不会小于CPU的核心数. 最消耗CPU的操作是SSL握手. 有两种方法可以最小化每个客户端对CPU的消耗:

* 开启长连接, 可以在一个连接内接收多个请求, 通过重用SSL会话参数来避免在并行请求和子请求内多处进行SSL握手. 
* 通过 `ssl_session_cache` 指令, 会话将会被保存在会话共享缓存内, 并被多个工作进程共享. 1MB 约能保存 4000 个会话. 默认缓存会在5分钟后失效. 可以通过 `ssl_session_timeout` 指令来设置缓存大小. 下面是使用10MB来缓存会话的多进程系统的配置:

```text
worker_processes auto;

http {
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         www.example.com;
        keepalive_timeout   70;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
```

## SSL证书链



