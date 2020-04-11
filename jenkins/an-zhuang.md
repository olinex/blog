# 安装

### 安装环境

```bash
yum install -y git
yum install -y java-1.8.0-openjdk  java-1.8.0-openjdk-devel
```

### 安装Jenkins

```bash
wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

### 开放端口

```bash
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

### 修改配置

```bash
vim /etc/sysconfig/jenkins

JENKINS_HOME="/data/jenkins"
```

### 持久化存储

```bash
mkdir /data/jenkins
chown -R jenkins:jenkins /data/jenkins
```

### 启动服务

```bash
systemctl enable jenkins
systemctl start jenkins
```

### 初始化服务

通过浏览器访问8080端口, 进入jenkins的初始化界面, 这需要一点时间, 请耐心等待. 当页面提示需要输出初始化密码时, 可通过以下命令打印出密码, 复制粘贴至输入框并继续:

```bash
cat /data/jenkins/secrets/initialAdminPassword
```

提示是否安装插件时, 建议初次使用时, 安装jenkins推荐的常用插件

