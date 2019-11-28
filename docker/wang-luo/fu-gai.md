---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 覆盖

覆盖网络驱动会在多个 Docker 守护主机上创建一个子网络. 这个网络位于\(覆盖\)主机的特定网络之上, 允许连接到网络的容器 \(包括集群的服务容器\) 都能安全的通信. Docker 将会透明地处理每个包的路由, 从 Docker 守护主机到指定的目标容器.

当你初始化或者加入一个集群的时候, Docker 主机上会创建两个新的网络:

* 一个名为 `ingress` 的覆盖网络, 它控制了集群服务的数据流量. 当你创建了一个集群服务但并没有将其连接到任何一个自定义覆盖网络时, 它会自动加入 `ingress` 网络.
* 一个名为 docker\_gwbridege 的网桥网络, 他连接了处于同一个集群内各自独立的 Docker 守护进程.

你可以通过 `docker network create` 命令创建一个自定义覆盖网络, 同样的方式你可以创建一个自定义网桥网络. 服务或容器可以同时连接多个网络, 但是服务或容器只能通过彼此都加入的网络来通信.

尽管你可以将集群的服务和独立的容器都加入同一个覆盖网络, 但他们的默认行为和配置方式是不相同的. 因此, 本主题的其余部分被划分为:

1. 适用于所有覆盖网络的操作
2. 只是用于集群服务的操作
3. 只是用于独立容器的操作

## 适用于所有覆盖网络的操作

### 创建覆盖网络

#### 先决条件

* 你需要为覆盖网络内每个 Docker 主机之间打开以下端口, 以保持通信状态:
  * 2377/tcp - 用于集群管理通信
  * 7946/tcp 7946/udp - 用于节点通信
  * 4789/udp - 用于覆盖网络流量
* 在创建覆盖网络之前, 你需要通过 `docker swarm init` 命令将 Docker 守护进程初始化为集群管理器, 或者通过 `docker swarm join` 命令将 Docker 守护进程加入集群. 以上方法均会创建一个默认的覆盖网络 `ingress` , 集群服务会默认使用它. 即使你完全不打算使用它, 你也需要执行以上操作. 接下来, 你就可以创建自定义的覆盖网络了.

要创建能够用于集群服务的覆盖网络, 使用以下命令:

```bash
docker network create -d overlay my-overlay
```

使用 `--attachable` 参数, 可以要创建能够用于集群服务的覆盖网络, 也可以创建能够用于两个不同 Docker 守护进程下的独立容器的覆盖网络:

```bash
docker network create -d overlay --attachable my-attachable-overlay
```

你可以设置特定的 IP 地址范围, 子网罗, 网关或其他选项, 通过 `docker network create --help` 查看更多.

### 在覆盖网络内加密流量

所有的集群服务管理流量都会默认地被 AES-GCM 模式加密. 集群内的管理节点会每12小时更新一次加密用的密钥.

为了加密应用数据, 可以在创建网络的时候使用 `--opt encrypted` 参数. 这会使用 vxlan 等级的 IPSEC 加密. 但这个加密会对性能造成不可忽视的影响, 因此你需要对其进行测试, 才能在生产环境中使用.

当你在覆盖网络中使用加密时, 在服务周期性访问覆盖网络的节点上, Docker 会创建 IPSEC 通道. 这些通道同样会使用 AES-GCM 模式来加密, 并且管理服务器会每12小时自动刷新密钥.

{% hint style="danger" %}
不要在访问 Windows 节点的时候使用加密的覆盖网络

加密覆盖网络并不支持 Windows. 如果 Windows 节点连接到了加密的覆盖网络, 并不会发生错误, 但节点并不能通过网络通信.
{% endhint %}

### 自定义默认入口网络

大部分用户永远不需要配置 `ingress` 网络, 不过 17.05+ 的 Docker 允许你这么做. 如果自动选择的子网络和网络上已经存在的子网络冲突, 或者你需要自定义其他低级网络设置 \(如MTU\) , 这个功能将非常有用.

自定义 `ingress` 网络需要删除并重新创建它. 而 `ingress` 网络通常在你创建服务之间就已经被创建了. 如果你已经创建了服务并且开放了端口, 你需要在删除 `ingress` 网络前停止这些服务.

当 ingress 网络不存在时, 没有开放端口的服务会继续运行, 但不会进行负载均衡. 而开放了端口的服务会受到影响, 例如开放 80 端口的 WordPress 服务.

* 通过 `docker network inspect ingress` 命令检查 `ingress` 网络, 然后停止所有连接到这个网络的服务. 这些服务应该开放了端口, 例如开放了 80 端口的 WordPress 服务. 如果这些服务没有停止, 下一步操作将会报错.
* 删除已经存在 ingress 网络:

```bash
docker network rm ingress

WARNING! Before removing the routing-mesh network, make sure all the nodes
in your swarm run the same docker engine version. Otherwise, removal may not
be effective and functionality of newly created ingress networks will be
impaired.
Are you sure you want to continue? [y/N]
```

