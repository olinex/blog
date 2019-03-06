---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第四部分 - 集群

## 前提条件

* 已安装 Docker \(1.13+\)
* 已安装 [Docker Compose](https://docs.docker.com/compose/overview/) .
* 已安装 [Docker Machine](https://docs.docker.com/machine/overview/) . [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/) 和 [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/) 已经预先安装了, 你可以安心使用. 在 Linux 系统中, 你需要 [手动安装](https://docs.docker.com/machine/install-machine/#installing-machine-directly) . 在 Windows 10 以前的系统, 并且 CPU 没有 Hyper-v 功能, 请使用 [Docker Toolbox](https://docs.docker.com/toolbox/overview/) .
* 已完成 第一部分 的阅读
* 已完成 第二部分 的阅读
* 已将 friendlyhello 镜像上传至仓库. 我们在后续需要使用这个共享镜像
* 已将镜像作为已部署的容器运行
* 与 第三部分 相同的 `docker-compose.yml`

## 简介

在 第四部分 , 你将学会如何将应用部署为集群, 并在不同的计算机上运行. 通过将多个计算机加入到集群, 使得多容器, 多计算机的应用成为可能.

## 集群

群是一组运行 Docker 的计算机集合. 当它们加入集群后, 你可以继续运行 Docker 命令, 但命令将会在 **群管理器** 当中运行. 加入群的计算机可以是物理机或是虚拟机, 每个加入群的计算机都将视为一个 **节点** .

群管理器可以以不同的策略运行容器:

* 空节点 - 仅使用最少数量的计算机来运行容器
* 全局 - 确保每台计算机都拥有指定容器的实例

你可以通过 `docker-compose.yml` 文件来指定集群的策略.

群管理器是集群中唯一能运行你的命令的计算机, 且能对其他将要作为工作机器加入群的计算机进行验证. 工作机器只负责提供服务, 但不能对其他机器进行管理.

现在, 你已经在本地机器上架设了一个单机模式的 Docker . 但是 Docker 运行 集群模式 . 立即启用集群模式会使当前计算机成为群管理器. 从现在开始, Docker 将会在集群中运行命令, 而不是只在当前计算机上.

### 启用集群 <a id="set-up-your-swarm"></a>

群由多个节点组成, 节点可以为物理机或虚拟机. 基本的概念非常简单: 运行 `docker swarm init` 开启群模式, 并将当前的机器作为群管理器, 在其他机器运行 `docker swarm join` , 将它们作为工作机器加入到集群中. 



