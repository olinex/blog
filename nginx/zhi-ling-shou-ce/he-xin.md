---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 核心指令

## 指令

**absolute\_redirect**

{% tabs %}
{% tab title="语法" %}
**absolute\_redirect** on \| off;
{% endtab %}

{% tab title="默认值" %}
absolute\_redirect on;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
如果设置为off, Nginx将会发起相对原路径的跳转\(1.11.8\)
{% endtab %}
{% endtabs %}

#### alias

{% tabs %}
{% tab title="语法" %}
**alias** path;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
location
{% endtab %}

{% tab title="说明" %}
定义一个特定路径的别名. 例如:

```text
location /i/ {
    alias /data/w3/images/;
}
```

当请求 `/i/top.gif` 时, `/data/w3/images/top.gif` 将会被返回.

`path` 参数可以包含除了 `$document_root` 和  `$realpath_root` 的变量.

如果 `alias` 定义在一个路径参数为正则表达式的上下文内, 且包含需要匹配的模式, 则 `alias` 的参数可以包含匹配到的内容, 例如:

```text
location ~ ^/users/(.+\.(?:gif|jpe?g|png))$ {
    alias /data/w3/images/$1;
}
```

当路径参数与 alias 参数的结尾匹配时:

```text
location /images/ {
    alias /data/w3/images/;
}
```

最好使用 root 指令:

```text
location /images/ {
    root /data/w3;
}
```
{% endtab %}
{% endtabs %}

####  **chunked\_transfer\_encoding**

{% tabs %}
{% tab title="语法" %}
 **chunked\_transfer\_encoding** on \| off;
{% endtab %}

{% tab title="默认值" %}
chunked\_transfer\_encoding on;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
尽管标准要求分块传输编码必须开启, 但某些应用依然有可能不支持, 因此可以通过 `chunked_transfer_encoding` 指令关闭这个功能.
{% endtab %}
{% endtabs %}

####  **client\_body\_buffer\_size**

{% tabs %}
{% tab title="语法" %}
**client\_body\_buffer\_size** size**;**
{% endtab %}

{% tab title="默认值" %}
client\_body\_buffer\_size 8k \| 16k;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
可以设置http请求体的缓冲区大小. 如果请求体的大小大于缓冲区的大小, 那么整个请求体或请求体的一部分将会写入临时文件内. 默认情况下, 缓冲区大小等于两个内存页大小. 因此在 x86 \(即32位\) 系统中, 缓冲区大小为 8k, 而在 x86-64 \(即64位\) 系统中, 通常为 16k.
{% endtab %}
{% endtabs %}

#### client\_body\_in\_file\_only

{% tabs %}
{% tab title="语法" %}
**client\_body\_in\_file\_only** on \| clean \| off;
{% endtab %}

{% tab title="默认值" %}
client\_body\_in\_file\_only off;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
配置 Nginx 是否要将整个客户端请求保存在一个文件当中. 这个命令适合在调试的时候使用.

当该指令的值设置为 `on` 的时候, 临时文件在处理完请求后不会呗删除.

当该指令的值设置为 clean 的时候, 临时文件会在处理请求的时候被保留, 直到处理结束.
{% endtab %}
{% endtabs %}

#### client\_body\_in\_single\_buffer

{% tabs %}
{% tab title="语法" %}
**client\_body\_in\_single\_buffer** on \| off;
{% endtab %}

{% tab title="默认值" %}
client\_body\_in\_single\_buffer off;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
配置 Nginx 是否将整个客户端请求体保存在一个单一缓存内. 当时用 `$request_body` 变量的时候, 建议开启这个功能, 来减少复制操作的次数.
{% endtab %}
{% endtabs %}

#### client\_body\_temp\_path

