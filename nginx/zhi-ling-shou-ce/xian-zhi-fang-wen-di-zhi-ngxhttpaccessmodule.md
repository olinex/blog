---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# 限制访问地址

`ngx_http_access_module` 模块可以通过指定来源地址, 来限制客户端的访问.

同样的, 可以通过 `ngx_http_auth_base_module`, `ngx_http_auth_request_module`, `ngx_http_auth_jwt_module` 来控制. 若以上多种控制方式同时生效, 可以通过 `satisfy` 指令, 来明确控制逻辑, 是任意控制条件都满足时运行请求访问, 还是需要同时满足所有控制条件.

## 示例

```text
location / {
    deny 192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny all;
}
```

控制规则将按照由上到下的顺序依次检查, 直到有任意一条满足. 在上述示例中, 只有 IPv4 网络的 192.168.1.0/24  \( 不包含192.168.1.1 \) , 10.1.1.0/16 和 IPv6 网络的 2001:0db8::/32 可以访问. 当存在很多规则时, `ngx_http_geo_module` 更加合适.

## 指令

#### allow

{% tabs %}
{% tab title="语法" %}
**allow** address \| CIDR \| unix: \| all
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
http, server, location, limit\_except
{% endtab %}

{% tab title="说明" %}
允许特定的网络地址访问. 如果设置为特殊的 unix: 地址 \(1.5.1\), 则允许所有的 UNIX 套接字 访问
{% endtab %}
{% endtabs %}

#### deny

{% tabs %}
{% tab title="语法" %}
**deny** address \| CIDR \| unix: \| all
{% endtab %}

{% tab title="默认值" %}
无
{% endtab %}

{% tab title="上下文" %}
http, server, location, limit\_except
{% endtab %}

{% tab title="说明" %}
拒绝特定的网络地址访问. 如果设置为特殊的 unix: 地址 \(1.5.1\), 则拒绝所有的 UNIX 套接字 访问
{% endtab %}
{% endtabs %}





