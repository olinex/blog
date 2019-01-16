---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 新手手册

## 前言

Nginx拥有一个主进程和一系列的工作进程. 主进程主要的作用是读取/测试配置, 并维护工作进程. 而工作进程才是真正处理请求的进程. Nginx基于事件模型和操作系统, 能有效地在工作进程之间分配请求. 工作进程的数量取决于配置文件, 或者也可以自动调整为可用 cpu 内核的数量.

Nignx和它的模块的工作方式取决于配置文件, 配置文件的名称默认为nginx.conf, 一般位于以下文件夹内:

* /usr/local/nginx/conf
* /etc/nginx
* /usr/local/etc/nginx

## 启动/停止/重新加载配置文件

为了启动Nginx, 需要运行可执行文件, 一旦Nginx的服务启动了, 它就能通过调用 -s 参数进行操作

```bash
nginx -s 信号
```

信号 主要为以下四种:

1. stop 快速停止
2. quit 可靠地停止
3. reload 重新加载配置文件
4. reopen 重新打开日志文件

例如, 想要等待所有的工作进程将当前的请求处理完成后再停止Nginx服务, 需要执行以下命令:

```bash
nginx -s quit
```

{% hint style="info" %}
执行该命令的用户应该与启动Nginx的用户相同
{% endhint %}

对配置文件的修改并不会立刻生效, 需要重新加载配置文件, 需要执行以下命令:

```text
nginx -s reload
```

当主进程接收到重新加载配置的信号, 主进程会检查新配置文件的语法并尝试启用. 如果启用成功, 主进程会创建新的工作进程并发送消息至旧的工作进程, 要求它们关闭. 如果不成功, 主进程会回滚至旧配置并继续运行. 旧的工作进程接收到关闭命令后, 将停止接收新的请求连接并继续服务当前的请求, 直到所有的当前请求都服务完毕, 旧的工作进程将会关闭.

信号同样可以通过Unix的工具 \(例如 `kill`\) 发送至Nginx进程. 在这种情况下信号将直接通过进程ID发送. Nginx的主进程ID默认记录在nginx.pid文件内, 它通常保存在以下文件夹下:

* /usr/local/nginx/logs
* /var/run

例如, 如果主进程的ID是1628, 要将QUIT信号发送给Nginx, 则需要执行:

```bash
kill -s QUIT 1628
```

要获取所有正在运行中的Nginx进程, 可以通过 `ps` 工具, 例如:

```bash
ps -ax | grep nginx
```

## 配置文件的结构

Nginx由模块组成, 而模块则通过配置文件中的指令所控制. 指令分为简单指令和块指令, 简单指令通过指令名称和参数组成, 指令名称和参数通过空格分割, 通过分号 \(;\) 结尾. 块指令和简单指令具有相同的结构, 不同的是, 块指令通过一系列花括号 \({  }\) 包装的附加指令结尾, 而非分号. 如果块指令可以在花括号内包含其他的指令, 则花括号部分称之为环境.

最外层不被任何其他块包含的称之为主环境. `events`和`https`指令在主环境内, `server`指令在`http`环境内, `location`指令在`server`环境内.

## 静态资源服务

网络服务器其中一个重要的任务是提供对外的静态资源服务 \(例如图片和HTML页面\). 跟着以下的教程, 你会实现一个基于请求获得不同路径资源的服务. 这需要修改配置文件, 在`http`块内的`server`块内增加两个`location`指令.

首先, 打开配置文件. 默认配置文件内已经有了一些最常见的`server`指令示例. 现在让我们在`http`块内新创建一个`server`指令:

```text
http {
    server {
    
    }
}
```

一般情况下, 配置文件内也许会包含一系列的`server`指令, 通过端口和监听的服务名区分. 当Nginx需要确定哪个`server`来处理请求时, 它会检查并对比请求的URI头和`server`块的端口和服务名配置, 并确定`location`指令的地址参数.

在server块内创建一个`location`指令:

```text
http {
    server {
        location / {
            root /data/www;
        }
    }
}
```

这个`location`指令定义了 "/" 前缀, 这个前缀将会与请求的URI对比. 如果匹配, URI的路径将会被拼接到`root`指令的参数后, 在这里, 将会根据请求的路径从本地文件系统查找/data/www内的文件. 如果有多个`location`指令能够匹配, Nginx将会挑选前缀最长的. 示例中的`location`指令提供了最短的前缀, 因此只有当其他的`location`指令匹配失败之后, 才会使用该指令.

