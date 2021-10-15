<!--
 * @Descripttion: 
 * @version: 
 * @Author: cm.d
 * @Date: 2021-10-15 14:36:31
 * @LastEditors: cm.d
 * @LastEditTime: 2021-10-15 17:53:10
-->

# SRE实战(01)初识+探索SRE如何推进好大夫在线技术债务改造【阅读随笔】

## 原文链接

原文来自: [SRE实战(01)初识+探索SRE如何推进好大夫在线技术债务改造](https://www.infoq.cn/article/vBUfppUrg9LfeLRoGx3t)

## 阅读随笔

最近好大夫连发了几篇文章，前两天刚看了他们在微服务治理上做了哪些事情，对于如何拆解topic、演进路线等等在方法论上有很多值得借鉴的地方，今天他们又发了篇新文章，看标题和内容，应该是针对SRE的一些内容，这一篇还比较浅显，大部分都是SRE常用的方法论，几个关键点记录一下：

### SRE稳定性保障规划图

![SRE稳定性保障规划图](https://static001.geekbang.org/infoq/93/933d56bfc5189c33cb9bef8491842325.webp)

### 如何衡量稳定性

1. MTTF (Mean Time To Failure，平均无故障时间)，指系统无故障运行的平均时间，取所有从系统开始正常运行到发生故障之间的时间段的平均值。MTTF =∑T1/ N；

2. MTTR (Mean Time To Repair，平均修复时间)，指系统从发生故障到维修结束之间的时间段的平均值。MTTR =∑(T2+T3)/ N；

3. MTBF (Mean Time Between Failure，平均失效间隔)，指系统两次故障发生时间之间的时间段的平均值。MTBF =∑(T2+T3+T1)/ N

MTBF= MTTF+ MTTR

衡量稳定性：AO = MTBF / (MTBF + MTTF)

衡量指标就是 SLI，衡量的目标就是 SLO，如果针对服务质量的还有一个 SLA

PS：第N次见到这个公式了，但是实际工作中确实很少用

### 如何选择合适的 SLI，设置合理的 SLO

![衡量指标制定黄金法则](https://static001.geekbang.org/infoq/ff/ff4380b372f190e3ebd8af490ae2621d.webp)

### 服务依赖性

1. Fan-in：入向依赖，这个指标指代了组件外部类依赖于组件内部类的数量
2. Fan-out：出向依赖，这个指标指代了组件内部类依赖于组件外部类的数量；
3. I：不稳定性，I=Fan-out/（Fan-in+Fan-out）。该指标的范围是[0,1],I=0 意味着组件是最稳定的，I=1 意味着组件是最不稳定的。

PS：应该很好理解......

### 其他

剩下都是一些比较常见的概念，就不在这里记录了
