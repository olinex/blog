---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第一部分 - 基础

## 概念定义

Docker 提供给开发人员和系统管理员, 利用容器来开发/部署/运行应用的平台. 使用 Linux 容器来为应用进行部署被称为 **容器化** . 容器并不是一个新技术, 不过它们能更加轻松的部署应用.

容器化越来越流行是因为:

* 灵活 - 即使是那些最复杂的应用都可以被容器包装
* 轻量 - 容器能够共享主机内核
* 可更替 - 你可以动态更新和升级部署
* 可移植 - 你可以在本地构建, 部署在云端, 并且在任何地方运行
* 可扩展 - 你可以增加并自动发布容器副本
* 可堆叠 - 你可以动态堆叠服务

## 镜像和容器

镜像是一种可运行的包, 包含了运行应用所需的所有内容, 代码, 库, 环境变量和配置文件.

容器是通过运行镜像来启动的. 它是镜像的运行实例. 你可以通过 `docker ps` 命令来查看所有运行中的容器.

## 容器和虚拟机

容器在 Linux 系统内本地运行, 并且跟其他容器共享主机的内核. 他们以离散的方式运行进程, 使用的内存不会比其他进程更多.

相比之下, 虚拟机\(VM\)将会运行一个完整的子操作系统, 并通过虚拟机管理程序访问主机资源. 一般来说, 虚拟机会提供给应用不必要的资源.

![&#x6765;&#x6E90;&#x4E8E;&#x5B98;&#x65B9;&#x6587;&#x6863;\(https://docs.docker.com/get-started/\)](../../.gitbook/assets/container-2x.png)

![&#x6765;&#x6E90;&#x4E8E;&#x5B98;&#x65B9;&#x6587;&#x6863;\(https://docs.docker.com/get-started/\)](../../.gitbook/assets/vm-2x.png)

## 准备你的 Docker 环境

* 安装社区版本 - [Community Edition](https://docs.docker.com/install/) \(CE\)
* 安装企业版本 - [Enterprise Edition](https://docs.docker.com/ee/supported-platforms/) \(EE\)

### 检查 Docker 版本

运行 `docker --version` 检查版本:

```bash
docker --version
Docker version 17.12.0-ce, build c97c6d6
```

运行 `docker info` \(或 `docker version` \) 来查看 docker 的更多信息:

```bash
docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.12.0-ce
Storage Driver: overlay2
...
```

{% hint style="info" %}
为了避免产生权限错误, 可以将你的用户加入到 docker 权限组内.
{% endhint %}

## 测试 Docker

通过运行一个简单的 `hello-world` 镜像来检查 docker:

```bash
docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

显示已经下载到你的主机上的 `hello-world` 镜像:

```bash
docker image ls
```

显示 `hello-world` 容器, 它已经打印了信息并退出. 如果容器还在运行, 可以不需要 `--all` 参数:

```bash
docker container ls --all
CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
```

## 结论

容器化可以无缝实现 持续集成/持续交付 \(CI/CD\). 例如:

* 应用不需要系统依赖
* 更新可以推送到分布式应用的任意部分
* 资源的比例可以最大优化

通过 Docker , 扩展你的应用不再需要运行笨重的虚拟机, 只需要运行新的可执行文件.

