<!--
 * @Descripttion: 
 * @version: 
 * @Author: cm.d
 * @Date: 2021-11-06 13:45:22
 * @LastEditors: cm.d
 * @LastEditTime: 2021-11-06 14:18:15
-->
# 反应式单体：如何从 CRUD 转向事件溯源【阅读随笔】

## 原文链接

原文链接：[反应式单体：如何从 CRUD 转向事件溯源](https://www.infoq.cn/article/JJhGL9fivChDMV2s4d00)

## 阅读随笔

说实话很久没有关注企业级软件开发的一些方式方法了，今天看了这篇文章也才是第一次听说事件溯源这种新的模型，翻翻资料记录一下。  
事件溯源的本质是对通过CURD将状态变更而导致相关联的业务产生影响进行抽象，抽象为对业务领域事件的处理方法，文中举了个例子：传统的 CRUD 方式进行系统设计时，我们主要关注的是状态以及如何在一个分布式环境中由多个用户进行状态的创建、更新和删除操作，而事件溯源方式关注的是领域事件，它们何时发生以及它们如何表达业务意图。在事件溯源方式中，状态是事件的具体化（materialization），这只是领域事件多种可能的使用方式之一。

### 如果我们能重新开始的话，系统会是什么样子呢？

这里丢了两篇关于事件溯源的文章，先mark住：
+ [较新的文章](https://www.confluent.io/blog/event-sourcing-cqrs-stream-processing-apache-kafka-whats-connection/)
+ [较旧的文章](https://martinfowler.com/eaaDev/EventSourcing.html)
其中较旧的文章是Martin Fowler的大作，可以拜读下  

![一图流](https://static001.geekbang.org/infoq/58/58af75b33ee2d371a2fbfdea5b01f67b.webp)

上面的图是事件溯源架构中的一般流程，用户通过command API发出command，旨在改变某个实体（通过 entity-id 进行唯一标识）的状态。通过aggregate聚合，聚合要根据当前的实体状态决定接受或拒绝命令。如果一条命令被接受的话，聚合要发布一个或多个领域事件同时要更新当前实体的状态。

### 使用 Kafka Streams 作为事件溯源框架

继续理解上面的图，能看到在aggregate中有个Stateful，实际就是对状态的存储，对这个状态的变更，触发下领域事件。文中使用Kafka Streams+Rocksdb作为整个处理器，Rocksdb用来存储状态。采用 exactly-once 语义来保证事务肯定会被发送到下游。通过kafka的topic，保证同一个entry id只能被一个进程处理。

### 在我们的单体 CRUD 系统中，是如何引入领域事件的？

![一图流](https://static001.geekbang.org/infoq/b2/b29de78aa7c39e19051fd9d1de6d43e6.webp)

这里面用户通过RestAPI变更了数据库，触发了CDC捕捉，推动命令发出，下面介绍CDC

### 变更数据捕获（Change Data Capture，CDC）

其实很容易想到就是订阅了Binlog，233333，然后转换成command。然后通过kafa stream发布到aggregate，aggregate通过entry-id获取在rocksdb里的数据比较状态差异化，生成领域事件

### CDC 记录代表了已提交的变化，为什么它们不是事件呢？

当执行无状态转换时，我们无法对来自不同表的 CDC 记录做出正确的反应，因为不同的表之间无法保证顺序。最终，我们可能会在获得 Order 记录之前就处理了 OrderLine 记录。一个好的领域事件将提供一些关于 Order 的上下文，将其作为 OrderLine 事件的一部分。采用有状态的转换允许我们使用聚合状态作为 OrderLine 的存储，并且只有在 Order 数据到达之后才发布 OrderLine 事件。这是聚合作为实体事件源的责任的一部分。

### 后续的文章会持续跟进的话题

+ 如何使用 Kafka Streams 来表达聚合的事件溯源概念。
+ 如何支持一对多的关系。
+ 如何通过重新划分事件来驱动反应式应用。
+ 如何重新处理命令的历史，确保在响应事件的反应式服务不停机的情况下重建事件。
+ 最后，如何在多中心的 Kafka 中运行有状态的转换

