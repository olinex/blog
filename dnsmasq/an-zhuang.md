---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 安装

### 安装

```bash
yum install -y dnsmasq
```

### 修改配置

```bash
touch /etc/resolv.dnsmasq.conf
touch /etc/dnsmasq.hosts
vim /etc/dnsmasq.conf
```

```bash
resolv-file=/etc/resolv.dnsmasq.conf
listen-address=<intranet>,127.0.0.1
addn-hosts=/etc/dnsmasq.hosts
no-hosts

```

### 启动服务

```bash
systemctl enable dnsmasq
systemctl staart dnsmasq
```

