---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 字典

### Kubernetes

是一个领先且全新的基于容器技术的分布式架构方案

### Borg

Kubernetes的前身, 是谷歌内部使用的大规模集群管理系统

### 资源对象声明

Kubernetes将各种资源\(如硬盘, 机器, 服务\)对象化, 并通过YAML文件来声明, 绝大多数情况下我们都是通过YAML文件来控制Kubernetes

### Node

Kubernetes集群内的服务器资源, 我们称之为节点, 可以是虚拟机也可以是物理机

### Master Node

Kubernetes集群内的控制节点, 可以是一个节点, 也可以是多个节点

### Worker Node

Kubernetes集群内的工作节点, 用户自定义的服务一般都运行在工作节点上, 控制节点可以是工作节点, 但一般不这么做

### Etcd

运行在控制节点上的关键服务, Kubernetes中用于存储各种配置的数据库, 尤其是资源对象的声明配置

### Kube API Server

运行在控制节点上的关键服务, 提供了Kubernetes集群内所有资源增删改查的REST API

### Kube Controller Manager

运行在控制节点上的关键服务, 直接控制了所有资源对象

### Kube Scheduler

运行在控制节点上的关键服务, 负责根据资源情况, 间接调度资源对象

### Kubelet

运行在工作节点上的关键服务, 受Kube Controller Manager直接管理, 负责管理工作节点上其余资源对象

### Kube Proxy

运行在工作节点上的关键服务, 负责节点内各个资源对象的通信与负载均衡, 以及各个节点间的通信

### Container Engine

运行在工作节点上的关键服务, 负责节点内容器的管理

### 虚拟IP池

也称为集群IP池, Kubernetes在创建时, 会选用一个网段作为集群内部网络访问的虚拟IP池, Kubernetes会为一些资源对象赋予这些IP

### Pod

Kubernetes集群内, 服务的最小单位, 也被称为容器组, 是多个容器实例的组合, 容器实例至少包含2个

### Pod IP

每个容器组在实例化后, 都会从虚拟IP池获取到一个用于集群内部访问的IP

### Pause

容器组内的根容器实例, 容器组内其余容器实例通过共享pause的Pod IP与Pod外部进行通信, 也通过共享pause的挂载卷共享硬盘资源

### Endpoint

容器组通常会开放自身的一些端口供外部访问, 容器的Pod IP和端口共同组成了一个端点, 一个容器组可以有多个端点

### Requests

容器组对资源的最小申请量, 包括CPU和内存, 但不包括硬盘

### Limits

容器组对资源的最大申请量, 包括CPU和内存, 但不包括硬盘

### Label

标签是Kubernetes集群内一个非常重要的核心概念, 一个是一个键值对, 可以被附加到各种资源对象上, 资源对象可以拥有任意数量不同的标签, 相同的标签也可以附加到不同的资源对象上

### Label Selector

当两个不同的资源对象需要进行关联时, 其中一个资源对象可以通过标签过滤器, 选定符合一组资源对象进行关联, 可以类比为SQL语句的where查询条件

### Equal Based Label Selector

仅支持等式的标签选择器

### Set Based Label Selector

支持多种逻辑的标签选择器, 如 !=   IN

### Replication Controller

集群控制器, 简称RC, 这里所说的集群并非是整个Kubernetes集群, 而是一组容器组, 或者更通俗地讲是"容器组控制器", RC通过Equal Based Label Selector将一组容器组划分为一个集群, Kubernetes则可以通过RC的配置, 自动管理这些容器组. 目前RC已经是一个"落后"的概念, 从Kubernetes 1.2之后, 使用Replica Set来替代

### Replica Set

新一代RC, 简称RS, 与RC不同的是, RS支持Set Based Label Selector, 并且在此基础之上, 衍生了很多更加高级的资源对象, 如Deployment, Job, CronJob

### Job

一次性任务, RS资源对象的一种, Kubernetes对这种资源对象进行了限制, 整个Kubernetes集群内, 同名的Job资源对象有且仅能有一个, Job执行完后也不会被清除. 适用于一次性任务, 如数据库初始化, 用户创建等

### CronJob

定时任务, RS资源对象的一种, 类似于Linux的crontab, Kubernetes会定时创建并运行该资源对象所定义的容器组, 适用于需要定时启动的任务, 如定时巡检, 数据导出等

### DaemonSet

守护进程服务, RS资源对象的一种, Kubernetes会为每个节点部署其定义的容器组, 适用于每个节点都要运行的服务, 如日志收集Filebeat, 流量监控.

### Deployment

