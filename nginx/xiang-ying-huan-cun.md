---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 响应缓存

这一节描述了如何配置和启用缓存功能, 它能缓存被代理服务器的响应.

## 概述

当缓存被启动后, Nginx扩展会将响应保存在硬盘内, 并且当浏览器请求时将缓存直接返回, 而不需要每次都转发请求来获取相同的内容.

## 启用响应缓存

要启用缓存, 需要将 `proxy_cache_path` 指令放入在 `http` 的顶层环境内. 指令的第一个参数为必填参数, 它是保存响应缓存地本地系统路径, 并且必填的 `keys_zone` 参数定义了共享内存区域的名称和大小, 共享内存区域保存了缓存的元信息:

```text
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m;
}
```

接着, 将 proxy\_cache 指令置于你想要缓存服务器响应的环境中 \( 可以为协议环境, server, location \), 指明通过 `keys_zone` 参数定义的共享内存区域名称 \( 在这里为 one \)

```text
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m;
    server {
        proxy_cache mycache;
        location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```

注意, `keys_zone` 参数指定的大小并不限制响应缓存数据的总大小. 响应缓存本身是保存在文件系统中. 为了限制响应缓存的大小, 将 `max_size` 参数添加到 `proxy_cache_path` 指令内.

{% hint style="info" %}
但要注意, 缓存可能会暂时超过这个限制
{% endhint %}

## 缓存时涉及的进程

在缓存时, 涉及了两个新的进程:

缓存管理进程 - 会周期性的启动并检查缓存状态. 如果缓存的大小超过了在 `proxy_cache_path` 指令中 `max_size` 参数的限制, 管理进程会删除访问次数最少的一批数据. 如上面所说的, 缓存数据的大小有可能会在缓存管理进程的启动期间临时地超过限制.

缓存加载进程- 只会在Nginx启动时运行一次. 它会将先前缓存数据的元信息写入到共享内存区域. 一次性加载所有的缓存数据会消耗所有的资源, 导致Nginx在启动后的几分钟内性能下降. 为了避免这种情况, 通过在 `proxy_cache_path` 指令内增加以下参数来迭代式地加载缓存:

* loader\_threshold - 加载缓存的持续时间, 以毫秒为单位, 默认为200毫秒
* loader\_files - 每次加载的最大缓存数, 默认为100个
* loader\_sleeps - 每次加载缓存的间隔时长, 已毫秒为单位, 默认为50毫秒

在下面的实例中, 每次迭代300毫秒或直到200个缓存被加载完:

```text
proxy_cache_path /data/nginx/cache keys_zone=one:10m loader_threshold=300 loader_files=200;
```

## 指定要缓存的请求

默认情况下, Nginx扩展会将第一次响应保存起来, 这些响应的HTTP请求方法为 `GET` 和 `HEAD`. Nginx扩展会将请求的字符串作为缓存的键. 如果对于请求的来说, 缓存响应的键值相同, Nginx扩展会将缓存响应发送给客户端. 你可以通过在 http, server , location 块内添加多个指令来控制哪个响应应该被缓存.

通过添加 proxy\_cache\_key 指令, 改变将请求作为计算键值的行为:

```text
proxy_cache_key "$host$request_uri$cookie_user";
```

为了使具有相同键值的请求在一次请求次数后才被缓存起来, 可以添加 `proxy_cache_min_uses` 指令:

```text
proxy_cache_min_uses 5;
```

除了 GET 和 HEAD 请求的响应会被缓存, 可以通过在 `proxy_cache_methods` 指令添加别的HTTP方法:

```text
proxy_cache_methods GET HEAD POST;
```

## 限制或禁用缓存

默认情况下, Nginx能够缓存无限的响应. 只有在缓存的大小超过了最大限制,

By default, responses remain in the cache indefinitely. They are removed only when the cache exceeds the maximum configured size, and then in order by length of time since they were last requested. You can set how long cached responses are considered valid, or even whether they are used at all, by including directives in the `http {}`, `server {}`, or `location {}` context:

