<!--
 * @Descripttion: 
 * @version: 
 * @Author: cm.d
 * @Date: 2021-11-06 14:19:07
 * @LastEditors: cm.d
 * @LastEditTime: 2021-11-06 15:37:07
-->
# 耗时 18 个月，我们构建了一个真正可扩展的无服务器 SQL 数据库【阅读随笔】

## 原文链接

原文链接：[耗时 18 个月，我们构建了一个真正可扩展的无服务器 SQL 数据库](https://www.infoq.cn/article/4cCk2dbTFXP3k6pNvNhe)

## 阅读随笔

这篇文章是官方的一篇文章，讲述的是CockroachDB做无服务器化(其实也是一种serverless)的方案和思考，值得一读

### 单租户架构

![一图开始](https://static001.geekbang.org/infoq/69/69146f9c7d971cb28f36e588f8d7d3de.jpeg)

![第二图](https://static001.geekbang.org/infoq/62/62e95872586d8fca5369d7307a8a6c84.jpeg)

上面其实就是个典型的NewSQL的架构，CockroachDB本身也是参考spanner，所以有相关架构基础的话很容易理解  

### 多租户架构

![一图开始](https://static001.geekbang.org/infoq/ef/efa7ad8c632f40bbb665b6a3b00349c2.jpeg)

新的架构中，对于SQL层和分布式层做了隔离，存储层架构共享，数据不共享。数据层面对key进行处理，增加了针对每个租户的前缀，/<table-id>/<index-id>/<key> => /<tenant-id>/<table-id>/<index-id>/<key>，不同租户之间生成的kv隔离。存储节点还将认证所有来自 SQL 节点的通信，并每个租户只能访问以他们自己的租户标识符为前缀的密钥。为了确保单个租户无法垄断存储节点上的资源，会通过一些限流手段进行限制。  

### 无服务器架构

![一图开始](https://static001.geekbang.org/infoq/83/83a8ed3524e7178948e7583005114592.jpeg)

这个图其实很明晰了，唯一的点是proxy pod，在这里是作为连接的负载均衡，已经分配租户到租户的sqlpod。

### 扩展

### Autoscaler

Autoscaler 监控集群中每个 SQL pod 的 CPU 负载，并根据两个指标来计算 SQL pod 的数量：
+ 最近 5 分钟内的平均 CPU 使用率。
+ 最近 5 分钟内的 CPU 使用峰值。

通过平均CPU确定基线，即基础的SQL Pod数，但是实时通过峰值进行调整。当 Autoscaler 得出 SQL pod 的理想数量时，它将触发一个 K8s 调整过程，增加或删除 pod，以达到理想数量

![一图](https://static001.geekbang.org/infoq/d0/d0a8e0d8b844a06c8a5826bfac624ac2.jpeg0)

这里面还建立了一个预热池，用来快速加入负载，因为k8s拉起一个pod需要2-30s。  

另外，如果有请求过来到达Pod Proxy时，还没有任何SQL Pod生成，它将触发与 Autoscaler 所使用的相同的 K8s 调整过程。从预热的 SQL pod 池中提取出一个新的 pod，并设置标记标识现在可以用于连接。整个恢复过程只需要几分之一秒.
