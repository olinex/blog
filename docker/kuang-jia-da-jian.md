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



