---
description: '文章引用于RAHUL K(https://tecadmin.net/install-postgresql-server-centos/)'
---

# Postgresql-10安装

由于现阶段Postgresql的官方yum源最新版本为9.4\(截至至2018/11/08\), 因此需要配置指定的yum源后方可安装

* 安装yum源

```bash
yum install https://yum.postgresql.org/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
```

{% hint style="info" %}
当你看到这篇文章的时候, postgresql的版本可能已经大于10.2, 若要安装其他版本, 可以浏览ftp地址寻找指定版本的yum源
{% endhint %}

* 安装服务

```bash
yum install postgresql10-server postgresql10
```

* 初始化数据库

```text
/usr/pgsql-10/bin/postgresql-10-setup initdb
```

* 修改密码配置

```bash
vim /var/lib/pgsql/10/data/pg_hba.conf
```

```text
修改前 host    all             all             127.0.0.1/32            ident
修改后 host    all             all             127.0.0.1/32            trust
```

{% hint style="info" %}
postgresql-10以前配置文件处于/var/lib/pgsql/data/pg\_hba.conf
{% endhint %}

* 运行服务

```bash
systemctl start postgresql-10.service
systemctl enable postgresql-10.service
```

