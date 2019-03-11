---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第五部分 - 栈

## 前提条件

* 已安装 Docker \(1.13+\)
* 已安装 [Docker Compose](https://docs.docker.com/compose/overview/) 
* 已安装 [Docker Machine](https://docs.docker.com/machine/overview/) 
* 已完成 第一部分 的阅读
* 已完成 第二部分 的阅读
* 已将 friendlyhello 镜像上传至仓库. 我们在后续需要使用这个共享镜像
* 已将镜像作为已部署的容器运行
* 与 第三部分 相同的 `docker-compose.yml`
* 已按照 第四部分 教程部署了主机. 可通过 `docker-machine ls` 命令验证
* 已按照 第四部分 教程部署了集群. `docker-machine ssh myvm1 "docker node ls"` 命令校验. 如果集群已经正常运行, 节点的状态均为 `ready` 状态, 否则需要重新初始化集群并将工作主机加入至集群中

## 简介

在 第四部分 , 你学会了如何搭建集群, 它们运行 Docker 并在其上部署应用, 容器在多台主机上协同运行.

在 第五部分 , 你已经到达了分布式应用的最顶层: **栈** . 栈是一组互相关联的应用, 他们共享依赖的环境, 并且可以被同时部署和扩展. 单个栈可以定义和协调整个应用的功能 \(虽然复杂的应用可能需要多个栈\).

在 第三部分 创建 Compose 文件并运行 `docker stack deploy` 的时候, 你已经使用了栈了. 但这个栈只有一个服务运行在一个主机上, 通常这不会在生产环境使用. 现在你可以学习如何构建互相依赖的多个服务并且让他们在不同的主机上运行

## 增加新的服务并重新部署

在 `docker-compose.yml` 文件上增加服务是非常简单的. 首先, 新增一个免费的监控服务, 让我们能够查看堆栈是如何调度容器的.

1. 打来 `docker-compose.yml` 并按如下内容修改. 注意将 username/repo:tag 替换为你的镜像.

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```

文件内添加了一个新的服务, 名叫 `visualizer` . 注意其中两个新的属性: 

* volumes - 使 `visualizer` 服务能够访问 Docker 的主机套接字文件
* placement - 使 `visualizer` 服务只能运行在群控制器内

这是因为这个容器是通过 [Docker 开源项目](https://github.com/ManoMarks/docker-swarm-visualizer) 构建的, 它通过图标展示了 Docker 服务在集群中的运行状况.



