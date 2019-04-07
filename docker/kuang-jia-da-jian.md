---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 框架搭建

## 目标

本篇教程的主要目标, 是搭建一个自带私人仓库的单节点Docker集群框架, 你不必担心单节点的可扩展性, 当我们完成了框架搭建后, 集群的扩展将会非常自然.

## 前提条件

* 一台 `CentOS7` 的主机是我们所需要的全部了, Docker 对各个版本的 Linux 支持都非常的好, 其他系统都可以以此为参考, 但 CentOS7 以下的版本 Docker \(18.09\) 并没有经过完全测试.
* 一个私有二级域名, 在本教程后续都将使用 `<docker.domain.com>` 来标识, 若没有域名, 可以通过固定IP替代

## 框架结构

## 搭建步骤

### 安装 Docker

#### 卸载旧版本

```bash
sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
```

#### 通过yum源来安装 Docker

在最开始, 我个人建议大家通过仓库来安装 Docker, 这对我们的脑力负担最小.

* 安装相关的依赖包

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

* 安装 Docker 的yum源

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

* 安装最新版本的 Docker

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

若要安装指定版本的 Docker , 可以通过以下命令打印可以安装的版本:

```bash
yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```

再通过安装命令安装

```bash
sudo yum install docker-ce-<版本号> docker-ce-cli-<版本号> containerd.io
```

{% hint style="info" %}
版本号必须完整, 如`docker-ce-18.09.1`
{% endhint %}

* 运行 Docker

```bash
sudo systemctl start docker
```

* 验证 Docker 的运行情况

```bash
sudo docker run hello-world
```

### 安装 Docker 编排工具

Docker Compose 是 Docker 官方的服务编排工具, 它的使用非常简单, 只需要编写好 `docker-compose.yml` 文件就可以通过命令行控制服务的各个细节. 我们通过 `pip` 工具能够轻松安装编排工具:

```bash
pip install docker-compose
```

### 初始化集群

Docker 的集群创建是非常简单的, 我们现在只需要创建一个可以后续扩展的单节点集群:

```text
sudo docker swarm init
```

{% hint style="info" %}
若集群已经创建, 可以通过sudo docker swarm leave \[--force\] 来让节点离开这个集群, 若管理节点离开了集群, 且集群内再也没有别的节点, 则集群将会被废弃, 集群内的secret 文件将会被删除
{% endhint %}

### 生成TLS证书

* 先让我们暂且在 /home 下创建一个工作环境, 并在工作环境下创建一个 TLS 证书的存储文件

```bash
cd /home && mkdir docker && cd docker && mkdir certs
```

* 创建一个生成 TLS 证书的脚本 `generate-ssl.sh`

```bash
#!/usr/bin/env bash
openssl req \
-newkey rsa:4096 -nodes -sha256 -keyout certs/<docker.domain.com>.key \
-x509 -days <天数> -out certs/<docker.domain.com>.crt
```

* 运行脚本并生成证书

```bash
sh generate-ssl.sh
```

### 生成帐号密码

对于 Docker 的仓库服务, 我们希望能够从公网访问, 这便于我们在本地机器上进行开发, 但是在公网环境下是非常不安全的, 我们希望我们的镜像不会被公开, 也不希望任何未知人士都能修改我们的镜像. 因此我们需要为仓库服务添加鉴权机制, 注意当前的目录一定要在我们的工作环境 `/home/docker` 下, 为了将来能够更加方便地增加帐号密码, 可以将以下代码封装为一个脚本 `create-user.sh` :

```bash
#!/usr/bin/env bash

# get username and password
read -p "Enter your username: " USERNAME
read -p "Enter your password: " -s PASSWORD

docker run --rm --entrypoint htpasswd registry:2 -Bbn $USERNAME $PASSWORD >> auth/htpasswd
```

在 /home/docker 下创建一个 auth 文件夹用于保存帐号密码文件:

```bash
mkdir /home/docker/auth
```

运行我们刚才创建的 `create-user.sh` 脚本创建用户:

```bash
sh create-user.sh
```

我们在 `/home/docker/auth/` 下创建了 `htpasswd` 文件, 该文件会以 `帐号:密码哈希值` 的形式保存帐号密码.

### 运行仓库服务

非常好, 我们现在距离目标只剩下最后一步, 让我们通过 Compose 编排工具来部署仓库, 在 `/home/docker` 下创建一个 `docker-compose.yml` 文件, 用于创建仓库:

```yaml
# docker-compose 的语法有多种版本, 从 1.x 到最近的 3.7
version: "3.7"
services:
  # 声明了服务的名称为 registry
  registry:
    image: registry:2
    # 将容器内仓库的端口与主机的端口绑定, 可以绑定多对接口
    ports:
      # 主机端口:服务端口
      - 5000:5000
    deploy:
      # 只运行一个任务实例
      replicas: 1
      restart_policy:
        # 重新运行仓库任务的条件
        condition: on-failure
      placement:
        constraints:
          # 对部署的主机进行约束, 只允许任务实例部署在管理节点上
          - node.role == manager
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: "/certs/<docker.domain.com>.crt"
      REGISTRY_HTTP_TLS_KEY: "/certs/<docker.domain.com>.key"
      REGISTRY_AUTH: "htpasswd"
      REGISTRY_AUTH_HTPASSWD_PATH: "/auth/htpasswd"
      REGISTRY_AUTH_HTPASSWD_REALM: "Registry Realm"
    volumes:
      # 将卷 registry 和容器内的 /var/lib/registry 绑定
      - registry:/var/lib/registry
      # 将当前的 certs 文件夹和容器内的 /certs 绑定
      - ./certs:/certs
      # 将当前的 auth 文件夹和容器内的 /auth 绑定
      - ./auth:/auth
    # 将服务加入 manager 网络
    networks:
      - manager
volumes:
  registry:
# 创建一个网络
networks:
  manager:
```

最后, 执行以下命令便可以将仓库服务部署在集群内了:

```bash
docker stack deploy -c docker-compose.yml manager
```

{% hint style="info" %}
初次部署可能需要比较长的时间
{% endhint %}

## 验证

仓库部署完成后, 我们可以验证一下, 仓库是否工作正常

### 运行 hello-world

```bash
docker run hello-world
```

通过 `docker image ls` 可以看见, 我们已经从官方仓库下载了一个 `hello-world` 镜像

### 创建标签

```bash
docker tag hello-world <docker.domain.com>:5000/hello-world
```

### 登录仓库

```bash
docker login <docker.domain.com>
```

### 推送镜像

```bash
docker push <docker.domain.com>:5000/hello-world
```

### 删除镜像

```bash
docker image rm hello-world
docker image rm <docker.domain.com>:5000/hello-world
```

{% hint style="info" %}
删除 hello-world 镜像时, Docker 会提示有容器依赖这个镜像, 而导致无法删除, 可以通过 docker container rm 命令先移除容器
{% endhint %}

### 拉取镜像

```bash
docker pull <docker.domain.com>:5000/hello-world
```

拉取成功, 即代表我们的仓库已经部署成功了, 万岁!

## Portainer 安装

{% hint style="info" %}
以下内容并非必须了解的, 如果你对 Portainer 并不了解, 可以通过 [官网](https://www.portainer.io/) 获得更多的信息
{% endhint %}