无状态服务, RS资源的一种, 也是我们最常使用的一种资源对象, Kubernetes会认为这种服务不会有差异化的行为, 其管理的每个容器组都不会有差别, 容器组之间也不会有交互, 不需要知道彼此的存在, 因此可以自由地伸缩容器组实例个数, 适用于Web服务, GRPC服务等

### StatefulSet

有状态服务, RS资源的一种, 区别于Deployment, 有状态服务会为每个容器组赋予用于内部通讯的唯一ID, 且集群规模比较固定, 不能随意改动. 同时容器组会按顺序启动

### Horizontal Pod Autoscaler

容器组的自动横向扩展声明, 可以根据容器组整体的负载情况, 进行自动扩展

### Service

服务也是Kubernetes集群内一个非常重要的核心概念, 其目的是为了能够更加灵活地将不同的容器组抽象为一个具体对外公开地服务, 外部通过服务就可以访问其背后地容器组, 而不需要关心容器组是如何组织部署的. Sevice同样也是通过Label Selector, 将不同的容器组进行划分, Kubernetes会为每个服务指派一个固定的标识ID和虚拟IP, 通过标识ID能够解析出虚拟IP, 在集群地任意节点内, 都可以访问服务背后地容器组. 同时服务也会监听容器组地变化, 从而实现自动发现, 负载均衡等功能. 绝大多数情况下Service与Deployment成对出现.

### Cluster IP

集群IP, 也称为虚拟IP, 一般指的服务所被赋予虚拟IP

### Headless Service

Service的一个变种, 主要用于StatefulSet, 其与Service的主要不同点在与, Kubernetes不会为其赋予虚拟IP,  通过标识ID解析会返回StatefulSet背后所有容器组的Pod IP列表, 并且会为StatefulSet的每个容器组赋予用于外部访问的唯一ID, 如一个三节点的Kafka Statefulset, 它的Headless Service命名为kaf, StatefulSet命名为kafka, 则可以通过kafka-0.kaf, kafka-1.kaf, kafka-2.kaf, 访问到三个容器组.

### NodePort Service

Service的一个变种, 主要用于提供简单的外部访问方式, 该服务会将所有工作节点的某个指定端口开放, 并指向服务背后的容器组, 通过这种方式, 访问任意工作节点的指定端口都能够访问到这些容器组

### Ingress

上述的Service, 无论是那种类型, 都是基于TCP/IP层实现的负载均衡方案, 对于应用层路由, 我们需要使用Ingress来实现. Ingress本质上是多个Nginx容器组, 通过声明路由转发规则和服务的名称和端口, 可以将七层流量转发给指定服务, 同时也支持转发TCP/IP流量. 一般生产环境下, Kubernetes集群对外提供服务, 通过Ingress是最佳实践方案.

### 外部负载均衡

外部要想访问Kubernetes集群内的Ingress服务, 仍然缺少至关重要的一个环节: 该如何访问到集群内的Ingress服务呢? Kubernetes原生将这一部分工作交给了云原生厂商或使用者自身, 一般有以下方案:

1. 将ingress作为DaemonSet部署在每个节点上, 通过Ingress绑定nodePort Service, 如谷歌的nginx-ingress-controller
2. 将ingress作为Deployment, 通过Kube API Server提供的接口, 实时监控Ingress的部署情况, 并将流量转发给绑定了hostPort的Ingress容器组内

### Namespace

命名空间, 为了满足多租户的需求, Kubernetes内的资源对象, 绝大多数都可以被划分在不同的命名空间下

### Volume

存储卷, 是硬盘资源的一种申明, 由于硬盘资源的可共享特性, 因此硬盘资源往往需要预先申明并在容器组内的多个容器间挂载, 存储卷可以是Pause内的一个临时文件夹, 也是可以节点上的一个真实的文件路径, 更可以是NFS网络硬盘等

### Presistent Volume

网络存储卷预定义, 不可以直接被容器组直接挂载使用, 其定义了网络存储卷的类型/权限/回收策略等设置.

### Presistent Volume Claim

网络存储卷需求, 容器组要使用网络存储卷作为容器组内的存储卷使用, 需要先行定义网络存储卷需求声明, 并将该声明作为存储卷挂载到容器组中

### ConfigMap

配置文件资源对象, Kubernetes可以将YAML文件作为配置文件存储在Ectd中, 容器组可以将配置文件资源作为存储卷挂载到容器内部, 配置文件的任何变化, 都可以直接体现在容器内文件的变化上, 而无需重启容器

### Secret

加密文件资源对象, ConfigMap的一种变体, 内部的配置信息将会被加密存储