{% tabs %}
{% tab title="语法" %}
**client\_body\_temp\_path** path \[level1, \[level2, \[level3\]\];
{% endtab %}

{% tab title="默认值" %}
client\_body\_temp\_path client\_body\_temp;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
定义一个文件夹路径, 来保存临时文件, 这些临时文件保存了客户端请求体. 最多可以在指定文件路径下使用三级子目录结构. 例如, 在以下配置:

```text
client_body_temp_path /spool/nginx/client_temp 1 2;
```

临时文件的路径将会如下所示:

```text
/spool/nginx/client_temp/7/45/000001234567
```
{% endtab %}
{% endtabs %}

#### client\_body\_timeout

{% tabs %}
{% tab title="语法" %}
**client\_body\_timeout** time;
{% endtab %}

{% tab title="默认值" %}
client\_body\_timeout 60s;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
定义读取客户端请求体的超时值. 这个超时值限制的不是整个请求体的传输, 而是限制传输过程中两个请求包读取操作的时间间隔. 如果客户端在这个时间间隔内没有传输任何东西, 请求会被中断并返回 408 错误.
{% endtab %}
{% endtabs %}

#### client\_header\_buffer\_size

{% tabs %}
{% tab title="语法" %}
**client\_header\_buffer\_size** size;
{% endtab %}

{% tab title="默认值" %}
client\_header\_buffer\_size 1k;
{% endtab %}

{% tab title="上下文" %}
http, server
{% endtab %}

{% tab title="说明" %}
设置读取客户端请求头的缓存区大小. 对于大多数请求而言, 1k 字节的缓存已经足够了. 但是如果请求体包含了长 cookies, 或者请求来源于 WAP 客户端, 请求头可能会大于 1k 字节.
{% endtab %}
{% endtabs %}

#### client\_header\_timeout

{% tabs %}
{% tab title="语法" %}
**client\_header\_timeout** timeout;
{% endtab %}

{% tab title="默认值" %}
client\_header\_timeout 60s;
{% endtab %}

{% tab title="上下文" %}
http, server
{% endtab %}

{% tab title="说明" %}
定义了读取客户端请求头的超时值. 如果客户端没有将整个请求头在超时值内传输完, 请求将会被终端并返回 408 错误.
{% endtab %}
{% endtabs %}

#### client\_max\_body\_size

{% tabs %}
{% tab title="语法" %}
**client\_max\_body\_size** size;
{% endtab %}

{% tab title="默认值" %}
client\_max\_body\_size 1m;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
通过判断请求头部分内设置的 `Content-Length` , 限制客户端请求体的最大允许大小. 如果请求的大小超过了设置的值, 413 错误将会返回客户端. 请注意, 浏览器并不能正确地显示这个错误, 将 `size` 设置为0, 将会关闭这个检查.
{% endtab %}
{% endtabs %}

#### connection\_pool\_size

{% tabs %}
{% tab title="语法" %}
**connection\_pool\_size** size;
{% endtab %}

{% tab title="默认值" %}
connection\_pool\_size 256 \| 312;
{% endtab %}

{% tab title="上下文" %}
http, server
{% endtab %}

{% tab title="说明" %}
可以优化每个请求的内存分配. 这个配置对性能的影响很小, 因此不应该被使用. 默认情况下, 32位系统为 256 字节, 64位系统为 512 字节.

对于 1.9.8 之前的版本, 所有的系统均为 256 字节.
{% endtab %}
{% endtabs %}

#### default\_type

{% tabs %}
{% tab title="语法" %}
default\_type mime-type;
{% endtab %}

{% tab title="默认值" %}
default\_type text/plain;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
响应的默认 MIME 类型. 可以通过 `types` 命令可以设置文件名的后缀和 MIME 类型的映射.
{% endtab %}
{% endtabs %}

#### disable\_symlinks

{% tabs %}
{% tab title="语法" %}
**disable\_symlinks** on \| off \| if\_not\_owner \[from=part\];
{% endtab %}

{% tab title="默认值" %}
disable\_symlinks off;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
设置当 Nginx 打开文件时, 遇到符号链接的处理方式:

* off - 允许读取符号链接所代表的文件, 并不会检查权限
* on - 在打开文件时, 路径上有任何一个文件或文件夹为符号链接, 则拒绝访问文件
* if\_not\_owner - 再打开文件时, 路径上有任何一个文件或文件夹为符号链接, 且符号链接文件的所有者和所代表的文件的所有者不同时, 则拒绝访问文件

当我们将符号链接的处理方式设置为 `on` 或者 `if_not_owner` 的时候, 文件路径上所有的文件或文件夹都将会被检查. 若我们希望不要从路径的起始路径开始检查, 可以通过设置 `from=part` 参数来实现.

在这种情况下, 符号链接只会从指定的文件或文件夹处开始检查. 如果设置的值不是符号链接的起始路径, 则会检查整个路径. 如果设置的值匹配整个路径, 则不会进行检查. 起始路径的值可以包含 Nginx 变量:

```text
disable_symlinks on from=$document_root;
```

这个指令只会在拥有 openat\(\) 和 fstatat\(\) 接口的系统上有效. 例如 FreeBSD, Linux, Solaris.

on 和 if\_not\_owner 会增加进程的开销.

{% hint style="info" %}
在没有所需接口的系统上使用这个命令, 需要工作进程拥有指定文件的读权限.
{% endhint %}
{% endtab %}
{% endtabs %}

#### error\_page

{% tabs %}
{% tab title="语法" %}
**error\_page** code... \[=\[response\]\] uri;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
http, server, location, if in location
{% endtab %}

{% tab title="说明" %}
定义具体错误码所显示的页面. `uri` 参数可以包含 Nginx 变量. 例如:

```text
error_page 404                    /404.html;
error_page 500 502 503 504        /50x.html;
```

这会导致内部重定向到指定的 `uri` ,并且客户端的请求方法将变为 `GET` \( 除了 `GET` 和 `HEAD` 方法以外 \).

我们还能通过 `=response` 语法来修改响应的响应码:

```text
error_page 404 =200 /empty.gif;
```

如果错误响应是通过代理服务器或 FastCGI/uwsig/SCGI/gRPC 服务器来处理的, 并且服务器会返回不同的响应码, \(例如 200, 302, 401, 404\), 可以返回真实的响应码:

```text
error_page 404 = /404.php;
```

如果不需要在内部重定向的时候修改 URI 和 http, 可以将错误处理传递到指定的命名路径上下文内:

```text
location / {
    error_page 404 = @fallback;
}

location @fallback {
    proxy_pass http://backend;
}
```

{% hint style="info" %}
如果 uri 处理过程中出现了错误, 最后出现的错误状态码将会返回给客户端.
{% endhint %}

还可以使用 URL 重定向来处理错误:

```text
error_page 403            http://example.com/forbidden.html;
error_page 404 =301       http://example.com/notfound.html;
```

在这种情况下, 响应码 302 将会被返回给客户端. 响应码只能被修改为重定向状态码 \(301, 302, 303, 308\).

{% hint style="info" %}
307 状态码在 Nginx 1.1.16 和 1.0.13 之前将不会被重定向, 308 状态码在 Nginx 1.13.0 之前将不会被重定向
{% endhint %}

如果当前上下文没有定义 error\_page, 将会从上级上下文中继承.
{% endtab %}
{% endtabs %}

#### etag

{% tabs %}
{% tab title="语法" %}
**etag** on \| off;
{% endtab %}

{% tab title="默认值" %}
etag on;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
开启或关闭静态资源自动添加 ETag 响应头的功能\(1.3.3+\).
{% endtab %}
{% endtabs %}

#### http

{% tabs %}
{% tab title="语法" %}
**http** {...}
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
main
{% endtab %}

{% tab title="说明" %}
创建一个http虚拟服务器的上下文环境.
{% endtab %}
{% endtabs %}

#### if\_modified\_since

{% tabs %}
{% tab title="语法" %}
**if\_modified\_since** off \| exact \| before;
{% endtab %}

{% tab title="默认值" %}
if\_modified\_since exact;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
定义了如何比较请求头内的 `If-Modified-Since` 设置的时间和响应的修改时间:

* off - 不进行比较\(0.7.34+\)
* exact - 需要完全比配
* before - 响应的修改时间必须小于等于请求头内 `If-Modified-Since` 设置的时间.

 
{% endtab %}
{% endtabs %}

#### ignore\_invalid\_headers

{% tabs %}
{% tab title="语法" %}
**ignore\_invalid\_headers** on \| off;
{% endtab %}

{% tab title="默认值" %}
ignore\_invalid\_headers on;
{% endtab %}

{% tab title="上下文" %}
http, server
{% endtab %}

{% tab title="说明" %}
控制是否忽略名称无效的请求头, 有效的请求头名称必须为英文字符/数字/连字符/下划线\(下划线需要通过 `underscores_in_headers` 指令来控制\).
{% endtab %}
{% endtabs %}

#### internal

{% tabs %}
{% tab title="语法" %}
**internal**;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
location
{% endtab %}

{% tab title="" %}
控制指定的路径是否只能被内部请求访问. 对于外部请求, 客户端会收到 404 错误, 内部请求为:

* 通过 `error_page`, `index`, `random_index`, `try_files` 指令重定向的请求
* 通过 来自上游服务器的 `X-Accel-Redirect` 响应头重定向的请求
* 通过 `rewrite` 指令导向的请求
* 通过 `mirror` 指令, 响应附加, 请求验证, include virtual 发起的子请求

例如:

```text
error_page 404 /404.html;

location = /404.html {
    internal;
}
```

为了防止循环重定向, 每个请求只能有10次的内部重定向. 如果超过了这个限制, 将会返回 500 错误. 在这种情况下, "rewrite or internal redirection cycle" 信息将会被打印到错误日志中. 
{% endtab %}
{% endtabs %}

