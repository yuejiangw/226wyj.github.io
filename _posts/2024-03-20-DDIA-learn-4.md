---
layout: post
title: DDIA Learning - 4
categories: [Book, Learn]
description: 数据密集型应用 - 读书笔记
keywords: Architecture, Learn
---

# 第四章 - 数据编码与演化

在大多数情况下，更新应用程序功能的同时也要更新其存储的数据：

- 可能需要捕获新的字段或记录类型
- 可能需要以新的方式呈现已有数据

当数据格式或模式发生变化时，经常需要对代码进行相应调整，然而这并不容易：

- 对于服务器端，可能需要执行滚动升级（分阶段发布），即每次将新版本部署到少数几个节点，检查新版本是否正常运行，然后再逐步在所有节点上升级
- 对于客户端，依赖于用户的手动升级，存在一定滞后性

因此，新旧版本的代码以及数据格式可能在系统内共存，这要求我们的系统保持双向兼容性：

- 向后兼容：较新的代码可以读取旧代码编写的数据
- 向前兼容：较旧的代码可以读取新代码编写的数据

## 数据编码格式

程序通常至少使用两种不同的数据表示形式：

- 在内存中，保存在数据结构对象中，这些数据结构针对 CPU 的高效访问和操作进行了优化（通常使用指针）
- 将数据写入文件或通过网络传输时，编码为某种自包含的字节序列（如 JSON）

在这两种表示之间需要进行类型的转化，从内存中的表示到字节序列的转化称为编码（序列化），反之叫解码（反序列化）

### 方案 1 - 某种编程语言的编码库

使用某种编程语言内置的编码方案通常不是个好主意，原因如下：

- 编码与特定的编程语言绑定，使用另一种编程语言访问就非常困难
- 为了在相同的对象类型中恢复数据，解码过程需要能够实例化任意的类，这经常导致一些安全问题
- 在这些编码/解码库中，经常会忽略向前和向后的兼容问题
- 效率问题

### 方案 2 - JSON、XML 与二进制变体

与方案 1 相比，JSON 和 XML 不会囿于某一种编程语言，是可以由不同的编程语言编写和读取的标准化编码。CSV 是另一种流行的与语言无关的格式，尽管功能较弱。

JSON、XML 和 CSV 都是文本格式，可读性高，但除了语法外也有一些问题：

- 数字编码有许多模糊之处
- JSON 和 XML 对 Unicode 字符串有很好的支持，但不支持二进制字符串
- XML 的模式使用较为广泛，但 JSON 一般不会局限于使用模式。某些应用程序可能不得不硬编码适当的编码、解码逻辑来正确解释其信息
- CSV 没有模式，如果应用程序更改添加新的行或列，则必须手动处理

这三种编码方式中，JSON 由于其简单性以及在 Web 浏览器中支持而变成最流行的一种编码方式，但它相比二进制编码占据的空间过大。人们也开发了大量的二进制编码用以支持 JSON（如 BSON、BJSON 等），但没有一个像 JSON 和 XML 那样被广泛采用

### 方案 3 - Thrift 与 Protocol Buffers

Apache Thrift 和 Protocol Buffers（protobuf）是基于相同原理的两种二进制编码库，各有对应的代码生成工具。

Thrift 有两种不同的二进制编码格式，分别称为 BinaryProtocol 和 CompactProtocol。假如我们要对下面的结构体进行编码：

```thrift
struct Person {
    1: required string          userName,
    2: optional i64             favoriteNumber,
    3: optional list<string>    interests
}
```

采用 BinaryProtocol 编码一共需要 59 字节，如下所示：

![](/images/blog/ddia/chapter-4/thrift-binaryprotocol.png)

通过图中的字段分解我们可以得知，每个字段都有一个类型注释（前 2 个字节），并且可以在需要时指定长度。数据中出现的字符串也被编码为 ASCII 码。不过也可以发现，在编码中并没有字段名（即 userName，favoriteNumber，interests），但却包含数字类型的字段标签（1,2,3），这是因为字段标签就像是字段的别名，用来指示当前字段，但更加紧凑，因此可以省去引用字段全名。

采用 CompactProtocol 编码在语义上等价于 BinaryProtocol 但只要 34 字节即可，如下图所示。

![](/images/blog/ddia/chapter-4/thrift-compactprotocol.png)

可以发现，字段类型和标签号被打包进了单字节中，并使用可变长度整数实现。对于数字 1337，不使用全部的 8 字节，而是使用两个字节进行编码，每个字节的最高位用来指示是否还有更多的字节。

如果采用 Protocol Buffers 来对上述结构体的等价结构进行编码（如下所示）

```protobuf
message Person {
    required string user_name       = 1;
    optional int64 favorite_number  = 2;
    repeated string interests       = 3;
}
```

则只用 33 字节就可以表示相同记录，可以看到它与 Thrift CompactProtocol 的编码十分相似：

![](/images/blog/ddia/chapter-4/protobuf.png)

对于 `protobuf` 字段被标记为 `required` 还是 `optional`，在编码中都没有任何影响，只是在运行检查时会进行校验。

#### 兼容性讨论

#### 字段标签兼容性

字段标签就是我们在定义结构体时为属性名赋予的整数值，它是该字段的唯一标识。字段标签对于编码数据的含义至关重要，可以随意更改字段名称，因为从上面的编码格式我们也可以看出，编码永远不直接引用字段名称。但不能更改字段标签，它会导致现有编码无效。

在添加新字段时，只需在原有的结构体中追加新字段并赋值一个新的字段标签即可。但要注意的是，为了考虑到向前的兼容性，因此不能将新字段设置为 `required`。

在删除字段时，只能删除可选字段，使用 `required` 修饰的字段永远不能删除，而且不能再次使用相同的标签号码

#### 数据类型兼容性

改变字段的数据类型这种情况是有可能发生的，但会存在精度丢失或被截断的风险。

对于 Protocol Buffer，它没有专门的列表或数组类型，而是用 `repeated` 标记来表示这样的一个重复字段。对于重复字段，表示同一个字段标签只是简单地多次出现在记录中。可以将 `optional` 字段更改为 `repeated` 字段，这样对于读取旧数据的新代码来说，可以看到一个包含零个或一个元素的列表；对于读取新数据的旧代码来说，只能看到列表的最后一个元素。

Thrift 有列表类型，它不支持从单值到多值的改变，但它具有支持嵌套列表的优点。

### 方案 3 - Avro

Avro 是另一种二进制编码格式，适合 Hadoop 的用例。它有两种模式语言：一种（Avro IDL）用于人工编辑，另一种（基于 JSON）更易于机器读取。例如，下面的 Avro IDL

```avro
record Person {
    string             userName;
    union {null, long} favoriteNumber = null;
    array<string>      interests;
}
```

等价的 JSON 为

```json
{
  "type": "record",
  "name": "Person",
  "fields": [
    { "name": "userName", "type": "string" },
    { "name": "favoriteNumber", "type": ["null", "long"], "default": "long" },
    { "name": "userName", "type": "array", "items": "string" }
  ]
}
```

上面示例的字节编码如下图所示，可以看到，该模式中没有标签编号，是目前所有模式中最紧凑的

![](/images/blog/ddia/chapter-4/avro.png)
