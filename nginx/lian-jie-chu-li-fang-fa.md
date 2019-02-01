---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 连接处理机制

Nginx支持多种连接处理机制. 具体哪些机制可用取决于当前的系统类型. 在支持多种机制的系统下, Nginx会自动选择最有效的机制. 然而如果你需要的话, 你可以通过 `use` 指令来选择要是有哪个处理机制.

Nginx可以支持以下机制:

* select - 标准的机制, 在那些缺乏更加有效的连接机制的系统上, 会自动构建该模块. 通过配置参数 `--with-select_module` 和 `--without-select_module` 可以强制构建/不构建模块.
* poll - 标准的机制, 在那些缺乏更加有效的连接机制的系统上, 会自动构建该模块. 通过配置参数 `--with-poll_module` 和 `--without-poll_module` 可以强制构建/不构建模块.
* kqueue - 在 **FreeBSD 4.1+**, **OpenBSD 2.9+**, **NetBSD 2.0**, **CentOS** 上更为高效的机制.
* epoll - 用于 **Linux 2.6+**.
* /dev/poll - 用于 **Solaris 7 11/99+**, **HP/UX 11.22+**, **IRIX 6.5.15+**, **Tru64 UNIX 5.1A+**.
* eventport - 用于 **Solaris 10+**, 因为一些总所周知的原因, 我们更加推荐使用/dev/poll

