---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第二部分 - 容器

## 前提条件

* 已安装 Docker \(1.13+\)
* 已完成 第一部分 的阅读
* 快速测试你的环境, 确保一切准备就绪

```text
docker run hello-world
```

## 简介

现在让我们通过 Docker 的方式构建一个分布式应用. 我们从底层开始, 构建一个容器. 在此之上的第三部分是我们的服务, 它定义了容器是如何在生产环境部署. 最后, 在最顶层, 也就是第五部分, 是我们的栈, 它定义了所有的服务是如何关联的.

* 栈
* 服务
* **容器** \(我们在这一层\)

## 新的开发环境

在过去, 如果你要开始开发一个 Python 应用, 你首先要在你的计算机上安装一个Python 编译器. 但是, 这需要你的计算机上的环境与生产环境相匹配, 并且能按预期运行.

通过 Docker, 你只需要获取一个可便携移动的 Python 编译器 作为镜像, 并且不需要安装. 接着, 将 Python 镜像和你的应用代码一起包装起来, 以确保你的应用/依赖/编译器能够一起运行.

这种可便携移动的镜像, 我们可以通过 `Dockerfile` 来配置.

## 通过 Dockerfile 创建容器

`Dockerfile` 定义了容器内的环境都包含了哪些东西. 在此环境中, 可以虚拟化地访问资源, 如网络接口和磁盘驱动器, 并于你系统的其他部分隔离, 因此你需要将接口和外部对应起来, 并且具体说明那些文件你要"复制"到虚拟环境内. 但是, 你可以预计, 你通过这个 Dockerfile 构建的应用在别的任何地方都具有相同的行为.

### Dockerfile

在你的计算机上创建一个空的文件夹. 进入这个文件夹并创建 `Dockerfile` 文件, 复制以下内容到这个文件. 注意理解每一行语句的注释.

```text
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

`Dockerfile` 提及了一些我们尚未创建的文件, `app.py` 和 `requirements.txt` 让我们来创建它们:

#### requirements.txt

```text
Flask
Redis
```

#### app.py

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

我们能看到, `pip install -r requirements.txt` 安装了Python 的 Flask 和 Redis 库, 并且应用打印了环境变量 `ENV` , 同时打印了 `socket.gethostname()` 获取的主机名. 最后, 因为 Redis 服务并没有运行 \(我们只安装了 Python 库\), 我们可以预期, 尝试使用 Redis 会报错并且输出错误信息.

{% hint style="info" %}
在容器内部获取主机名会获得容器的 ID , 这个 ID 类似运行可执行文件时产生的进程 ID
{% endhint %}

这就是全部了! 你不需要在你的系统上安装 Python 以及 `requirements.txt` 内的任何依赖, 也不需要通过在系统中构建或运行镜像来安装他们. 这看上去似乎并没有部署好环境, 但它的确做到了.

## 构建应用

现在我们可以构建应用了. 确保你现在的目录位置正处于刚刚创建的文件夹. 执行 `ls` 命令:

```bash
ls
Dockerfile    app.py    requirements.txt
```

执行构建命令. 这会创建一个 Docker 镜像, 通过 --tag 或者 -t 参数, 可以为镜像命名:

```bash
docker build --tag=friendlyhello
```

那么刚创建的镜像在哪里呢? 他在你的本地 Docker 镜像资源库内:

```bash
docker image ls
REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

镜像的标签默认为 `lastest` , 完整定义镜像名称和标签的语法为 `--tag=friendlyhello:v0.0.1` 

### Linux 用户的常见问题:

#### 代理服务器设置

代理服务器可能会导致连接应用时被阻塞, 如果网络存在代理服务, 添加以下内容到 `Dockerfile` 内, 通过 `ENV` 设置代理服务器的主机和端口:

```text
# 设置代理服务器
ENV http_proxy host:port
ENV https_proxy host:port
```

#### DNS 设置

错误的 DNS 配置可能导致 `pip` 工具出现故障. 你可以设置你自己的 DNS 服务器地址来使 `pip` 能够正常工作. 通过创建或修改配置文件 `/etc/docker/daemon.json 的 dns` 值, 可以改变 Docker 守护进程的 DNS 配置:

```javascript
{
    "dns" : ["your_dns_address", "8.8.8.8"]
}
```

列表内, 第一个元素是 DNS 的服务器地址, 第二个参数是作为备份的 Google DNS 服务器地址.

保存 daemon.json 配置文件并重启 Docker 服务:

```bash
sudo service docker restart
```

