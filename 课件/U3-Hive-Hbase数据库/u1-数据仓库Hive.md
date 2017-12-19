#### 简介

Hive是建立在 Hadoop 上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载(ETL)，这是一种可以存储、查询和分析存储在 Hadoop 中的大规模数据的机制。Hive 定义了简单的类 SQL 查询语言，称为 HQL，它允许熟悉 SQL 的用户查询数据。同时，这个语言也允许熟悉 MapReduce 开发者的开发自定义的 mapper 和 reducer 来处理内建的 mapper 和 reducer 无法完成的复杂的分析工作。

Hive 没有专门的数据格式。 Hive 可以很好的工作在 Thrift 之上，控制分隔符，也允许用户指定数据格式。

#### 适用场景

Hive 构建在基于静态批处理的Hadoop 之上，Hadoop 通常都有较高的延迟并且在作业提交和调度的时候需要大量的开销。因此，Hive 并不能够在大规模数据集上实现低延迟快速的查询，例如，Hive 在几百MB 的数据集上执行查询一般有分钟级的时间延迟。因此，

Hive 并不适合那些需要低延迟的应用，例如，联机事务处理(OLTP)。Hive 查询操作过程严格遵守Hadoop MapReduce 的作业执行模型，Hive 将用户的HiveQL 语句通过解释器转换为MapReduce 作业提交到Hadoop 集群上，Hadoop 监控作业执行过程，然后返回作业执行结果给用户。Hive 并非为联机事务处理而设计，Hive 并不提供实时的查询和基于行级的数据更新操作。Hive 的最佳使用场合是大数据集的批处理作业，例如，网络日志分析。

#### 特征

Hive 是一种底层封装了Hadoop 的数据仓库处理工具，使用类SQL 的HiveQL 语言实现数据查询，所有Hive 的数据都存储在Hadoop 兼容的文件系统(例如，Amazon S3、HDFS)中。Hive 在加载数据过程中不会对数据进行任何的修改，只是将数据移动到HDFS 中Hive 设定的目录下，因此，Hive 不支持对数据的改写和添加，所有的数据都是在加载的时候确定的。Hive 的设计特点如下。

● 支持索引，加快数据查询。

● 不同的存储类型，例如，纯文本文件、HBase 中的文件。

● 将元数据保存在关系数据库中，大大减少了在查询过程中执行语义检查的时间。

● 可以直接使用存储在Hadoop 文件系统中的数据。

● 内置大量用户函数UDF 来操作时间、字符串和其他的数据挖掘工具，支持用户扩展UDF 函数来完成内置函数无法实现的操作。

● 类SQL 的查询方式，将SQL 查询转换为MapReduce 的job 在Hadoop集群上执行。

#### 体系结构

**用户接口**

用户接口主要有三个:CLI，Client 和 WUI。其中最常用的是 CLI，Cli 启动的时候，会同时启动一个 Hive 副本。Client 是 Hive 的客户端，用户连接至 Hive Server。在启动 Client 模式的时候，需要指出 Hive Server 所在节点，并且在该节点启动 Hive Server。 WUI 是通过浏览器访问 Hive。

**元数据存储**

Hive 将元数据存储在数据库中，如 mysql、derby。Hive 中的元数据包括表的名字，表的列和分区及其属性，表的属性(是否为外部表等)，表的数据所在目录等。

**解释器、编译器、优化器、执行器**

解释器、编译器、优化器完成 HQL 查询语句从词法分析、语法分析、编译、优化以及查询计划的生成。生成的查询计划存储在 HDFS 中，并在随后由 MapReduce 调用执行。

**Hadoop**

Hive 的数据存储在 HDFS 中，大部分的查询由 MapReduce 完成(包含 * 的查询，比如 select * from tbl 不会生成 MapReduce 任务)。

#### 安装

**下载hive安装文件**

可以从Apache官网下载安装文件,即 http://mirror.bit.edu.cn/apache/hive/

除此之外，由于hive是默认将元数据保存在本地内嵌的 Derby 数据库中，但是这种做法缺点也很明显，Derby不支持多会话连接，因此本文将选择mysql作为元数据存储。

需要下载mysql的jdbc，然后将下载后的jdbc放到hive安装包的lib目录下， 下载链接是：<http://dev.mysql.com/downloads/connector/j/> ，也可以从上述云盘中获取。

**安装mysql来替换默认的Derby数据库 **

​      请参考 [Hive集成mysql数据库](http://www.cnblogs.com/kinginme/p/7249533.html)

**修改配置文件**

​     解压安装文件到指定的的文件夹 /opt/hive

​      tar -zxf apache-hive-2.1.0-bin.tar.gz -C  opt/hive

**设置环境变量**

​      vi /etc/profile

​	![159642-20170725120036412-1423372815](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170725120036412-1423372815.png)

**修改hive-site.xml文件**

​	![159642-20170726164501781-1852853436](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726164501781-1852853436.png)

![159642-20170726164402656-653868153](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726164402656-653868153.png)

**修改hive-env.sh**

​         cp hive-env.sh.template  hive-env.sh

​         vi  hive-env.sh

![img](file:///C:/Users/Administrator/Desktop/%E5%B7%A5%E4%BD%9C-%E5%A4%A7%E6%95%B0%E6%8D%AE/%E8%AF%BE%E4%BB%B6/U3-Hive-Hbase%E6%95%B0%E6%8D%AE%E5%BA%93/img/159642-20170725121909927-1145171090.png?lastModify=1513649449?lastModify=1513649449?lastModify=1513649449)

**运行hive**

​      运行hive之前首先要确保meta store服务已经启动，

​      nohup hive --service metastore > metastore.log 2>&1 &

![159642-20170726155402546-1166220885 (1)](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726155402546-1166220885 (1).png)

如果需要用到远程客户端(比如 Tableau)连接到hive数据库，还需要启动hive service

​     nohup hive --service hiveserver2 > hiveserver2.log 2>&1 &

![159642-20170726155659578-1527911828](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726155659578-1527911828.png)

然后由于配置过环境变量，可以直接在命令行中输入hive

![159642-20170726155836156-1878050728 (1)](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726155836156-1878050728 (1).png)

**测试hive是否可以正确使用**

**创建测试表dep**![159642-20170726165311421-1909402269](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726165311421-1909402269.png)

**通过Mysql查看创建的表**![159642-20170726164157796-2140675966](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726164157796-2140675966.png)

**通过UI页面查看创建的数据位**

**访问 xxx.xxx.xxx.xxx:50070**

![159642-20170726165802093-1673183418](C:\Users\Administrator\Desktop\工作-大数据\课件\U3-Hive-Hbase数据库\img\159642-20170726165802093-1673183418.png)