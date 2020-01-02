---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 入门

## 下载

获取ZooKeeper发行版, 下载最新的[稳定](http://zookeeper.apache.org/releases.html)版本.

## 安装

{% hint style="info" %}
ZooKeeper依赖Java环境, 需要安装相应版本的java-openjdk
{% endhint %}

### 独立运行

在独立模式下设置ZooKeeper服务器非常简单. 该服务包含在一个单独的JAR文件中.

下载稳定版本的ZooKeeper后, 将其解压缩并进入其目录.

要启动ZooKeeper, 你需要一个配置文件, 创建`conf/zoo.cfg`:

```bash
tickTime=2000
dataDir=/var/zookeeper
dataLogDir=/var/log/zookeeper
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

ZooKeeper使用log4j记录日志.

### 集群运行

以独立运行模式便于开发和测试. 但是在生产环境中, 我们应该以集群模式运行ZooKeeper. 同一个ZooKeeper应用下的复制组称为仲裁服务器, 仲裁的所有服务器都具有相同配置文件的副本.

{% hint style="info" %}
对于集群模式, 最少需要三台服务器, 强烈建议使用奇数台服务器. 如果只有两台, 可能会出现这样的情况: 器重一台服务器发生故障, 则没有足够多的服务器构成多数仲裁, 因此两个服务器其实并不如单个服务器稳定.
{% endhint %}

集群模式所需的`conf/zoo.cfg`配置类似于独立模式, 但存在一些区别, 例如:

```bash
tickTime=2000
dataDir=/var/zookeeper
dataLogDir=/var/log/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1.2888:3888
server.2=zoo2.2888:3888
server.3=zoo3.2888:3888
```

* initLimit - 用于限制仲裁中的ZooKeeper服务器必须连接到领导服务器的超时时长
* syncLimit - 用于限制服务器与leader服务器之间超时举例
* server.x - 列出了组成ZooKeeper集群的服务器, 它通过在数据目录中查找文件`myid`来知道它究竟是哪台服务器, 该文件以ASCII格式保存, 包含了服务器的编号. 服务器名称后的两个端口号`2888`和`3888`, 前者用于集群中进行仲裁时进行通讯, 以商定更新顺序. 更具体地说, ZooKeeper使用第一个端口将仲裁服务器连接到领导服务器. 当出现新的领导服务器的时候, 我们需要另一个端口来进行领导服务器选举, 也就是第二个端口.

{% hint style="info" %}
对于initLimit和syncLimit, 其单位是tickTime指定的时间. 在上述示例中, initLimit的超时为5个"嘀嗒", 即 2000ms \* 5 = 10秒.
{% endhint %}

{% hint style="info" %}
如果要在一台计算机上测试多个ZooKeeper服务器, 则每台服务器的指定名称为localhost, 病句有唯一的仲裁和领导选择端口\(例如上述示例中为2888:3888,2889:3889,2890:3890\)
{% endhint %}

## 管理存储

对于长时间运行的生产系统, 必须从外部管理ZooKeeper存储.

## 使用

### 连接到ZooKeeper

```bash
bin/zkCli.sh -server 127.0.0.1:2181
```

这使你可以执行一些简单的类似文件系统的操作.

连接后, 你应该会看到类似一下内容的信息:

```bash
Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
```

在shell中, 输入`help`以获取可以执行的命令列表, 如下所示:

```bash
[zkshell: 0] help
ZooKeeper host:port cmd args
    get path [watch]
    ls path [watch]
    set path data [version]
    delquota [-n|-b] path
    quit
    printwatches on|off
    create path data acl
    stat path [watch]
    listquota path
    history
    setAcl path acl
    getAcl path
    sync path
    redo cmdno
    addauth scheme auth
    delete path [version]
    deleteall path
    setquota -n|-b val path
```

你可以尝试一些简单的命令, 以了解这种命令行界面, 首先, 打印文件列表:

```bash
[zkshell: 8] ls /
[zookeeper]
```

通过运行`create /zk_test my_data`创建一个新的znode. 这将会创建一个新的znode并将字符串"my\_data"与该节点关联, 你应该看到:

```bash
[zkshell: 9] create /zk_test my_data
Created /zk_test
```

再次打印文件列表:

```bash
[zkshell: 11] ls /
[zookeeper, zk_test]
```

现在, zk\_test目录已经创建.

接下来, 通过运行以下`get`命令来验证数据是否与znode关联:

```bash
[zkshell: 12] get /zk_test
my_data
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 5
mtime = Fri Jun 05 13:57:06 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0
dataLength = 7
numChildren = 0
```

我们可以通过`set`命令来修改zk\_test相关的数据:

```bash
[zkshell: 14] set /zk_test junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
[zkshell: 15] get /zk_test
junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
```

最后, 通过发送`delete`命令来删除节点:

```bash
[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
```



