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
{% endtab %}
{% endtabs %}



