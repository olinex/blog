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

#### env

{% tabs %}
{% tab title="语法" %}
**env** variable\[=value\];
{% endtab %}

{% tab title="默认值" %}
env TZ;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
默认情况下, 除了 `TZ` 之外, Nginx 会将从父进程继承来的环境变量全部移除. 这个命令运行保留一些变量, 修改他们的值, 或者创建新的环境变量. 这些变量将会:

* 将会在动态更新可执行文件的时候继承
* 用于 Perl 模块
* 用于工作进程. 我们需要记住的是, 通过这种方式控制系统库并不是总是有效的, 因为通常情况下, 库只会在初始化的时候检查这些变量, 而此时变量尚未被指令设置. 除了上面提到的动态更新可执行文件

TZ 变量总是被继承并可用于 Perl 模块, 除非这个变量被显示定义不再使用. 例如:

```text
env  MALLOC_OPTIONS;
env PERL5LIB=/data/site/modules;
env OPENSSL_ALLOW_PROXY_CERTS=1;
```

{% hint style="info" %}
Nginx 环境变量应该只在 Nginx 内部使用, 不应该被用户直接设置.
{% endhint %}
{% endtab %}
{% endtabs %}

#### error\_log

{% tabs %}
{% tab title="语法" %}
**error\_log** file \[level\];
{% endtab %}

{% tab title="默认值" %}
error\_log logs/error.log error;
{% endtab %}

{% tab title="上下文" %}
main, http, mail, stream, server, location
{% endtab %}

{% tab title="说明" %}
可配置日志. 不同的日志可以被设置为相同的级别. 如果在 `main` 上下文内, 日志配置没有被明确定义, 将会使用默认的日志文件.

第一个 `file` 参数定义了保存日志记录的文件路径. 特殊的 `stderr` 值对应了标准错误文件. 通过 `syslog:` 前缀声明, 可将日志写入 syslog. 通过 `memory:` 前缀和缓冲区大小 `size` 声明, 可将日志写入循环内存缓冲区, 通常被用与调试\(1.7.11+\).

第二个 level 参数定义了错误的等级, 错误的等级可以为: `debug` , `info` , `notice` , `warn` , `error` , `crit` , `alert` , `emerg` . 上述日志等级按照严重程度排序. 例如, 默认的 `error` 等级, 会导致 `error` , `crit` , `alert` , `emerg` 等级的日志被保存起来. 如果不明确指定日志等级, 则默认为 `error` 等级.

{% hint style="info" %}
为了能让 debug 等级的日志生效, Nginx 需要在构建时使用 --with-debug 参数, stream 上下文需要 1.7.11+ , mail 上下文需要 1.9.0+
{% endhint %}
{% endtab %}
{% endtabs %}



