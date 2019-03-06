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

## 启用集群

群由多个节点组成, 节点可以为物理机或虚拟机. 基本的概念非常简单: 运行 `docker swarm init` 开启群模式, 并将当前的机器作为群管理器, 在其他机器运行 `docker swarm join` , 将它们作为工作机器加入到集群中. 

### 创建虚拟机

{% tabs %}
{% tab title="本地虚拟机\(Mac/Linux/Windows 7/Windows 8\)" %}
你需要一个虚拟机管理程序来创建虚拟机, 我们使用 [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads) .

{% hint style="info" %}
如果你的Windows系统包含了 Hyper-V 功能, 例如 Windows 10, 就不需要安装 VirtualBox . 如果你已经安装了 [Docker Toolbox](https://docs.docker.com/toolbox/overview/) , VirtualBox 就已经安装了.
{% endhint %}

现在, 通过 `docker-machine` 创建两个虚拟机:

```bash
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```
{% endtab %}

{% tab title="本地虚拟机\(Windows 10/Hyper-V\)" %}
快速创建一个虚拟交换机, 供虚拟机使用, 以方便他们连接彼此.

1. 打开 Hyper-V Manager
2. 点击右边菜单栏的 **虚拟交换机管理器** \(Virtual Switch Manager\)
3. 点击 **创建虚拟交换机** 并选择类型为 **扩展**
4. 取名为 `myswitch` , 选择共享主机的可用网络适配器

现在, 通过 `docker-machine` 创建两个虚拟机:

```bash
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm2
```
{% endtab %}
{% endtabs %}

#### 显示虚拟机并获取他们的IP地址

现在已经创建了两个虚拟机, `myvm1` 和 `myvm2` .

打印所有的计算机并获取他们的IP地址:

```bash
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce
```

#### 初始化集群并增加节点

第一个机器将作为管理器, 它会运行管理命令并验证加入到集群的工作机器, 第二个机器是工作机器.

你可以通过 docker-machine ssh 命令来向你的虚拟机发送命令. 通过 `docker swarm init` 命令指定 `myvm1` 为群管理器:

```bash
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join --token <token> <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

务必在运行 docker swarm init 和 docker swarm join 的时候, 使用 2377 接口 \(群管理接口\) , 或不指定接口来使用默认接口.

通过 docker-machine ls 命令返回的IP地址包括了 2376 接口, 这是 Docker 守护进程接口, 请不要使用该接口, 否则会[报错](https://forums.docker.com/t/docker-swarm-join-with-virtualbox-connection-error-13-bad-certificate/31392/2)

Docker 允许你 使用系统自身的SSH , 如果因为一些原因不能通过群管理器发送命令, 可以在运行 ssh 命令时使用 `--native-ssh` :

```bash
docker-machine --native-ssh ssh myvm1 ...
```

通过 `docker-machine ssh` 命令, 发送 `docker swarm join` 命令, 使 myvm2 作为工作机器加入到新的集群中:

```bash
$ docker-machine ssh myvm2 "docker swarm join --token <token> <ip>:2377"

This node joined a swarm as a worker.
```

你现在已经创建了第一个集群!

运行 `docker node ls` 命令来查看已经加入了这个集群的节点:

```bash
$ docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

#### 离开集群

在每个节点上运行 `docker swarm leave` , 你可以重头开始构建集群

