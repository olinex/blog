---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 创建Nginx扩展和配置文件

与其他服务类似, Nginx和其扩展使用特定格式的文本作为配置文件. 一般默认配置文件的名称为nginx.conf, Nginx扩展一般被放置于/etc/nginx文件夹内, 而通过源码安装的Nginx, 存放位置一般根据系统和打包工具的不同有所不同, 常见的情况为:

* /usr/local/nginx/conf
* /etc/nginx
* /usr/local/etc/nginx

## 指令

配置文件通过指令及其参数构成. 简单指令通过分号结尾. 其他指令\(块指令\) 作为容器使用花括号包裹着相关指令, 他们通常被称为"块". 以下是一些简单指令的示例:

```text
user            nobody;
error_log       logs/error.log notice;
work/processes  1;
```

## 功能特定的配置文件

为了使配置更加容易管理, 我们建议你将特定功能的指令切分成不同的文件保存在/etc/nginx/conf.d文件夹内, 并通过 `incldue` 指令将其导入到nginx.conf主配置文件中.

```text
include conf.d/http;
incldue conf.d/stream;
include conf.d/exchange-enhanced;
```

## 环境

一些最顶级的指令, 通常被称之为环境, 他们将不同流量类型的指令组合在一起:

* events - 处理一般的连接
* http - 处理http流量
* mail - 处理邮件流量
* stream - 处理TCP和UDP流量

我们将 指令不在以上环境内的情况称之为 处于主环境内.

## 虚拟服务器

在每个流量处理的环境中, 你可以创建一个或多个`server`指令来定义虚拟服务器用以处理请求. 根据流量类型的不同, 可以放入ser`v`er块内的指令也不同.

对于HTTP流量, 每个`server`指令控制了如何处理来至不同IP地址或域名的流量. 一个或多个`location`指令控制了如何处理不同URL的流量.

对于邮件和TCP/UDP流量\( mail 和 stream 环境 \) `server`指令控制了如何处理来至不同TCP端口或UNIX socket 的流量.

## 多环境的配置示例

```text
user nobody; # 在主环境下的指令

events {
    # 处理一般的连接
}

http {
    # 配置如何处理HTTP并对所有server指令生效
    
    server {
        # 配置如何处理第一个HTTP虚拟服务器
        location /one {
            # 配置如何处理URI以/one开头的请求
        }
        location /two {
            # 配置如何处理URI以/two开头的请求
        }
    }
    
    server {
        # 配置如何处理第二个HTTP虚拟服务器
    }
}

stream {
    # 配置如何处理TCP/UDP并对所有server指令生效
    server {
        # 配置如何处理第一个TCP虚拟服务器
    }
}
```

## 继承

一般情况下, 子环境 \( 一个处于另一个环境内的环境 \) 将会继承父环境的指令配置. 一些指令甚至会在多层环境下生效, 你可以在子环境内重写从父环境继承的指令配置.

