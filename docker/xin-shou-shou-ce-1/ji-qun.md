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

在 第四部分 , 你将学会如何将应用部署为集群, 并在不同的计算机上运行. 通过

Here in part 4, you deploy this application onto a cluster, running it on multiple machines. Multi-container, multi-machine applications are made possible by joining multiple machines into a “Dockerized” cluster called a **swarm**.

