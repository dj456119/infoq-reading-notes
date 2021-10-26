# 作业帮Kubernetes Serverless在大规模任务场景下的落地和优化【阅读随笔】

## 原文链接

原文链接：[作业帮Kubernetes Serverless在大规模任务场景下的落地和优化](https://www.infoq.cn/article/3u8pSDfaYbD0BfRAuyy4)

## 阅读随笔

最近作业帮连发文章，感觉在k8s体系还是耕耘了不少东西。在之前阅读过他们对调度器的优化后，这是第二篇，讲serverless的场景。不过作业帮对serverless的使用，并不是采用了function的模式，这个serverless我个人理解只是针对job的视角不去关注服务器本身，所以是servlerless。从某种角度上看，这么理解其实也没问题。

### 核心问题

1. 集群内节点稳定性
  + 频繁pod创建销毁 => cgroup过多 => memory cgroup不能即时回收，读取/sys/fs/cgroup/memory/memory.stat变慢 => kubelete需要定时读取该文件来统计mem namesapce消耗 => cpu sys升高
  + memory cgroup不能即时回收的原因：mem cgroup释放会遍历所有缓存页，这些缓存页并不是被mem cgroup使用完实时释放，而是延时释放，只有内核再次用到这些内存页才会回收。因此会导致遍历时间变长
  + Job的特点，短生命周期，大量级
2. 网络模式采用CNI模式，Pod数量有限，需要预留给job，导致大量资源被浪费
3. 定时任务经常0点大批量执行，抢占线上业务

### 虚拟节点解决方案

采用serverless+虚拟节点来解决问题

#### 虚拟节点+serverless本身的方案问题

1. 自研日志消费服务
2. 监控报警统一