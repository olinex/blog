---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 配置为CentOS服务



添加用户

```bash
sudo groupadd kafka
sudo useradd -M -s /bin/false -g kafka kafka
cp /opt/kafka_2.13-2.4.0/server.properties /opt/kafka_2.13-2.4.0/kafka.properties
sudo chown kafka:kafka -R /opt/kafka_2.13-2.4.0/
sudo chmod 750 -R /opt/kafka_2.13-2.4.0/
```

创建服务配置

```bash
vim /etc/systemd/system/kafka.service
```

```bash
[Unit]
Description=Kafka Service
After=network.target
After=syslog.target

[Service]
SyslogIdentifier=kafka

Type=simpleforking
User=kafka
Group=kafka
ExecStart=/opt/kafka_2.13-2.4.0/bin/kafka-server-start.sh /opt/kafka_2.13-2.4.0/config/kafka.properties
ExecStop=/opt/kafka_2.13-2.4.0/bin/kafka-server-stop.sh /opt/kafka_2.13-2.4.0/config/kafka.properties

[Install]
WantedBy=default.target
```

注册服务

```bash
sudo systemctl enable kafka
```

启动服务

```bash
sudo systemctl start kafka
```