重启后, 尝试重新运行构建命令.

## 运行应用

运行应用, 通过 `-p` 参数, 将本地主机的 4000 端口和容器的 80 端口绑定建立映射:

```bash
docker run -p 4000:80 friendlyhello
```

你可以看到一条信息, 表明 Python 已经在 `http://0.0.0.0:80` 端口上运行了应用. 但这个信息来自于容器内部, 因为内部并不知道应用接口 80 已经映射到本地主机的 4000, 你需要通过 `http://localhost:4000` 接口访问.

通过浏览器你可以看到网页内容:

![&#x6765;&#x6E90;&#x4E8E;&#x5B98;&#x65B9;&#x6587;&#x6863;\(https://docs.docker.com/get-started/part2/\)](../../.gitbook/assets/app-in-browser.png)

{% hint style="info" %}
如果你使用了Docker Toolbox on Windows 7, 需要使用 Docker 主机的 IP 来替代 localhost, 通过 docker-machine ip 可以查看 IP
{% endhint %}

你可以在命令行终端通过 `curl` 命令查看相同的内容:

```bash
$ curl http://localhost:4000

<h3>Hello World!</h3>
<b>Hostname:</b> 8fc990912a14<br/>
<b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

接口映射 `4000:80` 表示了 Dockerfile 内 `EXPOSE` 与开放接口的区别, 开放接口通过 `docker run -p` 命令设置. 

在终端内输入 CTRL+C 来停止应用.

{% hint style="info" %}
在 windows 系统中, CTRL + C 并不会停止容器的进程, 因此先要输入 CTRL + C 来获取输入管道\(或者重新打来一个新的\), 接着输入 docker container ls 来打印正在运行中的容器, 接着输入 docker container stop &lt;Container Name or ID&gt; 来停止指定的容器. 否则, 当你尝试重新运行容器时, 你会从守护进程收到一个错误响应.
{% endhint %}

现在让我们在后台运行运行应用:

```bash
docker run -d -p 4000:80 friendlyhello
```

你会获得容器的长 ID , 随后应用将会进入后台运行. 你的容器将会在后台运行. 你还可以通过 `docker container ls` 查看容器的缩写 ID \(两个 ID 在命令中都可以正常使用\)

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```

使用 CONTAINER ID 可以停止指定的进程:

```bash
docker container stop 1fa4ab2cf395
```

## 分享镜像

为了展示可移植性, 让我们将刚刚构建的镜像上传并在别的地方运行. 当你想要将容器部署在产品环境中时, 你需要知道如何将其推送到注册表内.

注册表是仓库的集合, 仓库是镜像的集合, 类似 GitHub 仓. 一个账号可以在注册表内可以创建多个仓库. docker 命令默认使用 Docker 的公开注册表.

### 通过 Docker ID 登录

如果你还没有 Docker 帐号, 可以在 [hub.docker.com](http://hub.docker.com) 注册.

在你的本地电脑登录到 Docker 的公开注册表:

```bash
docker login
```

### 标注镜像

将本地镜像与注册表内的仓库对应的表示方法为: `username/repository:tag` . `tag` 标签是可选的, 但建议使用, 因为注册表将根据标签对镜像进行版本管理. 建议为仓库和标签选取语义化的名称, 例如 `get-started:part2` . 这会将镜像推送至 get-started 仓库, 并且标注为 part2.

```bash
docker tag image username/repository:tag
```

例如:

```bash
docker tag friendlyhello gordon/get-started:part2
```

运行 `docker image ls` 来查看我们新标注的镜像:

```bash
$ docker image ls

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
gordon/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

### 推送镜像

上传你的镜像到注册表:

```text
docker push username/repository:tag
```

上传完毕后, 你的镜像将会公开生效, 如果你登录到 Docker Hub , 你会看到新的镜像已经创建.

### 从远程仓库获取镜像并运行

你现在可以通过 `docker run` 命令运行你的应用在任何主机上:

```text
docker run -p 4000:80 username/repository:tag
```

如果镜像不在本地主机上, Docker会从仓库拉取

```text
$ docker run -p 4000:80 gordon/get-started:part2
Unable to find image 'gordon/get-started:part2' locally
part2: Pulling from gordon/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for gordon/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

无论 `docker run` 在何处执行, 它会拉取你的镜像, 包括 Python 和 `requirements.txt` 内的依赖, 并运行你的代码. 它会将一切都放在一个简介的包内, 你不需要安装任何其他的东西\(除了 Docker 本身之外\).

