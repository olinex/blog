---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# HTTP负载均衡

## 概述

负载均衡在多应用实例情况下是一种常见的技术, 它优化资源使用, 最大化吞吐量, 减少延迟, 确保一定的容错率.

## 代理HTTP流量至一组服务器

为了使Nginx能将HTTP流量代理至指定的一组服务器, 首先你需要在`http`环境内通过`upstream`指令定义一个服务器组. 

在组内通过`server`指令定义服务器 \( 不要和定义虚拟服务器的`server`块指令混淆 \) . 例如, 以下配置定义了一个名叫 `backend` 的组, 它由三个服务配置组成 \( 它们或许会分解成更多的真实的服务器 \):

```text
http {
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
}
```

为了将请求传入服务器组内, 需要将组定义在`proxy_pass`命令内, 可选的命令还有:

* fastcgi\_pass
* memcached\_pass
* scgi\_pass
* uwsgi\_pass

在下面的示例, 虚拟服务器将所有的请求转发至`backend`上游代理服务器组:

```text
http {
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
    
    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

服务器组有三个服务器组成, 其中两个为同一个应用, 第三个为备用服务器. 因为在`upstream`没有指定负载均衡的算法, 因此Nginx使用了默认的轮询调度算法.

## 选择负载均衡算法

开源的Nginx支持四种负载均衡算法, Nginx扩展额外支持两种:

### 轮询调度

请求均匀的被传递给服务器, 可以通过参数 `weight` 指定比重. Nginx默认使用该算法.

```text
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
```

### 最少连接

请求将会被发送给当前连接数最少的服务器, 可以通过参数 `weight` 指定比重.

```text
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

### IP哈希

请求将根据浏览器的IP地址发送给指定的服务器. 在下面这个示例中, ipv4 地址的前三个八位字节或整个 ipv6 地址用于计算哈希值, 这个方法保证来自同一个地址的请求到达同一个服务器, 除非它不再可用.

```text
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

如果其中一台服务器需要从负载均衡服务器组内临时去除, 可以通过 `down` 参数将其去除, 这将保证浏览器IP的当前hash值相同. 发送至该服务器的请求将自动转发至组内的下一个服务器.

```text
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
    server backend2.example.com down;
}
```

### 通用哈希

请求可以根据用户指定键值哈希值来发送请求给指定的服务器, 键值可以为字符串, 变量,或者组合表达式. 例如, 键值可以为来源IP地址和端口的组合, 或者如下的URI:

```text
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com;
    server backend2.example.com;
}
```

可选的 `consistent` 参数将启用 ketama 一致性哈希算法, 根据用户定义的哈希键值, 请求将均匀地转发至所有上游服务器,  如果一个上游服务器被新增或删除, 只有很少的一部分键值需要重新映射, 从而最大限度地减少负载均衡和其他应用程序的缓存丢失.

### 最小延迟 \( Nginx扩展 \)

Nginx扩展将会选择平均延迟和当前连接数最少的服务器来接收请求, 最小平均延迟是基于`least_time`指令的参数来计算的:

* header - 从服务器接收到第一个字节开始计算
* last\_byte - 从服务器返回完整响应开始计算
* last\_byte inflight - 从服务器返回完整响应开始计算, 包括不完整的请求

```text
upstream backend {
    least_time header;
    server backend1.example.com;
    server backend2.example.com;
}
```

### 随机

每个请求将会被随机传递给服务器. 如果设置了 `two` 参数, Nginx首先会根据比重随机选择两个服务器, 然后会根据后续设置的算法进行选择, 后续算法可以为:

* least\_conn - 当前连接数最少
* least\_time=header - 从服务器接收到第一个字节的最小平均耗时\($upstream\_header\_time\)
* least\_time=last\_type - 从服务器接收到完整响应的最小平均耗时\($upstream\_header\_time\)

```text
upstream backend {
    random two least_time=last_byte;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
    server backend4.example.com;
}
```

对于多个负载均衡器将请求传递到同一组后端的分布式场景, 应该使用随机负载均衡算法. 对于负载均衡器能获取所有请求的场景, 应该是用其他的负载均衡算法, 例如轮询调度, 最小连接和最小延迟.

{% hint style="info" %}
当使用除了轮询调度之外的负载均衡算法时, 应该将指令置于所有server之上
{% endhint %}

## 服务器比重

在默认情况下, 使用轮询调度在服务器组内分发请求时, 将参考他们的比重. 使用 `weight` 参数配置服务器的比重, 缺省默认为1:

```text
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

