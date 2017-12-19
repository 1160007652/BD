## 数据排序

**数据排序**是许多实际任务执行时要完成的第一项工作，比如学生成绩评比、数据建立索引等。这个实例和数据去重类似，都是先对原始数据进行初步处理，为进一步的数据操作打好基础。下面进入这个示例。

> ##### **实例描述**

对输入文件中数据进行排序。输入文件中的每行内容均为一个数字，即一个数据。要求在输出中每行有两个间隔的数字，其中，第一个代表原始数据在原始数据集中的位次，第二个代表原始数据。

- 样例输入：

    文件1中的内容：
    2
    32
    654
    32
    15
    756
    65223
    ----------------------------------
    文件2中的内容：
    5956
    22
    650
    92
    ----------------------------------
    文件3中的内容：
    26
    54
    6
- 样列输出

```
1    2
2    6
3    15
4    22
5    26
6    32
7    32
8    54
9    92
10    650
11    654
12    756
13    5956
14    65223
```

> ##### **设计思路**

​	这个实例仅仅要求对输入数据进行排序，熟悉MapReduce过程的读者会很快想到在MapReduce过程中就有排序，是否可以利用这个默认的排序，而不需要自己再实现具体的排序呢？答案是肯定的。
​	但是在使用之前首先需要了解它的默认排序规则。它是按照key值进行排序的，如果key为封装int的IntWritable类型，那么MapReduce按照数字大小对key排序，如果key为封装为String的Text类型，那么MapReduce按照字典顺序对字符串排序。
​	了解了这个细节，我们就知道应该使用封装int的IntWritable型数据结构了。也就是在map中将读入的数据转化成 IntWritable型，然后作为key值输出（value任意）。reduce拿到\<key，value-list>之后，将输入的 key作为value输出，并根据value-list中元素的个数决定输出的次数。输出的key（即代码中的linenum）是一个全局变量，它统计当前key的位次。需要注意的是这个程序中没有配置Combiner，也就是在MapReduce过程中不使用Combiner。这主要是因为使用map和reduce就已经能够完成任务了。

> 程序代码

```
package com.hebut.mr;
 
import java.io.IOException;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
 
public class Sort {
 
    //map将输入中的value化成IntWritable类型，作为输出的key
    public static class Map extends
　　　　　　　　Mapper<Object,Text,IntWritable,IntWritable>{
        private static IntWritable data=new IntWritable();
       
        //实现map函数
        public void map(Object key,Text value,Context context)
                throws IOException,InterruptedException{
            String line=value.toString();
            data.set(Integer.parseInt(line));
            context.write(data, new IntWritable(1));
        }
       
    }
   
    //reduce将输入中的key复制到输出数据的key上，
    //然后根据输入的value-list中元素的个数决定key的输出次数
    //用全局linenum来代表key的位次
    public static class Reduce extends
            Reducer<IntWritable,IntWritable,IntWritable,IntWritable>{
       
        private static IntWritable linenum = new IntWritable(1);
       
        //实现reduce函数
        public void reduce(IntWritable key,Iterable<IntWritable> values,Context context)
                throws IOException,InterruptedException{
            for(IntWritable val:values){
                context.write(linenum, key);
                linenum = new IntWritable(linenum.get()+1);
            }
           
        }
 
    }
   
    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration();
        //这句话很关键
        conf.set("mapred.job.tracker", "192.168.1.2:9001");
       
        String[] ioArgs=new String[]{"sort_in","sort_out"};
     String[] otherArgs = new GenericOptionsParser(conf, ioArgs).getRemainingArgs();
     if (otherArgs.length != 2) {
     System.err.println("Usage: Data Sort <in> <out>");
         System.exit(2);
     }
     
     Job job = new Job(conf, "Data Sort");
     job.setJarByClass(Sort.class);
     
     //设置Map和Reduce处理类
     job.setMapperClass(Map.class);
     job.setReducerClass(Reduce.class);
     
     //设置输出类型
     job.setOutputKeyClass(IntWritable.class);
     job.setOutputValueClass(IntWritable.class);
     
     //设置输入和输出目录
     FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
     FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
     System.exit(job.waitForCompletion(true) ? 0 : 1);
     }
}
```

> ##### **程序结果**

**1. 准备测试数据**

​    通过Eclipse下面的"DFS Locations"在"/user/hadoop"目录下创建输入文件"sort_in"文件夹（**备注**："sort_out"不需要创建。）如图2.4-1所示，已经成功创建。

![img](http://images.cnblogs.com/cnblogs_com/xia520pi/201206/20120604132405862.png)              ![img](http://images.cnblogs.com/cnblogs_com/xia520pi/201206/201206041324056162.png)

​                  图2.4-1 创建"sort_in"                                                        图2.4.2 上传"file*.txt"

 

​	然后在本地建立三个txt文件，通过Eclipse上传到"/user/hadoop/sort_in"文件夹中，三个txt文件的内容如"实例描述"那三个文件一样。如图2.4-2所示，成功上传之后。

​	从SecureCRT远处查看"Master.Hadoop"的也能证实我们上传的三个文件。

![img](http://images.cnblogs.com/cnblogs_com/xia520pi/201206/20120604132405273.png)

查看两个文件的内容如图2.4-3所示：

![img](http://images.cnblogs.com/cnblogs_com/xia520pi/201206/201206041324069160.png)

​                                                             图2.4-3 文件"file*.txt"内容

**2）查看运行结果**

​	这时我们**右击**Eclipse 的"DFS Locations"中"/user/hadoop"文件夹进行刷新，这时会发现多出一个"sort_out"文件夹，且里面有3个文件，然后打开双 其"part-r-00000"文件，会在Eclipse中间把内容显示出来。如图2.4-4所示。