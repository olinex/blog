---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 权限架构

### 前言

我们使用Vue进行页面开发的时候, 在中大型的项目内, 经常会遇见这么两种需求:

1. 不同的用户, 权限不同, 能够访问的页面也不同
2. 不同的用户, 权限不同, 同一个页面的能看到的元素不同

在后端渲染的时代, 第一个需求一般是由后端控制的, 但在前后端分离的时代, 某个用户是否能够访问某个页面, 则需要前端直接控制; 而第二个需求, 若没有一个可扩展且配置灵活的架构, 将会是一个繁琐且极易出错的工作

在这里我们定义两个名词:

* 接口权限: 后端传给前端的权限列表, 一般为某个用户所具有的所有权限的列表, 可以为数字或字符串, 表现形式一般为:

```javascript
permissions: ["获取用户信息", "修改用户信息", "创建用户信息", "删除用户信息"]
```

* 页面权限: 前端自行定义的权限, 受制于后端接口权限, 但由于页面上不同的需求, 往往需要对接口权限进行组合或拆分

### 结构

为了实现高效易用的权限架构, 我们需要实现以下四个部分:

{% tabs %}
{% tab title="rights.js" %}
```javascript
// permissions: 接口权限列表
// strict: 表示该权限是否需要严格满足
// 当strict = true时, 用户必须拥有所有的接口权限, 才能拥有相关的页面权限
// 当strict = false时, 用户仅需满足至少一个接口权限, 就能拥有相关的页面权限
export default {
    "显示用户信息的菜单" : {
        permissions: ["获取用户信息", "修改用户信息", "创建用户信息", "删除用户信息"],
        strict: false
    },
    "显示用户信息的列表" : {
        permissions: ["获取用户信息"],
        strict: true
    },
    "显示用户信息的创建按钮" : {
        permissions: ["创建用户信息"],
        strict: true
    },
    "显示用户信息的删除按钮" : {
        permissions: ["获取用户信息", "删除用户信息"],
        strict: true
    },
    "显示用户信息的修改按钮" : {
        permissions: ["获取用户信息", "修改用户信息"],
        strict: true
    }
}
```
{% endtab %}

{% tab title="has-right.js" %}
```javascript
// 导入rights.js
import rights from 'rights';

// 检查用户是否拥有指定的页面权限right
// permissions为后端获取的用户接口权限
export function hasRight(permissions, right) {
  if (!right) {
    return true;
  }
  if (!rights[right]) {
    return false;
  }
  if (rights[right].strict) {
    if (rights[right].permissions.length > 0) {
      return rights[right].permissions.every(perm => (permissions.includes(perm)))
    }
    return false
  } else {
    if (rights[right].permissions.length > 0) {
      return rights[right].permissions.some(perm => (permissions.includes(perm)))
    }
    return true
  }
}
```
{% endtab %}

{% tab title="router.js" %}
```javascript
// 改造路由配置, 当某个页面需要进行权限控制时, 通过meta配置页面权限
{
    name: 'UserList', path: 'user/list',
    component: () => import('pages/user/List.vue'),
    meta: {verboseName: '用户信息列表', right: '显示用户信息的列表'}
}

// 实现"看门狗"函数
function watchDog(store, to, from, next) {
  // check user permission
  if (to.matched.every(route => hasRight(store.state.user.permissions, route.meta.right))) {
    return next();
  } else {
    return next({name: 'Error403'});
  }
}

// 注册为路由的守卫函数
// 当用户没有足够的接口权限["获取用户信息"]时, 将会被重定向至403页面
router.beforeEach((to, from, next) => {
  watchDog(store, to, from, next);
});
```
{% endtab %}

{% tab title="App.vue" %}
```markup
<template>
  <div id="q-app">
    <router-view></router-view>
    <button v-show="$perm('显示用户信息的创建按钮')">create</button>
    <button v-show="$perm('显示用户信息的删除按钮')">delete</button>
  </div>
</template>

<script>
  import Vue from "vue";
  // 注册perm帮助方法, 用以在页面元素内判断用户的页面权限
  Vue.prototype.$perm = function (right) {
    return hasRight(store.state.user.permissions, right)
  };
  export default {
    name: 'App'
  }
</script>
```
{% endtab %}
{% endtabs %}

#### rights.js

为页面权限和接口权限映射关系配置文件, 前端可以通过统一的行径将接口权限和页面权限解耦, 接口权限的变化对前端的影响将被截留在这里, 通过高效灵活的配置, 可以大大减少前端的配置工作, 统一的配置也能减少前端很多的工作

#### has-right.js

核心的鉴权函数, 实现了鉴权的严格/宽松模式

#### router.js

改造路由配置并增加路由守卫函数, 可以对整个页面进行权限控制

#### App.vue

增加vue的全局鉴权函数perm, 通过v-show指令, 可以对页面中具体的某个组件进行权限控制