* 通过 `--ingress` 参数创建一个新的覆盖网络, 包括你想要的自定义选项. 以下实例设置了一个 MTU 为 1200 字节, 子网络为 `10.11.0.0/16` , 并且将网关设置为 `10.11.0.2`

```bash
docker network create \
--driver overlay \
--ingress \
--subnet=10.11.0.0/16 \
--gateway=10.11.0.2 \
--opt com.docker.network.driver.mtu=1200 \
my-ingress
```

{% hint style="info" %}
你可以为你的创建的 ingress 网络命名, 但是你只能拥有一个 ingress 网络.
{% endhint %}

* 重新启动刚才停止的服务

### 自定义 docker\_gwbridge 接口

`docker_gwbridge` 是一个虚拟网桥, 它将覆盖网络 \(包括 ingress 网络\) 链接到了各个 Docker 守护进程主机的物理网络. 当你为 Docker 主机初始化集群或者加入集群的时候, 它会被自动创建, 但它不是一个 Docker 的设备, 而是存在于 Docker 主机的内核中. 如果你需要对其进行设置, 你必须在将 Docker 主机加入到群中之前或从群中临时删除主机之后进行自定义.

* 停止 Docker
* 删除已存在的 `docker_gwbridge` 接口

```bash
sudo ip link set docker_gwbridge down
sudo ip link del dev docker_gwbridge
```

* 启动 Docker, 但不要初始化或加入集群
* 通过 `docker network create` 命令和自定义的配置, 创建或重建 `docker_gwbridge` . 以下的示例使用了子网络 `10.11.0.0/16` . 

```bash
docker network create \
--subnet 10.11.0.0/16 \
--opt com.docker.network.bridge.name=docker_gwbridge \
--opt com.docker.network.bridge.enable_icc=false \
--opt com.docker.network.bridge.enable_ip_masquerade=true \
docker_gwbridge
```

* 初始化或加入集群, 这时 `docker_gwbridge`已经存在, Docker 不会再自动创建它.

## 适用于集群服务的操作

### 在覆盖网络上开放端口

连接到同一个覆盖网络的集群服务只会为彼此开放所有的端口. 为了让服务外部能够访问某个端口, 这个端口必须在 `docker service create` 或 `docker service update` 命令上通过 `-p` 或 `--publish` 参数开放. 该参数支持冒号分隔的旧语法和逗号分隔的新语法, 但逗号分隔的语法有更加好的语义性.

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x53C2;&#x6570;&#x503C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>-p 8080:80
          <br />&#x6216;&#x8005;</p>
        <p>-p published=8080,target=80</p>
      </td>
      <td style="text-align:left">&#x5C06;&#x670D;&#x52A1;&#x7684; 80/tcp &#x7AEF;&#x53E3;&#x6620;&#x5C04;&#x5230;&#x8DEF;&#x7531;&#x7F51;&#x683C;&#x7684;
        8080/tcp</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>-p 8080:80/udp
          <br />&#x6216;&#x8005;</p>
        <p>-p published=8080,target=80,protocol=udp</p>
      </td>
      <td style="text-align:left">&#x5C06;&#x670D;&#x52A1;&#x7684; 80/udp &#x7AEF;&#x53E3;&#x6620;&#x5C04;&#x5230;&#x8DEF;&#x7531;&#x7F51;&#x683C;&#x7684;
        8080/udp</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>-p 8080:80/tcp -p 8080:80/udp
          <br />&#x6216;&#x8005;</p>
        <p>-p published=8080,target=80,protocol=tcp -p published=8080,target=80,protocol=udp</p>
      </td>
      <td style="text-align:left">&#x5C06;&#x670D;&#x52A1;&#x7684; 80/tcp &#x7AEF;&#x53E3;&#x6620;&#x5C04;&#x5230;&#x8DEF;&#x7531;&#x7F51;&#x683C;&#x7684;
        8080/tcp, &#x5E76;&#x4E14;&#x5C06;&#x670D;&#x52A1;&#x7684; 80/udp &#x7AEF;&#x53E3;&#x6620;&#x5C04;&#x5230;&#x8DEF;&#x7531;&#x7F51;&#x683C;&#x7684;
        8080/udp</td>
    </tr>
  </tbody>
</table>### 集群服务绕过路由网格

默认情况下, 集群服务通过路由网格开放端口. 当你连接到任何一个集群节点的开放端口时 \(无论这个端口上是否正在运行服务\) , 你都会被重定向到正在运行这个服务的工作节点. 实际上, Docker 充当了集群服务的负载均衡器. 使用路由网格的服务在虚拟 IP \(VIP\) 模式下运行. 即使服务在每个节点都运行了实例 \(通过 --mode global 参数\) 也会使用路由网格. 使用路由网格时, 不能明确是哪个 Docker 节点在为客户端请求提供服务.



