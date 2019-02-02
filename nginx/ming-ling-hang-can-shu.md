---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 命令行参数

## -? \| -h

打印命令行工具的帮助文档

## -c file

使用其他配置文件地路径替代默认的路径

## -g directives

通过字符串设置全局配置指令, 例如:

```bash
nginx -g "pid /var/run/nginx.pid; worker_processes `sysctl -n hw.ncpu`;"
```

## -p prefix

设置保存Nginx服务配置文件的路径前缀, 默认值为 /usr/local/nginx

## -q 

在测试配置期间禁止显示非错误消息

## -s signal

发送信号到主进程. 信号可以为:

* stop - 快速关闭
* quit - 正常关闭
* reload - 重新加载配置, 创建新的工作进程, 并正常关闭旧的工作进程
* reopen - 重启打来日志文件

## -t

测试配置文件, 检查配置文件是否符合语法, 并尝试打开相关的文件

## -T

与 -t 相同, 但会打印配置到标准输出\( 1.9.2+ \)

## -v

打印Nginx的版本

## -V

打印Niginx的版本, 编译器的版本, 配置参数

