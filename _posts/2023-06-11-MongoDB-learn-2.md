---
layout: post
title: MongoDB Learning - 2
categories: [MongoDB, Learn]
description: 《MongoDB权威指南》学习笔记 - 2
keywords: MongoDB, Learn
---

## 创建、更新和删除文档

### 创建文档

使用 `insert` 插入一个文档是向 MongoDB 中添加数据的最基本的方法，`insert` 的参数是一个文档或者文档的数组，例如：

```javascript
db.foo.insert({bar: 'baz'})
```

如果插入的文档没有 `_id` 字段，MongoDB 会自动为文档添加一个 `_id` 字段，这个字段的值是一个 `ObjectId` 类型的值，这个值是 MongoDB 生成的一个 12 字节的唯一标识符，这个值在集合中是唯一的。

如果要插入多个文档，使用批量插入会更快一些，因为一次批量插入只是单个的 TCP 请求，避免了许多零碎的请求所带来的开销。

当执行插入的时候，使用的驱动程序会将数据转换成 `BSON` 形式，然后将其送入数据库。数据库解析 `BSON`，检验其是否包含 `_id` 键并且文档不超过 4MB，除此之外不做别的数据验证。

### 删除文档

删除文档使用 `remove` 方法，`remove` 方法的参数是一个查询文档，例如：

```javascript
db.foo.remove({bar: 'baz'})
```

如果不加参数，`remove` 方法会删除集合中的所有文档，但是不会删除集合本身，原有的索引也会保留。删除数据是永久性的，不能撤销也不可恢复。

删除文档通常会很快，但是要清除整个集合，直接删除集合会更快一些。

### 更新文档

更新文档使用 `update` 方法，`update` 方法的参数是一个查询文档和一个要将匹配到的文档更新成的文档，例如：

```javascript 
db.foo.update({bar: 'baz'}, {bar: 'baz', zzz: 'xxx'})
```

在这种情况下，`update` 方法会将匹配到的文档替换成新的文档，如果不想替换整个文档，可以使用 `$set` 操作符（update operator），例如：

```javascript
db.foo.update({bar: 'baz'}, {$set: {bar: 'baz', zzz: 'xxx'}})
```

`$set` 操作符会将指定的字段添加到文档中，如果文档中已经存在这个字段，那么就会更新这个字段的值。

需要注意的一点是，如果只想更新文档中的一部分，那么一定要使用更新操作符，否则会将整个文档替换掉。

对于更多相关的操作符，可以参考 [update operators](https://docs.mongodb.com/manual/reference/operator/update/)。