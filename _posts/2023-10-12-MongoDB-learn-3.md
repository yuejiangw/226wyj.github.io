---
layout: post
title: MongoDB Learning - 3
categories: [Learn]
description: Udemy Mongo 课程笔记
keywords: MongoDB, Learn
---

> 课程：[udemy](https://www.udemy.com/course/best-mongodb/)

## NOSQL 中的 Relationship

### 一对一关系

假设我们有两个 collection，一个来存储 people，一个来存储 address，这两个 collection 中的数据具有一对一关系：

people collection 中的数据如下所示，其中 `address` 字段的值对应是是 address collection 中数据的 `id` 字段

```javascript
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "address": 1
}
```

address collection 中的数据如下所示：

```javascript
{
    "_id": 1,
    "city": "Shanghai",
    "street": "aaa",
    "building": 1,
    "room": 1021
}
```

这样存储的话会有一个问题，就是我们想要获取 people collection 中的数据的话，要经过两次查询，第一次查 people，第二次查 address，效率比较低。为此，我们的解决办法是，将它们写在同一个 collection 内，新的数据格式如下：

```javascript
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "address": {
        "city": "Shanghai",
        "street": "aaa",
        "building": 1,
        "room": 1021
    }
}
```

这样我们只需要一次查询就可以获取所需信息，也可以通过 projection 来筛选返回的 field，非常方便。

### 一对多关系

一对多关系也是我们生活中很常见的一种关系，假如我们想要表示一个人有多个订单，那么首先我们可以采用如下这种 schema：

```javascript
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "orders": [
        {
            "date": "2018-12-10",
            "product": "book",
            "cost": 19.99
        },
        {
            "date": "2018-12-13",
            "product": "book",
            "cost": 39.99
        },
        {
            "date": "2018-12-12",
            "product": "computer",
            "cost": 2899
        },
    ]
}
```

上面的这种存储方式可以作为一种选择，但它有一些问题：

- 如果我们想要对所有订单进行处理的话，这种存储方式会比较困难，因为不同的人可能会有相同的订单，需要考虑去重
- Mongo DB 中单个 document 的大小限制是 16 M，如果订单数量较多的话可能会使 document 体积过大

为了解决上述两个问题，我们也可以把用户信息和订单信息拆分成两个 collection，使用 `id` 字段作为链接

**people collection**

```javascript
{
    "_id": 1,
    "name": "John Cart",
    "email": "jc@jc.com",
    "phone": 12345,
    "orders": [1, 2, 3]
}

```

**order collection**

```javascript
[
    {
        "_id": 1
        "date": "2018-12-10",
        "product": "book",
        "cost": 19.99
    },
    {
        "_id": 2,
        "date": "2018-12-13",
        "product": "book",
        "cost": 39.99
    },
    {
        "_id": 3,
        "date": "2018-12-12",
        "product": "computer",
        "cost": 2899
    },
]
```

可以看到，我们将用户信息和订单信息分别存入了两个 collection 中，在用户信息中有一个 `orders` 字段，它的格式是一个 array，存储了所有该用户名下的订单的 `id`

### 多对多

以下面的订单关系为例，顾客和商品之间是多对多关系，我们采用的策略是通过一张中间表来表示这种关系，这里是订单表。如果选择将订单信息全部存入顾客表中，则可扩展性太差。

![](/images/blog/mongo/many-to-many.png)