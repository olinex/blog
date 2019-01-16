---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 控制运行中的Nginx进程

## 主进程和工作进程

Nginx拥有一个主进程和至少一个工作进程. 如果缓存功能已经开启了, 缓存加载进程和缓存管理进程也将会运行.

主进程的主要作用是读取并加载配置文件, 同时也管理各个工作进程.

工作进程是真正处理请求的进程. Nginx依赖操作系统的机制来有效地在工作进程之间分发请求. 工作进程的数量取决于配置文件nginx.conf 内的 `worker_processes` 指令, 它可以设置为固定的数量或自动匹配CPU的核心数.

## 控制Nginx

为了重新加载你的配置, 你可以停止或重新启动Nginx, 或者发送信号至主进程. 可以通过 nginx -s 命令发送信号.

```bash
nginx -s <SIGNAL>
```

&lt;SIGNAL&gt; 可以为以下选项:

* quit - 从容地关闭
* reload - 重新加载配置文件
* reopen - 重新打开日志文件
* stop - 强制关闭 \(快速关闭\) 

或者使用 `kill` 命令发送信号给主进程. 主进程ID默认记录在nginx.pid文件内, 其一般存储在:

* /usr/local/nginx/logs
* /var/run

