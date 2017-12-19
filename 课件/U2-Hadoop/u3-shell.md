所有的**HDFS shell**操作命名可以通过**hadoop fs**获取

> [root@hadoop ~]# hadoop fs

常见操作

```
列出HDFS文件下面所有的文件
[root@hadoop ~]# hadoop fs -ls hdfs://hadoop:9000/
hdfs://hadoop:9000 为hadoop配置文件core-site.xml中配置的默认的文件系统名称，上述命令可以简写为：
[root@hadoop ~]# hadoop fs -ls /
文件上传：讲Llinux下的/usr/local/hadoop-1.1.2.tar.gz上传到hdfs下的/download文件夹下
[root@hadoop ~]# hadoop fs -ls /usr/local/hadoop-1.1.2.tar.gz  /download
查看上传的文件：循环列出/download下面的所有文件
[root@hadoop ~]# hadoop fs -lsr  /download
```

操作命令帮助

> [root@hadoop ~]# hadoop fs -help chown