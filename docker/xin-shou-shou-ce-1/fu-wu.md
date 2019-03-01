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

在运行 `docker stack deploy` 命令之前, 我们先要运行: 

```bash
docker swarm init
```

{% hint style="info" %}
这个命令将在 第四部分 讲解. 如果你不先运行该命令, 你会获得一个 "this node is not a swarm manager" 的错误
{% endhint %}

现在让我们运行部署命令, 你需要为应用命名. 在这里, 命名为 getstartedlab :

```bash
docker stack deploy -c docker-compose.yml getstartedlab
```

现在每个服务栈会运行五个容器实例, 容器包装了我们部署的镜像.

通过以下命令获取服务的ID:

```bash
docker service ls
```

查看 `web` 服务的输出, 其包含了应用的名称作为前缀. 如果应用如同示例一样的命名, 服务的名称为 `getstartedlab_web` . 同时显示的还有容器副本的数量, 镜像的名称和暴露的接口.

或者你可以运行 `docker stack services` 命令, 并以栈名作为参数. 下面示例的命令可以产看所有与栈 `getstartedlab` 相关的服务:

```bash
docker stack services getstartedlab
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
bqpve1djnk0x        getstartedlab_web   replicated          5/5                 username/repo:tag   *:4000->80/tcp
```

 在服务内, 每个运行的容器被称之为 **任务** . 任务被赋予了唯一的数字id, 最多为 `docker-compose.yml` 内定义的副本数量.

罗列服务内所有的任务:

```bash
docker service ps getstartedlab_web
```

任务还可以通过罗列系统上的容器命令来查看, 并不按照服务来筛选:

```text
docker container ls -q
```

在一行内运行 `curl -4 http://localhost:4000` 多次或在浏览器上刷新访问这个 URL 多次.

![&#x6765;&#x6E90;&#x4E8E;&#x5B98;&#x65B9;&#x6587;&#x6863;\(https://docs.docker.com/get-started/part3/\)](../../.gitbook/assets/app80-in-browser.png)

无论哪种访问方式, 容器 ID 都会发生改变, 从而证明服务已经开启了负载均衡; 对于每个请求, 5个任务会通过轮询调度的方式, 选取其中一个任务来处理请求.

为了查看栈内的所有任务, 运行 docker stack ps 以及指定应用名称, 将会打印以下内容:

```bash
docker stack ps getstartedlab
ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
uwiaw67sc0eh        getstartedlab_web.1   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
sk50xbhmcae7        getstartedlab_web.2   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
c4uuw5i6h02j        getstartedlab_web.3   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
0dyb70ixu25s        getstartedlab_web.4   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
aocrb88ap8b0        getstartedlab_web.5   username/repo:tag   docker-desktop      Running    
```

{% hint style="info" %}
Windows 10 的 PowerShell 已经包含了 curl 命令, 你还可以通过类似 [Git Bash](https://git-for-windows.github.io/) 或下载 [wget for Windows](http://gnuwin32.sourceforge.net/packages/wget.htm) 工具
{% endhint %}

{% hint style="info" %}
根据你环境的网络配置, 容器对于 HTTP 请求的响应时间有可能超过 30 秒. 这并不代表 Docker 或 集群的性能, 而是因为我们尚未讨论到的 Redis 依赖, 我们在本章后续会提及. 
{% endhint %}

## 扩展应用

你可以通过修改 docker-compose.yml 的 replicas 参数扩展应用, 保存并且重新执行以下命令:

```bash
docker stack deploy -c docker-compose.yml getstartedlab
```

Docker 将执行本地更新, 不需要关闭服务栈或者杀死任何容器.

重新运行 docker container ls -q 来查看按照新配置部署的实例. 如果增加了副本, 将会有更多的任务从而启动更多的容器.

## 注销应用和集群

### 注销应用

```text
docker stack rm getstartedlab
```

### 注销集群

```text
docker swarm leave --force
```

就跟通过 Docker 构建和扩展应用一样, 注销操作非常简单. 您在学习如何在生产中运行容器方面迈出了一大步。接下来, 您将学习如何在 Docker 计算机群集上, 真正地运行此应用程序的集群.

{% hint style="info" %}
构建文件定义了 Docker 的应用, 可以上传至 [Docker Cloud](https://docs.docker.com/docker-cloud/) , 或者使用 [Docker 企业版](https://www.docker.com/enterprise-edition) , 可以上传到任何硬件或者云厂商.
{% endhint %}

