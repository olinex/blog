---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 角色及用户控制

## 用户

MongoDB的用户是有数据库从属的. 也就是说, 如果我们创建了两个数据库db1和db2, 并创建了两个用户user1和user2, 他们可以各自从属于自己的数据库, 如user1从属db1, user2从属db2. 这是为了让MongoDB可以更好地支持多租户的机制.

### 查看所有用户

```text
user admin
db.auth('<admin-username>', '<admin-password>')
db.system.users.find()
```

### 创建用户管理员

```text
user admin
db.createUser({
    user: "<username>",
    pwd: "<password>"
    roles: [
        {role: "userAdminAnyDatabase", db: "admin"}
    ]
})
```

### 创建高级管理员

```text
user admin
db.createUser({
    user: "<username>",
    pwd: "<password>"
    roles: [
        {role: "userAdminAnyDatabase", db: "admin"},
        {role: "readWriteAnyDatabase", db: "admin"}
    ]
})
```

### 创建普通用户

```text
user <db>
db.createUser({
    user: "<username>",
    pwd: "<password>"
    roles: [
        {role: "readWrite", db: "<db>"}
    ]
})
```

## 角色



