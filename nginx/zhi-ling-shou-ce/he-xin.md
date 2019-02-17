---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 核心

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



