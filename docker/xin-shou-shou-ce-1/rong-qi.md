---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 第二部分 - 容器

## 前置条件

* 已安装 Docker \(1.13+\)
* 已完成 第一部分 的阅读
* 快速测试你的环境, 确保一切准备就绪

```text
docker run hello-world
```

## 简介

现在让我们通过 Docker 的方式构建一个应用. 我们从层次接口的底层开始, 构建一个容器. 在此之上的第三部分是我们的服务, 它定义了容器是如何在生产环境部署. 最后, 在最顶层, 也就是第五部分, 是我们的栈, 它定义了所有的服务是如何关联的.

* 栈
* 服务
* **容器** \(我们在这一层\)

## 新的开发环境

在过去, 如果你要开始开发一个 Python 应用, 你首先在你的计算机上安装一个Python 编译器. 但是, 这会导致一种情况: 你的计算机上的环境需要完美地运行, 以便应用按预期运行, 并且还需要与生产环境相匹配.

通过 Docker, 你只需要获取一个可便携移动的 Python 编译器 作为镜像, 并且不需要安装. 接着, 将 Python 镜像和你的应用代码一起包装起来, 以确保你的应用/依赖/编译器能够一起运行.

这种可便携移动的镜像, 我们可以通过 `Dockerfile` 来配置.

## 通过 Dockerfile 创建容器

`Dockerfile` 定义了容器内的环境都包含了那些东西. 在此环境中, 可以虚拟化地访问资源, 如网络接口和磁盘驱动器, 并于你系统的其他部分隔离, 因此你需要将接口和外部对应起来, 并且具体说明那些文件你要"复制"到虚拟环境内. 但是, 你可以预计, 你通过这个 Dockerfile 构建的应用在别的任何地方都具有相同的行为.

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

运行应用, 通过 `-p` 参数, 将本地机器的 4000 端口和容器的 80 端口绑定建立映射:

```bash
docker run -p 4000:80 friendlyhello
```

你可以看到一条信息, 表明 Python 已经在 `0.0.0.0:80` 端口上运行了应用. 但这个信息来自于容器内部, 

