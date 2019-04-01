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

群管理器是集群中唯一能运行你的命令的计算机, 且能对其他将要作为工作主机加入群的计算机进行验证. 工作主机只负责提供服务, 但不能对其他主机进行管理.

现在, 你已经在本地主机上架设了一个单机模式的 Docker . 但是 Docker 运行 集群模式 . 立即启用集群模式会使当前计算机成为群管理器. 从现在开始, Docker 将会在集群中运行命令, 而不是只在当前计算机上.

## 启用集群

群由多个节点组成, 节点可以为物理机或虚拟机. 基本的概念非常简单: 运行 `docker swarm init` 开启群模式, 并将当前的主机作为群管理器, 在其他主机运行 `docker swarm join` , 将它们作为工作主机加入到集群中. 

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

第一个主机将作为管理器, 它会运行管理命令并验证加入到集群的工作主机, 第二个主机是工作主机.

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

通过 `docker-machine ssh` 命令, 发送 `docker swarm join` 命令, 使 `myvm2` 作为工作主机加入到新的集群中:

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

在每个节点上运行 `docker swarm leave` , 你可以重头开始构建集群.

## 在集群内部署应用

以上最麻烦事情的已经解决了. 现在我们只需要重复 第三部分 提及的过程就能在新的集群部署. 记住, 只有 myvm1 会作为群管理器来执行 Docker 命令.

### 配置 docker-machine 终端作为群管理器

到目前为止, 你可以通过 `docker-machine ssh` 命令与虚拟机通信. 另一个选择是运行 `docker-machine env  <machine>` 命令, 来使得当前的终端与虚拟机上的 Docker 守护进程同性. 这个方法更加适用于接下来的操作, 因为它允许你使用本地的 docker-compose.yml 文件来部署应用, 而不需要复制配置文件到任何地方.

输入 `docker-machine env myvm1` ,  终端打印出来的命令会打印配置命令, 复制粘贴最后一行并运行, 使得你的终端能够连接 `myvm1` 虚拟机.

配置命令会根据你的系统的不同而有所不同:

{% tabs %}
{% tab title="Mac, Linux" %}
运行 `docker-machine env myvm1` 来获取配置命令, 

```bash
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```

运行最后一行命令:

```bash
eval $(docker-machine env myvm1)
```

运行 `docker-machine ls` 来验证 `myvm1` 当前是否为当前的机器.  星号表示已经激活.

```bash
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce
```
{% endtab %}

{% tab title="Windows" %}
运行 `docker-machine env myvm1` 来获取配置命令, 

```bash
PS C:\Users\sam\sandbox\get-started> docker-machine env myvm1
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://192.168.203.207:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\sam\.docker\machine\machines\myvm1"
$Env:DOCKER_MACHINE_NAME = "myvm1"
$Env:COMPOSE_CONVERT_WINDOWS_PATHS = "true"
# Run this command to configure your shell:
# & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```

运行最后一行命令:

```bash
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression
```

运行 `docker-machine ls` 来验证 `myvm1` 当前是否为当前的主机.  星号表示已经激活.

```bash
PS C:PATH> docker-machine ls
NAME    ACTIVE   DRIVER   STATE     URL                          SWARM   DOCKER        ERRORS
myvm1   *        hyperv   Running   tcp://192.168.203.207:2376           v17.06.2-ce
myvm2   -        hyperv   Running   tcp://192.168.200.181:2376           v17.06.2-ce
```
{% endtab %}
{% endtabs %}

### 在群管理器内部署应用

现在你已经处于 `myvm1` 的环境中, 你可以像在 第三部分 一样, 行使群管理器的能力, 通过 `docker stack deploy` 命令部署服务, 并使用你本地的 `docker-compose.yml` 文件. 这个命令也许需要几分钟来完成, 并需要等待一会, 部署才能生效. 在群管理器上使用 `docker service ps <service_name>` 命令, 可以验证是否所有的服务都已经被重新部署.

连接到 `myvm1` 是在 `docker-machine` 配置层面上的说法, 你依然可以访问到本地主机的文件. 确保你还在包含了 `docker-compose.yml` 文件的目录下.

运行以下命令将应用部署至 `myvm1` :

```bash
docker stack deploy -c docker-compose.yml getstartedlab
```

现在, 应用已经部署到集群了!

如果你的镜像是保存在私人仓库内, 而不是 Docker Hub, 你需要先通过 `docker login <your-registry>` 命令登录, 并且添加 --with-registry-auth 参数到刚才的命令:

```bash
docker login registry.example.com

docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab
```

通过 WAL 加密日志, 以上命令会将登录密令从本地的客户端发送至部署了服务的群节点当中. 通过这些信息, 节点能够登录到仓库并拉取镜像.

