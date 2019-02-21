---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 核心功能

## 指令

#### accept\_mutex

{% tabs %}
{% tab title="语法" %}
**accept\_mutex** on \| off;
{% endtab %}

{% tab title="默认值" %}
accept\_mutex off;
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
如果 `accept_mutex` 被开启, 工作进程会依次接收新的连接. 反之, 则所有的工作进程都将会收到新的连接通知, 同时如果新的连接数很少, 则部分工作进程会浪费系统资源.

{% hint style="info" %}
若系统支持 EPOLLEXCLUSIVE 选项, 或者使用了 reuseport 则不需要开启这个功能, 在 1.11.3 之前, 默认值为 on .
{% endhint %}
{% endtab %}
{% endtabs %}

#### accept\_mutex\_delay

{% tabs %}
{% tab title="语法" %}
**accept\_mutex\_delay** time;
{% endtab %}

{% tab title="默认值" %}
accept\_mutex\_delay 500ms;
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
如果 accept\_mutex 功能被开启了, 其中一个工作进程刚刚接收到了一个新的连接, 可以设置其他工作进程尝试重新接受新连接的最大超时值.
{% endtab %}
{% endtabs %}

#### daemon

{% tabs %}
{% tab title="语法" %}
**daemon** on \| off;
{% endtab %}

{% tab title="默认值" %}
daemon on;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
是否将 Nginx 作为守护进程, 一般用于开发.
{% endtab %}
{% endtabs %}

#### debug\_connection

{% tabs %}
{% tab title="语法" %}
**debug\_connection** address \| unix: CIDR;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
可以开启指定客户端连接的调试日志记录功能. 其他的连接将会按照 `error_log` 指令配置的日志等级来记录日志. 调试连接为 IPv4 或 IPv6 \(1.3.0+, 1.2.1+\) 的网络地址, 还可以为域名地址. 对于 UNIX 套接字的连接 \(1.3.0+, 1.2.1+\) , 调试日志可以通过 `unix:` 参数来开启.

```text
events {
    debug_connection 127.0.0.1;
    debug_connection localhost;
    debug_connection 192.0.2.0/24;
    debug_connection ::1;
    debug_connection 2001:0db8::/32;
    debug_connection unix:;
    ...
}
```

{% hint style="info" %}
要想使这个指令生效, Nginx 需要通过 `--with-debug` 指令来构建
{% endhint %}
{% endtab %}
{% endtabs %}

#### debug\_points

{% tabs %}
{% tab title="语法" %}
**debug\_points** abort \| stop;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
这个命令用于调试. 当内部错误发生的时候, 例如在重启工作进程时套接字泄露, 将 `debug_points` 设置为 `abort` 会创建一个核心文件, 设置为 `stop` 会停止进程, 以便系统调试器进行进一步的分析.
{% endtab %}
{% endtabs %}



