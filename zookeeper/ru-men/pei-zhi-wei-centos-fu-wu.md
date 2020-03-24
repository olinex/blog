---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 配置为CentOS服务

添加用户

```bash
sudo groupadd zookeeper
sudo useradd -M -s /bin/false -g zookeeper zookeeper
cp /opt/apache-zookeeper-3.5.6-bin/conf/zoo_sample.cfg /opt/apache-zookeeper-3.5.6-bin/conf/zoo.cfg
sudo chown zookeeper:zookeeper -R /opt/apache-zookeeper-3.5.6-bin/
sudo chmod 750 -R /opt/apache-zookeeper-3.5.6-bin/
sudo chown zookeeper:zookeeper -R /var/zookeeper
sudo chown zookeeper:zookeeper -R /var/log/zookeeper
```

创建服务配置

```bash
vim /etc/systemd/system/zookeeper.service
```

```bash
[Unit]
Description=ZooKeeper Service
After=network.target
After=syslog.target

[Service]
Environment=ZOO_LOG_DIR=/var/log/zookeeper
SyslogIdentifier=zookeeper

Type=forking
User=zookeeper
Group=zookeeper
ExecStart=/opt/apache-zookeeper-3.5.6-bin/bin/zkServer.sh start /opt/apache-zookeeper-3.5.6-bin/conf/zoo.cfg
ExecStop=/opt/apache-zookeeper-3.5.6-bin/bin/zkServer.sh stop /opt/apache-zookeeper-3.5.6-bin/conf/zoo.cfg
ExecReload=/opt/apache-zookeeper-3.5.6-bin/bin/zkServer.sh restart /opt/apache-zookeeper-3.5.6-bin/conf/zoo.cfg

[Install]
WantedBy=default.target
```

注册服务

```bash
sudo systemctl enable zookeeper
```

启动服务

```bash
sudo systemctl start zookeeper
```

