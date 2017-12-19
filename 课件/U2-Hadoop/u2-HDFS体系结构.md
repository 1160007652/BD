#### **前言**

> HDFS 是一个能够面向大规模数据使用的，可进行扩展的文件存储与传递系统。是一种允许文件通过网络在多台主机上分享的文件系统，可让多机器上的多用户分享文件和存储空间。让实际上是通过网络来访问文件的动作，由程序与用户看来，就像是访问本地的磁盘一般。即使系统中有某些节点脱机，整体来说系统仍然可以持续运作而不会有数据损失。

#### **HDFS体系结构**

![hdfs体系结构图1](C:\Users\Administrator\Desktop\工作-大数据\课件\U2-Hadoop\img\hdfs体系结构图1.jpg)

##### 简述

###### 主从结构

- 主节点，只有一个：namenode
- 从节点，有很多个：datanodes

###### namenode负责

- 接收用户操作请求
- 维护文件系统的目录结构
- 管理文件与block之间关系，block与datanode之间关系

###### datanode负责

- 存储文件
- 文件被分成block存储在磁盘上
- 为保证数据安全，文件会有多个副本

##### 采用Master-Slaver模式

###### 1. NameNode中心服务器（Master）

​	维护文件系统树、以及整棵树内的文件目录、负责整个数据集群的管理。

###### 2. DataNode分布在不同的机架上（Slaver）

​	在客户端或者NameNode的调度下，存储并检索数据块，并且定期向NameNode发送所存储的数据块的列表。

​	客户端与NameNode获取元数据。

​	客户端与DataNode交互获取数据。

​	默认情况下，每个DataNode都保存了3个副本，其中两个保存在同一个机架的两个不同的节点上。另一个副本放在不同机架上的节点上。

##### 基本概念

1. ###### 机架

   HDFS集群，由分布在多个机架上的大量DataNode组成，不同机架之间节点通过交换机通信，HDFS通过机架感知策略，使NameNode能够确定每个DataNode所属的机架ID，使用副本存放策略，来改进数据的可靠性、可用性和网络带宽的利用率。

2. ###### 数据块(block)

   HDFS最基本的存储单元，默认为64M，用户可以自行设置大小。

3. ###### 元数据

   指HDFS文件系统中，文件和目录的属性信息。HDFS实现时，采用了 镜像文件（Fsimage） + 日志文件（EditLog）的备份机制。文件的镜像文件中内容包括：修改时间、访问时间、数据块大小、组成文件的数据块的存储位置信息。目录的镜像文件内容包括：修改时间、访问控制权限等信息。日志文件记录的是：HDFS的更新操作。

   NameNode启动的时候，会将镜像文件和日志文件的内容在内存中合并。把内存中的元数据更新到最新状态。

4. ###### 用户数据

   HDFS存储的大部分都是用户数据，以数据块的形式存放在DataNode上。

   在HDFS中，NameNode 和 DataNode之间使用TCP协议进行通信。DataNode每3s向NameNode发送一个心跳。每10次心跳后，向NameNode发送一个数据块报告自己的信息，通过这些信息，NameNode能够重建元数据，并确保每个数据块有足够的副本。

##### HDFS写取数据的流程

​	![hdfswrite](C:\Users\Administrator\Desktop\工作-大数据\课件\U2-Hadoop\img\hdfswrite.jpg)

##### HDFS读取数据的流程

![hdfsread](C:\Users\Administrator\Desktop\工作-大数据\课件\U2-Hadoop\img\hdfsread.jpg)