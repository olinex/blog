---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# tmux

## 安装

### CentOS安装

#### 使用yum安装

```text
yum install -y tmux
```

#### 创建配置文件

```text
echo "set-option -g allow-rename off" >> ~/.tmux.conf
```

#### 创建新的会话

```text
tmux new -s <name>
```

#### 进入指定的会话

```text
tmux a -t <name>
```

## 命令列表

{% hint style="info" %}
tmux需要先执行命令前缀, 默认前缀为{CTRL + b} \(+号表示同时按下\). {{CTRL + b} - x} \(-号表示先后按下\)
{% endhint %}

### 系统

| 命令 | 说明 |
| :--- | :--- |
| {{CTRL + c} - ?} | 列出所有快捷键, 按q返回 |
| {{CTRL + c} - d} | 脱离当前会话 |
| {{CTRL + c} - s} | 列出所有的会话, 按q返回, 通过回车进入会话 |
|  |  |