在上述示例中, backend1.example.com 的比重为5, 其他两个服务器的比重默认为1, 但由于 192.0.0.1 被标志为备用服务器, 因此除非其余两台服务器均不可用, 否则将不会收到请求. 根据以上比重配置, 每6个请求, 5个将会被发送至 backend1.example.com, 剩下一个 被发送至 backend2.example.com.

## 服务器懒启动

服务懒启动的功能可防止最近恢复的服务器被请求连接压垮, 以免响应超时导致被Nginx再次标记为服务故障.

通过Nginx扩展, 懒启动可以使上游服务器在恢复或启用后, 将其比重从0逐步恢复至指定值. 在`server`指令后添加 `slow_start` 参数:

```text
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

Nginx扩展将会在指定时间 \( 此处为30秒 \) 逐渐将连接数增加值正常值.

{% hint style="info" %}
如果一个服务器组内只有一个服务器, server指令后的 max\_fails, fails\_timeout, slow\_start 都将被忽略, 并且服务器不会被标记为不可用
{% endhint %}

## 启用持续化会话

Nginx扩展的持续化会话将会发现用户的 session 并根据标记的 session 将请求发送至同一上游服务器

Nginx扩展支持三种持续化会话方法. 他们通过sticky 指令配置. 

### cookie

Nginx扩展会为没有会话 cookie 的响应添加会话 cookie, 并使用发送响应的服务器作为标识, 客户端下一次请求若包含会话 cookie, Nginx将会根据会话 cookie 内的服务器标识将请求发送至上次响应的服务器:

```text
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

在上述示例中, `srv_id` 参数将被设置为cookie的名称. `expires` 参数定义了浏览器端将会保存 cookie 多长时间 \( 这里是1小时 \). `domain` 参数定义了 cookie 的所在域, `path` 参数定义了 cookie 的所在路径, 除了 `srv_id` 参数外, 以上参数均为可选. 这是最简单的持续化会话方法.

### 路由

Nginx扩展提供了路由方法, 在客户端发出第一个请求时将其分配给指定路由. 所有后续请求都`server`指令的`route`参数进行比较. 请求的路由信息来源于URI或者cookie:

```text
upstream backend {
    server backend1.example.com route=a;
    server backend2.example.com route=b;
    sticky route $route_cookie $route_uri;
}
```

### cookie学习

Nginx扩展首先在请求和响应内搜索会话标识. 接着Nginx扩展"学习"那个上游服务器对应于那个会话标识. 一般情况下, 这些标识被定义在HTTP cookie内, 如果请求包含的会话标识已经被学习了, Nginx扩展会将请求转发至相应的服务器:

```text
upstream backend {
   server backend1.example.com;
   server backend2.example.com;
   sticky learn
       create=$upstream_cookie_examplecookie
       lookup=$cookie_examplecookie
       zone=client_sessions:1m
       timeout=1h;
}
```

在示例中, 上游服务器将会在响应内创建一个名为EXAMPLECOOKIE的 cookie, 作为会话标识.

必要的 `create` 参数定义了创建新会话的变量名. 在示例中, 新会话通过上游服务器设置的EXAMPLECOOKIE cookie 来创建.

必要的 `lookup` 参数定义了如何搜索已存在的会话. 在示例中, 将会在 客户端发送来的请求内 搜索 EXAMPLECOOKIE cookie.

必要的 `zone` 参数定义了存储所有会话信息的共享内存区域. 在示例中, 区域被命名为 client\_sessions, 其大小为 1mb.

这是一个更加精细复杂的持续化会话方法, 相比于前两个方法. cookie学习不需要在客户端保存任何cookie, 所有的信息都将会被保存在服务器端的共享内存区域内.

如果在集群中有多个Nginx实例使用了 cookie学习 的方法, 则可以在以下条件下同步其共享内存区域的内容:

* 共享内存区域的名称相同
* `zone_sync` 功能在所有的Nginx实例内启用
* sync 参数被启用

```text
upstream backend {
   server backend1.example.com;
   server backend2.example.com;
   sticky learn
       create=$upstream_cookie_examplecookie
       lookup=$cookie_examplecookie
       zone=client_sessions:1m
       timeout=1h;
       sync;
}
```

## 控制连接数

通过Nginx扩展, 通过使用 `max_conns` 参数限制上游服务器的最大连接数量.

如果上游服务器达到了最大连接数, 后续的连接将会被存入队列以在未来继续处理. 可以通过 `queue` 指令设置队列的最大保存请求数:

```text
upstream backend {
    server backend1.example.com max_conns=3;
    server backend2.example.com;
    queue 100 timeout=70;
}
```

如果队列已经饱和, 并且上游服务器依然无法在 `timeout` 参数指定的时间内接收请求, 客户端将会收到一个错误响应.

