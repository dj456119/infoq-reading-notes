
# 网易云音乐音视频算法的 Serverless 探索之路【阅读随笔】

## 原文链接

原文链接：[网易云音乐音视频算法的 Serverless 探索之路](https://xie.infoq.cn/article/99e1e90e4c9d2d505d42c2417)

## 阅读随笔

文章三个部分：

+ 现状：音视频技术在网易云音乐的应用情况，引入 Serverless 技术之前遇到的问题
+ 选型：调研 Serverless 方案时的考虑点
+ 落地和展望：进行了哪些改造，最终的落地效果和未来规划

### 现状

![一图说明当前架构](https://static001.geekbang.org/infoq/69/6976687174ee6452ff35b4384b1fb1ab.png)

音视频处理特点：  
![图](https://static001.geekbang.org/infoq/d9/d9e7873c5cff125a545a598e099315c8.png)

最终的音视频架构：  
![图](https://static001.geekbang.org/infoq/46/46fae1014be6411f70d06b5df3a601cc.png)

音视频处理平台执行层细化：  
![图](https://static001.geekbang.org/infoq/7b/7b915fc4428606afbc0cc00f037c387e.png)

处理机器资源的时间，甚至多余开发时间  

核心问题：

+ 存量增量差异变大，增量算法多，资源协调困难
+ 算法复杂度变高，对资源要求、规格、利用率变高
+ 需要有好用的弹性资源

### 选型

需要考虑的点：

+ 成本
+ 运行环境的支持
+ 弹性能力

决定采用公有云Function

### 落地

![新的架构](https://static001.geekbang.org/infoq/d6/d68e02138c6b9cb614855f5a101ff61a.png)

引入serverless需要关注的计费：

+ 内存规格
+ 触发计算次数
+ 公网出流量

最终效果：  

![图](https://static001.geekbang.org/infoq/76/764caab4780cffae9ecbb483c99dc3e9.png)

