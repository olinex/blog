---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第六部分 - 部署

## 前提条件

* 已安装 Docker \(1.13+\)
* 已安装 [Docker Compose](https://docs.docker.com/compose/overview/) 
* 已安装 [Docker Machine](https://docs.docker.com/machine/overview/) 
* 已完成 第一部分 的阅读
* 已完成 第二部分 的阅读
* 已将 friendlyhello 镜像上传至仓库. 我们在后续需要使用这个共享镜像
* 已将镜像作为已部署的容器运行
* 与 第五部分 相同的 `docker-compose.yml`

## 简介

你已经有了和第三部分相同的 Compose 文件. 这个文件一样可以在生产环境使用.

## 部署

### 安装社区版本的 Docker 引擎

根据你的系统平台选择你要[安装](https://docs.docker.com/install/#supported-platforms)的 Docker 引擎 

### 创建集群

在节点上运行 `docker swarm init`

### 部署应用

在云主机集群上运行 `docker stack deploy -c docker-compose.yml getstartedlab` 来部署应用.

```bash
docker stack deploy -c docker-compose.yml getstartedlab

Creating network getstartedlab_webnet
Creating service getstartedlab_web
Creating service getstartedlab_visualizer
Creating service getstartedlab_redis
```

现在你的应用已经在云主机上运行了.

### 运行集群命令验证部署情况

当你部署完以后, 你可以通过使用集群命令来浏览和管理集群. 以下是一些你现在很熟悉的一些例子:

* 使用 docker node ls

```bash
docker node ls

ID                            HOSTNAME                                      STATUS              AVAILABILITY        MANAGER STATUS
n2bsny0r2b8fey6013kwnom3m *   ip-172-31-20-217.us-west-1.compute.internal   Ready               Active              Leader
```

You can use the swarm command line, as you’ve done already, to browse and manage the swarm. Here are some examples that should look familiar by now:

