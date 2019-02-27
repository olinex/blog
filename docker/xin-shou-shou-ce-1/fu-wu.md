---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第三部分 - 服务

## 前提条件

* 已安装 Docker \(1.13+\)
* 已安装 [Docker Compose](https://docs.docker.com/compose/overview/). [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/) 和 [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/) 已经预先安装了, 你可以安心使用. 在 Linux 系统中, 你需要 [手动安装](https://github.com/docker/compose/releases) . 在 Windows 10 以前的系统, 并且 CPU 没有 Hyper-v 功能, 请使用 [Docker Toolbox](https://docs.docker.com/toolbox/overview/) .
* 已完成 第一部分 的阅读
* 已成 第二部分 的阅读
* 已将 friendlyhello 镜像上传至仓库. 我们在后续需要使用这个共享镜像
* 已将镜像作为已部署的容器运行.

## 简介

在第三部分, 我们将扩展我们的分布式应用并且开启负载均衡的功能. 为了实现这些功能, 我们必须在应用的层次关系上更进一步, 进入 **服务** 层面.

* 栈
* 服务\(我们现在在这里\)
* 容器\(在 第二部分 已经介绍\)

## 关于服务

在分布式应用中, 应用的每个部分都被称之为 "服务" . 例如, 想象你有一个视频共享网站, 它很可能包含一个保存应用数据的服务; 一个在用户上传文件后, 在后台对视频进行转码的服务; 一个提供前端页面的服务, 诸如此类.

一个服务只会运行一个镜像, 不过服务编译了镜像的运行方式, 如哪些接口会被使用, 容器应该运行多少个副本, 使得服务有足够的容量, 诸如此类. 扩展服务会改变容器实例的数量, 使得服务的进程有更多的计算资源.

幸运的是, 对于 Docker 平台来说, 定义, 运行和扩展服务是非常容易的, 只需要编写 `docker-compose.yml` 文件.

## 你的第一个 docker-compose.yml 文件

`docker-compose.yml` 文件定义了 Docker 容器在生产环境的行为

在你期望的位置保存以下文件为 `docker-compose.yml` . 确认你已经按照 第二部分 的教程将镜像文件上传至仓库, 并将 `username/repo:tag` 替换为你的镜像.

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

docker-compose.yml 文件告诉 Docker 执行以下操作:

* 从仓库拉取 第二部分 我们上传镜像
* 运行镜像的五个容器实例作为服务, 并命名为 `web` , 限制每个实例最多只能使用一个 CPU 核心 10% 的时间, 以及 50MB 的内存
* 如果实例启动失败, 则重启容器
* 将主机的 4000 端口与 `web` 服务的 80 端口建立映射
* 通过 负载均衡网络 `webnet` 使 `web` 的容器集合共享 80 端口\(在内部, 容器集合通过临时接口与 `web` 服务的 80 端口联通\)
* 按照默认配置定义一个网络, 并命名为 webnet \(负载均衡网络\)

## 运行负载均衡应用



