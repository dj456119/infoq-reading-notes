
# Apache ShardingSphere在京东白条场景的落地之旅【阅读随笔】

## 原文链接

[Apache ShardingSphere在京东白条场景的落地之旅](https://xie.infoq.cn/article/9a498468a83a8b550f0f48c31)

## 阅读随笔

京东白条数据架构演进：Solr+HBase => MongoDB => DBRep + ES + HBase

依然存在的问题：通过业务框架实现的数据分片方案导致业务代码复杂度增加、维护成本不断攀升，紧耦合的弊端原形毕露，应用每次升级都需要投入较多的精力对分片做相应调整，研发人员难以专注于业务本身。说白了就是早期基于业务维度去拆分数据，导致业务代码与拆分逻辑耦合

解耦目标：

+ 聚焦精力：将基于架构的数据库拆分，交给分表组件实现，研发精力需聚焦于业务本身；
+ 简化升级：解耦技术架构，简化业务系统升级工作的研发流程；
+ 规划未来：为系统提供良好的扩展能力，从容应对“618”和“11. 11”等活动。

引入ShardingSphere-JDBC，轻量级Java框架，客户端直连，jar包，完全兼容JDBC和其他ORM框架

![新的模式](https://static001.geekbang.org/infoq/14/14ec1cce2df3220d67ca6055e8658680.webp?x-oss-process=image/resize,p_80/format,jpg)

+ SQL 解析结果缓存
+ JDBC 元数据信息缓存
+ Bind 表 & 广播表的使用
+ 自动化执行引擎 & 流式归并
  