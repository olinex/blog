---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 概览

之所以 Docker 容器和服务这么强大, 其中一个很重要的原因是它们能够通过网络互相连接. Docker 容器和服务甚至不需要知道他们部署在 Docker 上, 也不需要知道对方是否工作在 Docker 网络中. 无论你的 Docker 主机是运行在 Linux 上还是 Windows 上, 亦或者是两者皆有, Docker 都可以通过与平台无关的方式管理它们.

这个主题定义了 Docker 网络的基本概念, 并准备设计和部署一个应用程序以充分利用这些功能.

这些内容基本上对所有的 Docker 都有效. 除了一些 [附加功能](https://docs.docker.com/network/#docker-ee-networking-features) 只能用与 Docker EE\(企业版\).

## 主题的范围

本次主题并不会对 Docker 网络如何在系统上工作的细节做深入探讨, 因此你不会找到以下的类似信息:

* Docker 如何在 Linux 系统操作 `iptables` 的规则
* Docker 如何在 Windows Server 系统操作路由规则
* Docker 如何形成和封装数据包或处理加密的信息

你可以查看 [Docker 和 iptables](https://docs.docker.com/network/iptables/) 和 [Docker 参考体系结构: 设计可扩展的便携式 Docker 容器网络](http://success.docker.com/article/networking) 来了解更多深入的技术细节. 

本次主题也不会提供任何关于如何创建/管理/使用网络的教程.

## 网络驱动

Docker 的网络子系统是可通过组件插拔来扩展的. 默认情况下 Docker 提供多个网络驱动, 以提供核心网络功能:

* `bridge` - 网桥 : 默认的网络启动. 如果不明确指定, 将默认以这种驱动创建网络. 这通常用于运行在单例容器内的应用
* host - 

