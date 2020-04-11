# 安装

### 安装环境

```bash
yum install -y git
yum install -y java-1.8.0-openjdk  java-1.8.0-openjdk-devel
```

### 安装Jenkins

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

