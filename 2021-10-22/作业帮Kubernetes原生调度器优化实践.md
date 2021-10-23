# 作业帮Kubernetes原生调度器优化实践【阅读随笔】

## 原文链接
原文链接：[作业帮Kubernetes原生调度器优化实践](https://www.infoq.cn/article/SACjBwF5r2rIHNDb17xQ)

## 阅读随笔

比较短的文章，基本路线是原生K8s调度算法 => 遇到的规模瓶颈 => 优化的调度算法这样的路线

#### 基础调度算法

先复习下k8s的基础调度算法：
![一图读懂](https://static001.geekbang.org/infoq/5b/5bd8c4755399fdc7ab049a99e60bb15d.webp)

Informer Path: 创建多个Informer来watch etcd的资源变更，etcd上是需要调度的资源的当前状况，比如Pos、Service等，当有资源发生变更，那么对应的Informer会发现变化，然后生成一个需要补偿的任务放到一个Fifo队列里，并且更新SchedulerCache

Secheduling Path：通过两个步骤调度任务：
+ Predicates：通过SchedulerCache获取所有满足任务的、可以调度的Node
+ Priorities：通过算法打分，选出分数最高的节点进行任务Bind

#### 大规模集群调度带来的问题

默认调度的缺陷：
+ 调度维度少
+ 不支持并发

目前作业帮的规模：pod 10w以上，整体资源分配60%，包含GPU、离在线混部多个场景

带来的业务问题：
+ 只能通过request进行调度，不能说明真实资源压力状况
+ 实时调度，对于明显有规律的不适用

#### 解决方案

目前针对上述问题，作业帮用了以下几种方案：
+ 高峰预测调度
+ 修改算法，支持多维度
+ 拆解调度器，将原有的调度器拆解为forecast-scheduler、gpu-scheduler、job-scheduler

这里面还用到了serverless来处理任务，说实话没看懂，等我打探一番后面补充

#### 未来

+ 更加细粒度的资源域划分
+ 抢占式调度