服务已经部署在 myvm1 和 myvm2 上.

```bash
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   gordon/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   gordon/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   gordon/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   gordon/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   gordon/get-started:part2  myvm2  Running
```

为了让你的终端能够和不同的主机通讯, 例如 `myvm2` , 重新运行 `docker-machine env` 命令, 接着运行输出命令的最后一行来指向 `myvm2` . 这只会在当前终端有效. 如果你更换了一个尚未配置的终端, 或者重新打来了一个, 你需要重新运行以上命令. 通过 `docker-machine ls` 命令可以罗列所有的主机, 查看他们都处于什么状态, 获取 IP 地址, 找到正在连接的主机. 查看 [Docker Machine 启动话题](https://docs.docker.com/machine/get-started/#create-a-machine) 学习更多.

或者你可通过 `docker-machine ssh <machine> "<command>"` 来包装 Docker 命令, 这个命令会登录到虚拟机并运行被包装的命令, 但是并不会使用你当前的主机的文件.

在 Mac 和 Linux 系统中, 你可以使用 docker-machine scp &lt;file&gt; &lt;machine&gt;:~ 将文件复制到主机上, 但是 Windows 系统则必须使用 Linux 仿真终端才能使用, 例如 [Git Bash](https://git-for-windows.github.io/) .

## 访问集群

你可以通过 IP 地址访问 `myvm1` 或 `myvm2` 上的应用.

你创建的网络会被共享并且负载均衡. 运行 `docker-machine ls` 查看虚拟机的 IP 地址并通过浏览器访问.

![&#x6765;&#x6E90;&#x4E8E;&#x5B98;&#x65B9;&#x6587;&#x6863;\(https://docs.docker.com/get-started/part4/\)](../../.gitbook/assets/app-in-browser-swarm.png)

浏览器上会循环显示五个随机的容器的 ID .

两个 IP 地址都能访问, 是因为集群中的节点都加入了入口路由网格. 这可确保部署在集群中的服务始终保留该端口, 而不管实际运行容器的是什么. 下面的示意图展示了三个节点的集群, 是如何将一个名叫 `my-web` 的服务通过路由网格发布在 `8080` 端口上的:

![&#x6765;&#x6E90;&#x4E8E;&#x5B98;&#x65B9;&#x6587;&#x6863;\(https://docs.docker.com/get-started/part4/\)](../../.gitbook/assets/ingress-routing-mesh.png)

{% hint style="info" %}
连接有问题?

要在集群中使用入口网络, 你需要在开启集群模式之前, 将节点的以下端口开放给彼此:

端口 7946 TCP/UDP 用于容器的网络发现

端口 4789 UDP 用于容器的入口网络
{% endhint %}

## 迭代并扩展你的应用

从现在开始你可以做任何你在 第二部分 和 第三部分 学到的操作.

通过修改 `docker-compose.yml` 文件扩展应用.

通过修改代码来改变应用的行为, 重新构建并且上传新的镜像文件. 

重新运行 docker stack deploy 命令来使得以上变更生效.

你可以通过 `docker swarm join` 命令加入任何的主机到集群中, 无论是物理的还是虚拟的, 来提升集群的性能. 只需要执行 `docker stack deploy` 命令, 你的应用会自动利用新的资源.

## 清除和重启

### 栈和集群

你可以通过 docker stack rm 命令移除栈:

```bash
docker stack rm getstartedlab
```

接下来, 如果你想移除集群: 

* 工作节点上执行 `docker-machine ssh myvm2 "docker swarm leave"`
* 在群管理器上执行 `docker-machine ssh myvm1 "docker swarm leave --force"`

在 第五部分 我们仍需要用到这个集群, 所以现在请不要移除它.

### 清除 docker-machine 终端的参数设置

你可以通过以下命名清楚当前终端设置的 Docker 环境变量:

{% tabs %}
{% tab title="Mac, Linux" %}
```bash
eval $(docker-machine env -u)
```
{% endtab %}

{% tab title="Windows" %}
```bash
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env -u | Invoke-Expression
```
{% endtab %}
{% endtabs %}

这会将终端和虚拟机断连, 并且你可以使用当前终端. 要了解更多, 查看 [清除环境变量](https://docs.docker.com/machine/get-started/#unset-environment-variables-in-the-current-shell) .

### 重启 Docker 主机

如果你重启了本地主机, Docker 主机服务将会停止. 你可以通过 `docker-machine ls` 命令查看主机的状态:

```bash
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```

重启指定的主机:

```bash
docker-machine start <machine-name>
```

例如:

```bash
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```

