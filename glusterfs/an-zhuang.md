---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 安装

部署分布式存储系统GlusterFS, 需要至少两个服务器节点, 我们假设存在三个服务器:

* node001    172.26.172.61
* node002    172.26.172.62
* node003    172.26.172.63

### 统一Host

```bash
# 需在所有的节点执行
echo "172.26.172.61    node001    node001" >> /etc/hosts
echo "172.26.172.62    node002    node002" >> /etc/hosts
echo "172.26.172.63    node003    node003" >> /etc/hosts
```

### 安装GlusterFS

对于CentOS 7以上的服务器而言, 可以通过yum源非常轻松地安装GlusterFS

```bash
# 需在所有的节点执行
yum install -y centos-release-gluster
yum install -y glusterfs-server
systemctl enable glusterd
systemctl start glusterd
```

### 格式化分区

```bash
# 需在所有的节点执行
fdisk /dev/vdb
n # 创建新分区
p # 创建主分区
1 # 分配分区号
/enter # 设置默认块大小
/enter # 设置最后的柱面
w # 保存
```

### 挂载分区

```bash
# 需在所有的节点执行
mkfs.xfs -i size=512 /dev/vdb1
mkdir -p /bricks/brick1
echo "/dev/vdb1 /bricks/brick1 xfs defaults 1 2" >> /etc/fstab
mount -a && mount
```

### 配置防火墙

```bash
# 需在所有的节点执行
firewall-cmd --zone=public --add-port=24007-24008/tcp --permanent
firewall-cmd --reload
```

### 节点连接

```bash
# 任意节点执行即可
gluster peer probe node001
gluster peer probe node002
gluster peer probe node003
```

### 创建分布式卷

```bash
# 需在所有的节点执行
mkdir /bricks/brick1/gv0
# 任意节点执行即可
gluster volume create gv0 replica 2 \
node001:/bricks/brick1/gv0 \
node002:/bricks/brick1/gv0 \
node003:/bricks/brick1/gv0

gluster volume start gv0
gluster volume info

```

### 验证测试

```bash
mount -t glusterfs node001:/gv0 /mnt
for i in `seq -w 1 100`; do cp -rp /var/log/messages /mnt/copy-test-$i; done
ls -lA /mnt | wc -l
ls -lA /bricks/brick1/gv0
```