接下来, 增加第二个`location`指令:

```text
http {
    server {
        location / {
            root /data/www;
        }
        location /images/ {
            root /data;
        }
    }
}
```

新增的`location`指令将会匹配以/images/为前缀的请求 \( "/" 的`location`指令也匹配这样的请求, 但它的前缀更加短\)

这个配置即将生效, 它会监听标准的 80 端口, 并可以通过本地地址 http://localhost/ 访问. 网络服务器将会从 /data/images 文件夹内匹配文件, 并将其作为响应返回给以 /images/为 URI 前缀的请求. 例如, 对于 URI 为 http://localhost/images/example.png 的请求, Nginx将会发送 /data/images/example.png 文件作为响应, 如果该文件不存在, Nginx将会返回 404 错误响应. 不以 /images/ 为前缀的请求将会被导向/data/www文件夹. 例如, 对于 URI 为 http://localhost/some/example.html 的请求, Nginx将会发送 /data/www/some/example.html 文件作为响应.

为了将以上新配置生效, 启动Nginx或发送`reload`信号至主进程:

```bash
nginx -s reload
```

## 部署一个简单的网络代理服务器

将Nginx作为代理服务器是一个常见的情况, 它会接收请求, 并将请求转发至被代理的服务器, 获取响应后, 再转发至客户端.

我们将会部署一个最基本的代理服务器, 对图片的请求Nginx将会返回本地文件夹的文件, 并将其他所有的请求转发至被代理的服务器. 在这个示例中, 两个服务器都会被定义在一个Nginx实例中.

首先, 通过新增一个server指令到Nginx的配置文件夹内来定义被代理服务器:

```text
server {
    listen 8080;
    root /data/up1;
    
    location / {
    }
}
```

这会是一个监听 8080 端口的简单服务器 \(在先前, 因为默认使用 80 端口, 所以没有配置`listen`指令\), 它将所有的请求导向 /data/up1的本地系统文件夹. 创建该文件夹并将一个 index.html 置于其内. 注意, 将`root`指令置于`server`块内. 当`location`块内没有包含`root`指令时, 将会使用这类`server`块下的`root`指令.

然后, 将先前段落配置文件内的服务器修改为代理服务器. 在第一个`location`指令内, 使用 `proxy_pass`指令指定被代理服务器的协议, 服务名和端口:

```text
server {
    location / {
        proxy_pass http://localhost:8080;
    }
    
    location /images/ {
        root /data;
    }
}
```

修改第二个`location`指令, 为了让其匹配指定的标准文件扩展名, 将其修改为这样:

```text
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

`location`指令的参数是正则表达式, 它会匹配所有以 .gif, .jpg, .png 为结尾的URI的请求. 正则表达式应该以 `~` 为开头. 符合的请求将会被导向 /data/images 文件夹.

当Nginx正在挑选合适的`location`指令为请求服务时, 它首先检查设置了前缀的一系列指令, 并记录最长的匹配`location`指令, 然后检查正则表达式的指令. 如果有任意正则表达式匹配, 则让其接收请求, 否则将使用刚才记录的.

最终代理服务器将会被设置为:

```text
server {
    location / {
        proxy_pass http://localhost:8080/;
    }
    
    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

这个服务器将会筛选以 .gif, .jpg, .png 为结尾的请求并将它们导向/data/images文件夹\( 将URI 添加至 root 指令的参数后 \) 并将所有其他请求转发至先前定义的被代理服务器.

要使新配置生效, 发送`reload`信号给Nginx.

## 部署FastCGI代理

Nginx可以为FastCGI服务器进行路由代理, FastCGI运用于各种框架或编程语言\(如PHP\)中.

最基本的FastCGI服务使用`fastcgi_pass`指令替代`proxy_pass`指令, 同时 `fastcgi_param` 指令可以将参数传递给FastCGI服务器. 假设FastCGI服务可以通过 localhost:9000 访问. 以先前章节的代理配置为基础, 使用 `fastcgi_pass` 替换 proxy\_pass 指令并将替换参数. 在PHP项目内, `SCRIPT_FILENAME` 参数作为脚本名称, `QUERY_STRING` 参数作为请求参数. 配置如下:

```text
server {
    location / {
        fastcgi_pass localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }
    
    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

除了请求静态图片文件以外, 部署的代理服务器会将所有其他的请求通过FastCGI协议路由至localhost:9000.

