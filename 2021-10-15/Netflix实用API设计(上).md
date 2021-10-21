
# Netflix实用API设计(上) 阅读笔记

## 原文链接

原文来自: [Netflix实用API设计(上)](https://xie.infoq.cn/article/9177627e96925ef1bd53ee01c)  

## 阅读笔记

### 如何通过FieldMask解决问题

本文比较简单，讲述了如何通过Protobuf的FieldMask来设计API，来解决通过Grpc和Protobuf在调用时会无谓的传输调用端不需要的字段的问题，尤其是这些额外的字段甚至可能出现跨服务请求的情况，文章举了这样一个栗子：
![一个调用栗子](https://static001.geekbang.org/infoq/9d/9da9b1b8ce6b7580ec3c8b13748e2c71.png)

当调用方请求GetProduction这个Api时，需要的两个字段scheduler和scripts需要从后面的服务获取，而调用方可能根本不需要这两个字段。

一个丑陋的解法：

```java
// Request with one-off "include" fields, not recommended
message GetProductionRequest {
  string production_id = 1;
  bool include_format = 2;
  bool include_schedule = 3;
  bool include_scripts = 4;
}
```

对于嵌套字段、维护性都非常不友好，通过FieldMask可解：

```java
import "google/protobuf/field_mask.proto";

message GetProductionRequest {
  string production_id = 1;
  google.protobuf.FieldMask field_mask = 2;
}

```

如果消费者只对产品标题和格式感兴趣，他们可以设置路径为“title”和“format”的 FieldMask：

```java
FieldMask fieldMask = FieldMask.newBuilder()
    .addPaths("title")
    .addPaths("format")
    .build();

GetProductionRequest request = GetProductionRequest.newBuilder()
    .setProductionId(LA_CASA_DE_PAPEL_PRODUCTION_ID)
    .setFieldMask(fieldMask)
    .build();
```

***按照惯例，如果请求中没有包含 FieldMask，则应该返回所有字段。***

这里面有个另外需要注意的点，使用FieldMask，实际上是基于字段名的发送而非字段号，这就导致当某一方修改了field的path，两边没有同步，就无法找到对应的字段了。

![有图有真相](https://static001.geekbang.org/infoq/8c/8cca684f14f0988e3995bb03bea70a38.png)

> 作者给出了几种解决方案:
>
> + 使用 FieldMask 时，永远不要重命名字段。这是最简单的解决方案，但并不总是可行。
> + 要求后端支持所有旧字段名。这解决了向后兼容性问题，但需要在后端添加额外的代码来跟踪所有历史字段名。
> + 弃用旧字段并创建一个新字段而不是重命名。在我们的示例中，我们将创建新的 title_name，并设置字段号为6。与前一个选项相比，这个选项的优点在于：允许生产者继续使用生成的描述符而不是自定义转换器；另外，可以让消费者很快察觉到某个字段已经被弃用了。  

使用预构建，可以为最常用的字段组合提供预构建的 FieldMask 客户端库。  

> 限制
>
> + 使用 FieldMask 限制了重命名消息字段的能力
> + 重复字段只允许出现在路径字符串的最后一个位置，这意味着不能在列表中选择（屏蔽）单个子字段。在可预见的未来，这种情况可能会改变，因为最近批准的谷歌 API 改进建议 AIP-161 Field masks[14]包含了对重复字段的通配符支持。
