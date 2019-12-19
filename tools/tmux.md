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
| {{CTRL + b} - ?} | 列出所有快捷键, 按q返回 |
| {{CTRL + b} - d} | 脱离当前会话 |
| {{CTRL + b} - s} | 列出所有的会话, 按q返回, 通过回车进入会话 |
| {{CTRL + b} - \[} | 进入复制模式, 滚动查看历史控制台打印 |
| {{CTRL + b} - :} | 进入命令行模式 |

### 窗口

| 命令 | 说明 |
| :--- | :--- |
| {{CTRL + b} - c} | 创建新的窗口 |
| {{CTRL + b} - &} | 关闭当前窗口 |
| {{CTRL + b} - \[0-9\]} | 切换到指定的窗口 |
| {{CTRL + b} - p} | 切换到上一个窗口 |
| {{CTRL + b} - n} | 切换到下一个窗口 |
| {{CTRL + b} - l} | 切换至刚才的窗口 |
| {{CTRL + b} - w} | 列出所有的窗口, 按q返回, 通过回车进入窗口 |
| {{CTRL + b} - ,} | 重命名当前的窗口名称 |
| {{CTRL + b} - .} | 修改当前窗口的编号, 输入数字后回车确定 |
| {{CTRL + b} - f} | 在所有窗口查找自定的文本 |

