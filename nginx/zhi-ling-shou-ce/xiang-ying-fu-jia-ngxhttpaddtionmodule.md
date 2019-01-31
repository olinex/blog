---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 响应附加\(ngx\_http\_addtion\_module\)

ngx\_http\_addtion\_module 模块是一种过滤器, 可以在响应体的内增加文本. 这个模块并没有默认构建, 你可以通过 --with-http\_addtion\_module 配置参数添加模块.

## 示例

```text
location / {
    add_before_body /before_uri;
    add_after_body /after_uri;
    addition_types application/json text/html;
}
```

## 指令

#### add\_before\_body

{% tabs %}
{% tab title="语法" %}
add\_before\_body uri;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
发起子请求, 并将请求结果添加到主请求的响应体**前面**. 空的uri参数会屏蔽上层的配置
{% endtab %}
{% endtabs %}

#### add\_after\_body

{% tabs %}
{% tab title="语法" %}
add\_before\_body uri;
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
发起子请求, 并将请求结果添加到主请求的响应体**后面**. 空的uri参数会屏蔽上层的配置
{% endtab %}
{% endtabs %}

#### addition\_types

{% tabs %}
{% tab title="语法" %}
addition\_types mime-type1 mime-types2 ... ;
{% endtab %}

{% tab title="默认值" %}
addition\_types text/html;
{% endtab %}

{% tab title="上下文" %}
http, server, location
{% endtab %}

{% tab title="说明" %}
需要0.7.9 +

将会在响应头内添加指定的MIME类型, 默认为 `text/html` . 添加 `*` 值可以匹配任何MIME类型\(0.8.29+\)
{% endtab %}
{% endtabs %}

