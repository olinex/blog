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

* 将证书写入 `docker secret`

```bash
docker secret create domain.crt certs/<docker.domain.com>.crt
docker secret create domain.crt certs/<docker.domain.com>.key
```

### 设置仓库标签

集群中需要一个单一节点来安装仓库镜像, 因此需要为其中一个节点设置标签来区分:

```bash
docker node update --label-add registry=true <node>
```

其中, `<node>` 为节点的 ID , 我们的集群中只有当前的节点, 可以通过 docker node ls 命令来查看, 该命令为当前节点增加了一个 `registry` 标签.

### 生成帐号密码

对于 Docker 的仓库服务, 我们希望能够从公网访问, 这便于我们在本地机器上进行开发, 但是在公网环境下是非常不安全的, 我们希望我们的镜像不会被公开, 也不希望任何未知人士都能修改我们的镜像. 因此我们需要为仓库服务添加鉴权机制, 注意当前的目录一定要在我们的工作环境 `/home/docker` 下, 为了将来能够更加方便地增加帐号密码, 可以将以下代码封装为一个脚本 `create-user.sh` :

```bash
#!/usr/bin/env bash

# get username and password
read -p "Enter your username: " USERNAME
read -p "Enter your password: " -s PASSWORD

docker run --rm --entrypoint htpasswd registry:2 -Bbn $USERNAME $PASSWORD >> auth/htpasswd
mkdir auth && docker run --rm --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
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

非常好, 我们现在距离目标只剩下最后一步, 创建一个 run-registry.sh 脚本, 用于创建仓库:

```bash
docker service create \
--name registry \
--secret <docker.domain.com>.crt \
--secret <docker.domain.com>.key \
--constraint 'node.labels.registry==true' \
--mount type=bind,src=/mnt/registry,dst=/var/lib/registry \
-e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/run/secrets/<docker.domain.com>.crt \
-e REGISTRY_HTTP_TLS_KEY=/run/secrets/<docker.domain.com>.key \
-e REGISTRY_AUTH=htpasswd \
-e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
--publish published=5000,target=5000 \
--replicas 1 \
registry:2
```

* `--name registry` 声明了服务的名称为 registry
* `--secret <docker.domain.com>.crt` 声明了仓库将会加载 TLS 证书文件
* `--secret <docker.domain.com>.key` 声明了仓库将会加载 TLS 密钥文件
* `--constraint 'node.labels.registry==true'` 声明了仓库只会在含有 registry 标签的节点部署
* `--mount type=bind,src=/mnt/registry,dst=/var/lib/registry` 将镜像内的位置和主机的真实位置绑定, `src` 为主机的真实位置, `dst` 为镜像内的位置
* `-e REGISTRY_HTTP_ADDR=0.0.0.0:5000` 声明了仓库服务开放的 IP 和端口
* -e REGISTRY\_HTTP\_TLS\_CERTIFICATE=/run/secret/&lt;docker.domain.com&gt;.crt 声明了仓库将会使用的 TLS 证书文件
* 