{% hint style="info" %}
如果有空闲的长连接在别的工作进程, max\_conns将会被忽略. 这会导致, 在多个工作进程间共享内存时, 所有连接的总数将可能会超过max\_conns设置的数量
{% endhint %}

## 被动地运行状况检查

当Nginx认为一个服务器不再可用, 它会临时停止再向他发送请求, 直到Nginx认为该服务器重新可用. 以下 `server` 指令的参数定义了Nginx判断服务器是否可用的条件:

* max\_fails - 当服务器连续尝试请求失败指定的次数后, Nginx会将其视为不再可用
* fail\_timeout - 当服务器因为连续请求失败被视为不再可用后, Nginx会在指定的时间后认为服务器再次可用

Nginx默认尝试请求1次, 可用状态10秒后重置. 因此如果一个服务器不能接收请求或不能返回响应, 则Nginx会立刻将其视为10秒内不可用: 

```text
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com max_fails=2;
}
```

## 主动地运行状况检查

以下是Nginx扩展内更加精细复杂的服务器可用性检查功能.

你可以通过周期性地发送特殊的请求至每一台服务器并检查它们的响应是否满足一定的条件, 来主动监控服务器运行状况.

为了开启运行状况地监控, 你需要在上游服务器组的 `location` 块内添加 `health_check` 指令, 上游服务器组内必须包含 `zone` 指令来定义一个保存服务器运行状况信息的公共内存区域:

```text
http {
    upstream backend {
        zone backend 64k;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
        server backend4.example.com;
    }
    
    server {
        location / {
            proxy_pass http://backend;
            health_check;
        }
    }
}
```

这个配置定义了一个名叫`backend`的上游服务器组和一个虚拟服务器, 虚拟服务器将所有地请求转发给了`backend`上游服务器组.

`zone` 指令定义了一个内存区域, 可以在不同工作进程间共享, 并保存服务器组的运行状况. 这使得不同的工作进程能够对上游服务器组返回的响应进行统一地计数.

`health_check` 指令在没有任何参数地情况下有以下地默认配置: 每五秒钟Nginx扩展将会发送一个请求给服务器组内的所有服务器. 如果任何错误或超时发生 \( 或者响应的状态码不为 2XX 或 3XX \) , 该服务器的运行状况则被认为是不正常的, Nginx扩展将会停止向其转发客户端地请求, 直到它再次通过了检查.

```text
location / {
    proxy_pass http://backend;
    health_check interval=5 fails=3 passes=2 uri=/some/path;
}
```

`health_check` 指令可以通过改变参数来改变检查行为. `interval` 参数将每次运行状况检查地间隔设置为5秒, `fails`=3 参数意味着运行状况为不健康的条件为连续检查失败3次, 而不是默认地1次. 通过设置 `pass` 参数, 服务器需要连续通过2次运行状况检查, 才会被认为再次运行正常, 而不是默认的只需要通过一次检查.

通过 `uri` 参数定义检查的请求\( 示例中的 /some/path \)路径来替换默认的 "/". 完整路径通过组装上游服务器ser`v`er指令的IP地址或域名和uri参数来形成. 例如, 在`backend`服务器组的第一个服务器, 运行状况检查地请求将会被发送至http://backend1.example.com/some/path.

最后, 你可以自定义Nginx扩展的运行状况判断条件. 条件通过 `match` 命令设置, 然后通过 `health_check` 指令的 `match` 参数指定.

```text
http {
    match server_ok {
        status 200-399;
        body !~ "maintenance mode";
    }
    
    server {
        location / {
            proxy_pass http://backend;
            health_check match=server_ok;
        }
    }
}
```

在上述示例, 要通过运行状况检查, 响应的状态码必须从200至399之间, 并且请求体不能和正则表达式相匹配. 

Nginx扩展可以通过 `match` 指令检查响应的状态码是否处在一定地范围内, 是否包含指定地响应头, 响应体是否匹配指定地正则表达式. `match` 指令可以包含一个状态码条件, 一个响应体条件和多个响应头条件. 响应必须满足 `match` 块内所有地条件才能通过状况检查.

在下面地示例中, `match` 块内的条件要求响应状态码必须为200, 响应类型头 `Cotent-Type` 必须为 `text/html`, 并且响应体内必须包含 "Welcome to nginx!" 字符串:

```text
match welcome {
    status 200;
    header Content-Type = text/html;
    body ~ "Welcome to nginx!";
}
```

在下面地示例中, 使用了排除符号 \( ! \), 响应不能为 301, 302, 303, 307, 且响应头内不能包含 Refresh头:

```text
match not_redirect {
    status ! 301-303 307;
    header ! Refresh;
}
```

