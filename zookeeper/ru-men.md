---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 入门

## 下载

获取ZooKeeper发行版, 下载最新的[稳定](http://zookeeper.apache.org/releases.html)版本.

## 安装

### 独立运行

在独立模式下设置ZooKeeper服务器非常简单. 该服务包含在一个单独的JAR文件中.

下载稳定版本的ZooKeeper后, 将其解压缩并进入其目录.

要启动ZooKeeper, 你需要一个配置文件, 创建`conf/zoo.cfg`:

```bash
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

* tickTime - ZooKeeper使用的基本时间单位\(毫秒\). 其作用于心跳监测, 并且最小会话超时将会是tickTime的两倍.
* dataDir - 存储内存数据库快照的位置, 除非另有说明, 否则也存储数据库更新的事务日志.
* clientPort: 监听客户端连接的端口.

该配置文件可以被任意命名, 但是为了便于讨论, 将其命名为conf/zoo.cfg.

创建配置文件后, 就可以启动ZooKeeper了:

```bash
bin/zkServer.sh start
```



