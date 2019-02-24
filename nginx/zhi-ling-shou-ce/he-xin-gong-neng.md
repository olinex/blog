---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 核心功能

## 指令

### accept\_mutex

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

### accept\_mutex\_delay

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

### daemon

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

### debug\_connection

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

### debug\_points

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

### env

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

### error\_log

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

### events

{% tabs %}
{% tab title="语法" %}
**events** {...}
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
提供配置文件的上下文, 其中包含了影响连接处理的指令.
{% endtab %}
{% endtabs %}

### include

{% tabs %}
{% tab title="语法" %}
**include** file \| mask;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
any
{% endtab %}

{% tab title="说明" %}
包含其他的文件, 或者和 `mask` 匹配的文件. 包含的文件内必须包括语法正确的指令和块.

例如:

```text
include mime.types;
include vhosts/*.conf;
```
{% endtab %}
{% endtabs %}

### load\_module

{% tabs %}
{% tab title="语法" %}
**load\_module** file;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
加载动态模块\(1.9.11+\):

```text
load_module modules/ngx_mail_module.so;
```
{% endtab %}
{% endtabs %}

### lock\_file

{% tabs %}
{% tab title="语法" %}
**lock\_file** file;
{% endtab %}

{% tab title="默认值" %}
lock\_file logs/nginx.lock;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
Nginx 使用锁定机制来实现 `accept_mutex` 和从共享内存中获取数据并序列化. 在大多数系统中, 锁是通过原子操作实现的, 这个命令将会被忽视. 而在其他系统中, 则使用"锁文件"实现. 这个指令制定了锁文件的前缀.
{% endtab %}
{% endtabs %}

### master\_process

{% tabs %}
{% tab title="语法" %}
**master\_process** on \| off;
{% endtab %}

{% tab title="默认值" %}
master\_process on;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
决定是否启动工作进程. 这个命令适用于开发人员.
{% endtab %}
{% endtabs %}

### multi\_accept

{% tabs %}
{% tab title="语法" %}
**multi\_accept** on \| off;
{% endtab %}

{% tab title="默认值" %}
multi\_accept off;
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
如果 multi\_accept 被关闭了, 工作进程会一次接受一个新连接. 否则, 一个工作进程将会一次接收所有的新连接.

{% hint style="info" %}
如果连接处理机制 kqueue 被启用, 这个指令将会被忽略, 因为它会向主进程报告有多少新连接等待连接.
{% endhint %}
{% endtab %}
{% endtabs %}

### pcre\_jit

{% tabs %}
{% tab title="语法" %}
**pcre\_jit** on \| off;
{% endtab %}

{% tab title="默认值" %}
pcre\_jit off;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
开启或关闭对配置文件分析时, 正则表达式的"实时编译"\(PCRE JIT\).

这个配置可以显著提高正则表达式的处理速度\(1.1.12+\).

{% hint style="info" %}
从 8.20 开始, 通过 --enable-jit 构建参数, 可以使 PCRE 库生效并开启实时编译. 构建好 PCRE 库后\(--with-pcre=\), 通过 --with-pcre-jet 构建参数开启.
{% endhint %}
{% endtab %}
{% endtabs %}

### pid

{% tabs %}
{% tab title="语法" %}
**pid** file;
{% endtab %}

{% tab title="默认值" %}
pid logs/nginx.pid;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
设置保存主进程id的文件位置.
{% endtab %}
{% endtabs %}

### ssl\_engine

{% tabs %}
{% tab title="语法" %}
**ssl\_engine** device;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
定义了 SSL 硬件加速器的名称
{% endtab %}
{% endtabs %}

### thread\_pool

{% tabs %}
{% tab title="语法" %}
**thread\_pool** name threads=number \[max\_queue=number\];
{% endtab %}

{% tab title="默认值" %}
thread\_pool default threads=32 max\_queue=65536;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
定义一个线程池及其名称, 使工作进程能够无阻塞地在多个线程间读写文件.

`threads` 参数定义了线程池内线程的数量.

当所有的线程都处于忙碌状态, 新的任务会在队列内等待. max\_queue 参数限制了队列内最多能等待多少任务. 默认情况下, 最多有 65536 个任务可以在队列内. 当队列满载时, 任务会结束并返回错误.
{% endtab %}
{% endtabs %}

### timer\_resolution

{% tabs %}
{% tab title="语法" %}
**timer\_resolution** interval;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
减少工作进程内计时器的分辨率, 从而减少系统调用 `gettimeofday()` 的次数. 默认情况下每当收到内核事件, `gettimeofday()` 会被调用一次. 通过减少分辨率, `gettimeofday()` 只会在特定周期 `interval` 内调用一次

例如:

```text
timer_resolution 100ms;
```

间隔的实现方式取决于连接处理机制:

* 当连接处理机制为 `kqueue` 时, 为 `EVFILT_TIMER`
* 当连接处理机制为 `eventport` 时, 为 `timer_create()`
* 其他情况下, 为 `setitimer()`
{% endtab %}
{% endtabs %}

### use