运行状况检查同样可以作用于 FastCGI, memcached, SCGI 和 uwsgi 协议.

## 在工作进程间共享内存

如果在 `upstream` 块内没有包含 `zone` 指令, 每个工作进程都将维护一份自己的服务组配置和相关计数器. 计数器包含了每个组内服务器的连接次数和连接失败次数. 因此, 服务器组的配置不能动态调整. 

当在 `upstream` 块内包含了 `zone` 指令, 上游服务器地配置将会被保存内存中并在所有地工作进程中共享. 这个方案是可以动态配置的, 因为工作进程获取了相同地服务器组设置并使用了相同地计数器. 主动运行状况检查和动态上游服务器组配置要求必须有 `zone` 指令. 不过, 上游服务器组的其他功能都将因此获益.

例如, 如果服务器组的配置不能共享, 每个工作进程都需要管理自己地失败请求计数器 \( 通过 `max_fails` 参数设置 \). 在这种情况下, 每个请求都将只能访问一个服务器. 当器重一个服务器处理请求并失败, 其他工作进程将对此一无所知. 当其中一些服务器被认为不再可用时, 其他的工作进程会继续向这些服务器发送请求. 因此要使服务器被明确地认为不可用, 在由 `fail_timeout` 参数设置地时间范围内, 失败地尝试次数必须等于 `max_fails` 乘于工作进程的个数. 或者说, `zone` 指令保证了预期地行为.

同样的, 没有 `zone` 指令, 至少在低负载地情况下, 最小连接负载均衡算法将不会按照预期执行. 这个方法将转发请求至当前连接数最少地服务器. 如果服务器组的配置不能共享, 每个工作进程使用他们自己的连接数计数器并可能将请求发送给一个刚接收了其他工作进程的请求的服务器. 或者, 你可以通过增加请求来减少这个影响. 在高负载的情况下, 请求将被均匀地发送给各个服务器, 最小连接负载均衡算法将会按照预计的情况执行.

## 设置内存共享区域地大小

事实上我们不可能推荐一种理想的内存大小, 因为各种使用情况都不尽相同. 内存所需的大小取决于我们使用的功能 \( 例如 会话持续化, 运行状况检查,  DNS解析 \) 和使用上游服务器组.

例如, 使用路由模式的会话持续化方法和单个运行状况检查, 一个256KB 的内存区域可以容纳以下几种上游服务器的信息:

* 128个通过IP地址和端口指定的服务器
* 88个通过域名和端口指定的服务器, 且域名只能解析一个IP地址
* 12个通过域名和端口指定的服务器, 域名可以解析多个IP地址

## 通过DNS配置负载均衡

通过DNS, 可以在运行过程中, 修改服务器组的配置.

在上游服务器组内, 通过 server 指令定义了域名的服务器, Nginx扩展可以监视对相应 DNS 记录中的 ip 地址列表的更改, 并且可以自动地将修改应用于上游服务器的负载均衡配置, 而不需要重新启动. 这可以通过在 `http` 环境内增加 `resolver` 指令并在 `server` 指令上增加 `resolve` 参数来实现:

```text
http {
    resolver 10.0.0.1 valid=300s ipv6=off;
    resolver_timeout 10s;
    server {
        location / {
            proxy_pass http://backend;
        }
    }
    upstream backend {
        zone backend 32k;
        least_conn;
        # ...
        server backend1.example.com resolve;
        server backend2.example.com resolve;
    }
}
```

在上述示例, 在 `server` 指令内添加 `resolve` 指令, 将让Nginx扩展周期性地解析 backend1.example.com 和 backend2.example.com 的IP地址列表.

`resolver` 指令定义了保存IP地址的DNS服务器, Nginx扩展会向其发送请求 \( 10.0.0.1 \). 默认情况下, Nginx扩展会按照DNS记录内设置的生命周期 \( TTL \) 来重新解析DNS记录, 但你可以通过 `valid` 参数重写这个定义, 在上述示例中, 重新解析地间隔为300秒.

可选的 `ipv6=off` 参数意味着只有 IPv4 地址会被用于负载均衡, 否则默认 IPv4 和 IPv6 都将会解析.

如果一个域名对应解析出多个IP地址, 这些IP地址都将被保存在上游服务器组的配置中并进行负载均衡. 在我们的示例中, 使用了最小连接负载均衡算法. 如果服务器地IP地址地列表被改变了, Nginx扩展将立刻使用新的地址进行负载均衡.

## 通过Nginx扩展API进行负载均衡动态配置

通过Nginx扩展, 上游服务器组地配置可以通过Nginx扩展API进行动态修改. 配置命令可以作用组内地所有或者部分服务器, 修改部分服务器的参数, 增加或删除服务器.

