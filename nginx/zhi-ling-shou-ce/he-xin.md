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

{% tab title="Second Tab" %}
client\_body\_buffer\_size 8k \| 16k;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
可以设置http请求体的缓冲区大小. 如果请求体的大小大于缓冲区的大小, 那么整个请求体或请求体的一部分将会写入临时文件内. 默认情况下, 缓冲区大小等于两个内存页大小. 因此在 x86 \(即32位\) 系统中, 缓冲区大小为 8k, 而在 x86-64 \(即64位\) 系统中, 通常为 16k.
{% endtab %}
{% endtabs %}

