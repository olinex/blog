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

在云服务器集群上运行 `docker stack deploy -c docker-compose.yml getstartedlab` 来部署应用.

```bash
docker stack deploy -c docker-compose.yml getstartedlab

Creating network getstartedlab_webnet
Creating service getstartedlab_web
Creating service getstartedlab_visualizer
Creating service getstartedlab_redis
```

现在你的应用已经在云服务器上运行了.

#### 运行集群命令验证部署情况

当你部署完以后, 你可以通过使用集群命令来浏览和管理集群. 以下是一些你现在很熟悉的一些例子:

* 使用 docker node ls 打印集群内的所有节点:

```bash
docker node ls

ID                            HOSTNAME                                      STATUS              AVAILABILITY        MANAGER STATUS
n2bsny0r2b8fey6013kwnom3m *   ip-172-31-20-217.us-west-1.compute.internal   Ready               Active              Leader
```

* 通过 docker service ps &lt;service&gt; 查看某个服务的实例列表:

```bash
[getstartedlab] ~/sandbox/getstart $ docker service ps vy7n2piyqrtr
ID                  NAME                  IMAGE                            NODE                                          DESIRED STATE       CURRENT STATE            ERROR               PORTS
qrcd4a9lvjel        getstartedlab_web.1   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 20 seconds ago                       
sknya8t4m51u        getstartedlab_web.2   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 17 seconds ago                       
ia730lfnrslg        getstartedlab_web.3   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 21 seconds ago                       
1edaa97h9u4k        getstartedlab_web.4   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 21 seconds ago                       
uh64ez6ahuew        getstartedlab_web.5   sam/getstarted:part6   ip-172-31-20-217.us-west-1.compute.internal   Running             Running 22 seconds ago        
```

#### 在云服务器上为服务开放端口

At this point, your app is deployed as a swarm on your cloud provider servers, as evidenced by the `docker`commands you just ran. But, you still need to open ports on your cloud servers in order to:

在这个时候, 你的应用已经在云服务器上部署为集群服务了, 并且已经通过刚刚的 docker 命令来证明. 但是, 你需要开放云服务器的端口:

* 如果使用了多个节点, 需要让 `web` 服务 和 `redis` 服务能够互相访问
* 允许任意工作节点对 `web` 服务的入站流量, 使得浏览器可以访问 `Hello World` 和 `Visualizer` 服务.
* 允许管理服务器对其他服务器的 SSH 入站流量\(可能已经你的云服务器已经设置好了\)

你需要为服务开放以下端口:

| Service | Type | Protocol | Port |
| :--- | :--- | :--- | :--- |
| web | HTTP | TCP | 80 |
| visualizer | HTTP | TCP | 8080 |
| redis | TCP | TCP | 6379 |

执行此操作的方法因云服务器而已. 我们以 亚马逊网站服务\(AWS\) 为例.

* `redis` 服务如何保存数据?

为了使 `redis` 服务正常工作, 你需要通过 `ssh` 登录到运行了管理服务器的云服务器, 并在 `/home/docker/` 创建 `data/` 目录, 然后运行 `docker stack deploy` . 另一个选择是改变 `docker-stack.yml` 内的文件路径为云服务器内的已存在路径. 这个例子并不包含这个这个步骤, 因此 redis 服务并不在示例中并不会更新.

## 迭代和清理

从现在开始, 你已经可以做任何在前面教程上学到的事情了.

* 通过修改 `docker-compose.yml` 文件并且通过 `docker stack deploy` 命令重新部署
* 通过修改代码并重新构建, 再推送新的镜像, 来修改应用的行为. \( 要重新部署, 需要重新执行早前的 [构建应用](https://docs.docker.com/get-started/part2/#build-the-app) 和 [发布镜像](https://docs.docker.com/get-started/part2/#publish-the-image) \)
* 你可以通过 docker stack rm 命令清除服务栈, 例如:

```bash
docker stack rm getstartedlab
```

与在本地的 Docker 虚拟主机内运行集群的情况不同, 无论你是否关闭本地主机, 你部署在云服务器上的集群和应用都将不受影响地继续使用.