{% tabs %}
{% tab title="语法" %}
**use** method;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
设置了 Nginx 使用何种连接处理机制, 一般情况下不需要显式声明, 因为 Nginx 会自动使用最有效的机制.
{% endtab %}
{% endtabs %}

### user

{% tabs %}
{% tab title="语法" %}
**user** user group;
{% endtab %}

{% tab title="默认值" %}
user nobody nobody;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
设置了启用工作进程的用户和用户组, 如果用户组被忽略, 则会使用和用户同名的用户组.
{% endtab %}
{% endtabs %}

### worker\_aio\_requests

{% tabs %}
{% tab title="语法" %}
**worker\_aio\_requests** number;
{% endtab %}

{% tab title="默认值" %}
worker\_aio\_requests 32;
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
当开启了 `aio` 并且连接处理机制为 `epoll` , 可以设置单个工作进程的异步 I/O 操作的最大未完成数\(1.1.4+, 1.0.7+\).
{% endtab %}
{% endtabs %}

### worker\_connections

{% tabs %}
{% tab title="语法" %}
**worker\_connections** number;
{% endtab %}

{% tab title="默认值" %}
worker\_connections 512;
{% endtab %}

{% tab title="上下文" %}
events
{% endtab %}

{% tab title="说明" %}
设置工作进程能够同时开启的最大连接数.

你一定要牢记, 这个连接数影响了所有的连接\(包括与被代理服务器的连接\), 而不是只与客户端的. 另外, 真正能够同时开启的最大连接数, 不能超过当前开启文件的最大数量, 这个数量能够通过 `worker_rlimit_nofile` 配置
{% endtab %}
{% endtabs %}

### worker\_cpu\_affinity

{% tabs %}
{% tab title="语法" %}
**worker\_cpu\_affinity** cpumask ...;

**worker\_cpu\_affinity** auto \[cpumask\];
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
将工作进程与指定的 CPU 集合绑定. 每个 CPU 集合由位掩码指定. 每个工作进应该通过定义一个单独的集合. 默认情况下, 工作进程不与任何特定的 CPU 集合绑定.

例如:

```text
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;
```

将每个工作进程和不同的 CPU 绑定:

```text
worker_processes    2;
worker_cpu_affinity 0101 1010;
```

将第一个工作进程和 CPU0/CPU2 绑定, 将第二个工作进程和CPU1/CPU3 绑定. 

以下示例适用于超线程. 特殊的值 auto 允许自动地将可用的 CPU 与工作进程绑定:

```text
worker_processes auto;
worker_cpu_affinity auto;
```

可选的 mask 参数可以限制哪些 CPU 用于自动绑定:

```text
worker_cpu_affinity auto 01010101;
```

{% hint style="info" %}
这个命令只能用与 FreeBSD 和 Linux 系统.
{% endhint %}
{% endtab %}
{% endtabs %}

### worker\_priority

{% tabs %}
{% tab title="语法" %}
**worker\_priority** number;
{% endtab %}

{% tab title="默认值" %}
worker\_priority 0;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
定义了工作进程的计划优先级, 如同 `nice` 命令实现的一样: 负数的 `number` 代表了更高的优先级. 数字只允许从 -20 到 20. 例如:

```text
worker_priority -10;
```
{% endtab %}
{% endtabs %}

### worker\_processes

{% tabs %}
{% tab title="语法" %}
**worker\_processes** number \| auto;
{% endtab %}

{% tab title="默认值" %}
worker\_processes 1;
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
定义了工作进程的数量. 最合适的数量取决于很多因素, 包括但不限于:

* CPU 的核心数
* 存储数据的硬盘数量
* 负载模式

当有疑问的时候, 设置为 可用的 CPU 核心数会是一个好的选择.

auto 会尝试自动选择一个合适的值\(1.3.8+ , 1.2.5+\).
{% endtab %}
{% endtabs %}

### worker\_rlimit\_core

{% tabs %}
{% tab title="语法" %}
**worker\_rlimit\_core** size;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
更改工作进程的核心文件 \(RLIMIT\_CORE\) 的最大大小的限制. 用于在不重新启动主进程的情况下增加限制. 
{% endtab %}
{% endtabs %}

### worker\_rlimit\_nofile

{% tabs %}
{% tab title="语法" %}
**worker\_rlimit\_nofile** number;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
更改工作进程打开文件的最大数量 \(RLIMIT\_nofile\) 的限制。用于在不重新启动主进程的情况下增加限制.
{% endtab %}
{% endtabs %}

### worker\_shutdown\_timeout

{% tabs %}
{% tab title="语法" %}
**worker\_shutdown\_timeout** time;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
设置软关闭工作进程的超时值. 当超过了 time 设置的值, Nginx 会尝试主动关闭所有当前开启了的连接\(1.11.11+\).
{% endtab %}
{% endtabs %}

### worker\_directory

{% tabs %}
{% tab title="语法" %}
**worker\_directory** directory;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
定义了工作进程的当前工作目录. 这个指令主要用于开发核心文件时, 工作进程拥有指定目录的写入权限.
{% endtab %}
{% endtabs %}

