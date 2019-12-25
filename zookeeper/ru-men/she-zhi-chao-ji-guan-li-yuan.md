---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 设置超级管理员

管理员的设置需要修改`bin/zkServer.sh`, 通过关键词`nohup`找到ZooKeeper的启动语句:

```text
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" \
"-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
```

在其后增加超级管理员的认证信息:

```text
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" \
"-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
"-Dzookeeper.DigestAuthenticationProvider.superDigest=${ZOO_SUPER_DIGEST}"
```

增加ZOO\_SUPER\_DIGEST环境变量, 并重启ZooKeeper服务:

```text
export ZOO_SUPER_DIGEST="<username>:<hash>"
bin/zkServer.sh restart
```

登录后, 通过超级管理员的帐号密码, 就可以访问任何受控节点了:

```text
bin/zkCli.sh -server
addauth digest <username>:<password>
```

