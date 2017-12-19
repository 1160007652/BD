#### 简介

**Hadoop**是一个由Apache基金会所开发的分布式系统基础架构。

用户可以在不了解分布式底层细节的情况下，开发分布式程序。充分利用集群的威力进行高速运算和存储。

**Hadoop**实现了一个分布式文件系统（Hadoop Distributed File System），简称**HDFS**。HDFS有高**容错性**的特点，并且设计用来部署在低廉的（low-cost）硬件上；而且它提供高吞吐量（high throughput）来访问应用程序的数据，适合那些有着超大数据集（large data set）的应用程序。HDFS放宽了（relax）POSIX的要求，可以以流的形式访问（streaming access）文件系统中的数据。

**Hadoop**的框架最核心的设计就是：**HDFS**和**MapReduce**。

**HDFS**为海量的数据提供了存储。

**MapReduce**为海量的数据提供了计算。

#### 特点

- ###### 扩容能力（Scalable）

  > 能可靠的（reliably）存储和处理千兆（PB）数据。

- ###### 成本低（Economical）

  > 可以通过普通机器组成服务器来分发以及处理数据。这些服务器群总计可达数千个节点

- ###### 高效率（Efficient）

  > 通过分发数据，hadoop可以再数据所在的节点上并行（parallel）处理它们，这使得处理的非常的快速。

- ###### 可靠性（Reliable）

  > hadoop能自动的维护数据的多份副本，并且在任务失败后能自动的重新部署（redeploy）计算任务。

