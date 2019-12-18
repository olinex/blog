---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# ACL权限

ZooKeeper使用ACL来控制对其节点的访问. ACL实现与UNIX文件访问权限非常相似: 它使用权限位来控制和禁止对节点及其所适用范围的各种操作. 但与标准的UNIX权限不同, ZooKeeper没有用户/用户组和其他三个标准范围的限制. 

同时注意, ACL仅与特定的节点有关, 与节点的子节点无关. 例如, /app仅可被ip: 172.16.16.1读取, /app/status是任意可读的, 则任何人都可以读取/app/status, 而不是指定ip.

ZooKeeper支持可插入身份验证方案. 使用格式`scheme:id`来指定aclid, 其中scheme是身份验证方案. 例如, ip:172.16.16.1是地址为172.16.16.1的主机的aclid.

当客户端连接到ZooKeeper并对其进行身份验证时, ZooKeeper会提取与该客户端相关的所有aclid, 当客户端尝试访问节点时, 将对比检查节点的ACL. ACL由\(scheme:expression, perms\)组成. 例如, \(ip:19:22.0.0/16, READ\)为IP地址以19.22开头的任何客户端提供READ权限.

## 权限

ZooKeeper支持以下权限:

* CREATE - 能创建一个节点
* READ - 能从节点获取数据和列出其子节点
* WRITE - 能为节点设置数据
* DELETE - 能删除一个子节点
* ADMIN - 能设置权限

### 内置验证方案

Zookeeper具有以下内置方案:

* world - 仅具有一个aclid: world:anyone, 代表任何人都可以
* auth - 没有使用任何id, 代表了已经通过认证的用户
* digest - 使用digest:username:password格式的aclid. 作为ACL表达式时, 按照格式:username:base64\(sha1\(password\)\)来书写
* ip - 使用客户端主机IP作为aclid. 作为ACL表达式时, 按照格式ip:address/bits来书写

## 设置权限

### 查看权限

```text
# getAcl <path>
getAcl /
```

### 设置权限

```text
# setAcl <path> scheme:expression:perm
setAcl / world:anyone:c
setAcl / ip:19:22.0.0/16:r
setAcl / auth:w
setAcl / digest:user:+owfoSBn/am19roBPzR1/MfCblE=:d
```

### 增加认证用户

```text
# addauth digest username:password
addauth digest user:123456
```

