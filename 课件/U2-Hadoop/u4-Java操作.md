#### **要点导航**

- ##### 实例1：使用java.net.URL访问HDFS文件系统
- ##### 实例2：使用FileSystem访问HDFS文件系统
- ##### 实例3：创建HDFS目录
- ##### 实例4：删除HDFS目录
- ##### 实例5：查看文件或目录是否存在
- ##### 实例6：列出目录下的文件或目录名称
- ##### 实例7：查看文件存储位置
- ##### 实例8：将本地文件写入到HDFS中

#### **使用java.net.URL访问HDFS文件系统**

##### HDFS的API使用说明

1. 如果要访问HDFS，HDFS客户端必须有一份HDFS的配置文件也就是hdfs-site.xml,从而读取Namenode的信息。


2. 每个应用程序也必须拥有访问Hadoop程序的jar文件

    3. 操作HDFS,也就是HDFS的读和写,最常用的类FileSystem

###### **实例1：使用java.net.URL访问HDFS文件系统**

>  操作：显示HDFS文件夹中的文件内容
>
>  * 使用java.net.URL对象打开数据流
>  * 使用静态代码块使得java程序识别Hadoop的HDFS url

```
 package TestHdfs;
 
 import java.io.InputStream;
 import java.net.URL;
 import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
 import org.apache.hadoop.io.IOUtils;
 /**
 * @author SimonsZhao
 * HDFS的API使用
 * 1.如果要访问HDFS，HDFS客户端必须有一份HDFS的配置文件
 * 也就是hdfs-site.xml,从而读取Namenode的信息。
 * 2.每个应用程序也必须拥有访问Hadoop程序的jar文件
 * 3.操作HDFS,也就是HDFS的读和写,最常用的类FileSystem
 * 操作：显示HDFS文件夹中的文件内容
 * 1.使用java.net.URL对象打开数据流
 * 2.使用静态代码块使得java程序识别Hadoop的HDFS url
 */
 public class MyCat {
      static{
          URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
      }
      public static void main(String[] args) {
          InputStream input=null;
          try {
             input = new URL(args[0]).openStream();
              IOUtils.copyBytes(input,System.out,4096,false);
          } catch (Exception e) {
             System.err.println("Error");
          }finally{
              IOUtils.closeStream(input);
          }
      }
 }
```

>打包MyCat类为jar包
>
>1. 在指定文件夹下创建示例文件demo
>
>   [root@neusoft-master filecontent]# vi demo
>
>2. 上传文件至HDFS的data目录，data目录需要首先创建。
>
>   [root@neusoft-master filecontent]# hadoop dfs -put demo /data/
>
>3. 查看是否上传成功
>
>   [root@neusoft-master filecontent]# hadoop dfs -ls /data/
>
>4. 将已经打包好的jar文件上传至linux并切换到相应文件夹运行hadoop命令执行
>
>   从结果可以看出能够显示出来demo文件的内容
>
>   [root@neusoft-master filecontent]# hadoop jar MyCat.jar hdfs://neusoft-master:9000/data/demo

######  **实例2：使用FileSystem访问HDFS文件系统**

> 操作：将本地文件系统的文件通过java-API写入到HDFS文件
>
> 1.本地文件系统和HDFS中应该首先创建指定的目录
>
>     linux中创建文件命令：mkdir test
>
>     HDFS中创建文件夹命令：hadoop dfs -mkdir /data/

```
 package TestHdfs;
 
 import java.io.BufferedInputStream;
 import java.io.FileInputStream;
 import java.io.FileNotFoundException;
 import java.io.InputStream;
 import java.io.OutputStream;
 import java.net.URI;
  
 import org.apache.hadoop.conf.Configuration;
 import org.apache.hadoop.fs.FSDataOutputStream;
 import org.apache.hadoop.fs.FileSystem;
 import org.apache.hadoop.fs.Path;
 import org.apache.hadoop.io.IOUtils;
 
/**
  * @author SimonsZhao
  *将本地文件系统的文件通过java-API写入到HDFS文件
  */
 public class FileCopyFromLocal {
     public static void main(String[] args) throws Exception {
         String source="/usr/local/filecontent/demo";//linux中的文件路徑,demo存在一定数据
        String destination="hdfs://neusoft-master:9000/data/test";//HDFS的路徑
         InputStream in = new BufferedInputStream(new FileInputStream(source));
         //HDFS读写的配置文件
         Configuration conf = new Configuration();
         //调用Filesystem的create方法返回的是FSDataOutputStream对象
         //该对象不允许在文件中定位，因为HDFS只允许一个已打开的文件顺序写入或追加
         FileSystem fs = FileSystem.get(URI.create(destination),conf);
         OutputStream out = fs.create(new Path(destination));
         IOUtils.copyBytes(in, out, 4096, true);
     }
}
```

> 1. 程序打包并传至linux文件系统中
>
>    请参考实例1的打包过程
>
> 2. 程序运行及结果分析
>
>    a. 查看指定jar包是否成功上传，在linux中使用ls或ll命令
>
>    b. 执行jar命令
>
>    ```
>    [root@neusoft-master filecontent]# hadoop jar FileSystemDemoCat.jar 
>    ```

###### **实例3：创建HDFS目录**

> 创建HDFS目录
>
> 实例：HDFS创建test2目录 　　
>
> 1. 明确在HDFS文件系统中创建目录的具体地址，在程序中通过args[0]参数提供用户输入，如
>
>    ```
>    hdfs://neusoft-master:9000/data/test2
>    ```

 ```
package TestHdfs;

import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
/**
 * @author SimonsZhao
 * 创建HDFS目录
 * 实例：HDFS创建test2目录
 * hadoop jar CreateDir.jar hdfs://neusoft-master:9000/data/test2
 */
 public class CreateDirction {
     public static void main(String[] args) {
         //HDFS路径：hdfs://neusoft-master:9000/data/test2
         String uri=args[0];//从键盘输入路径参数
         Configuration conf = new Configuration();
         try {
            FileSystem fs = FileSystem.get(new URI(uri),conf);
             Path dfs = new Path(uri);
             fs.mkdirs(dfs);
         } catch (Exception e) {
             e.printStackTrace();
         }finally{
             System.out.println("SUCESS");
         }
     }
 }
 ```

> 1. 将jar包上传到Linux
>
>    请参考第一个程序的导出jar包的过程。 
>
> 2. 程序运行及结果分析
>
>    ```
>    [root@neusoft-master filecontent]# hadoop jar CreateDir.jar hdfs://neusoft-master:9000/data/test2
>    ```

###### **实例4：删除HDFS目录**

```
package TestHdfs;

import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
/**
 * @author SimonsZhao
 * 删除HDFS上面的文件
 */
public class DeleteFile {
    public static void main(String[] args) {
        String uri="hdfs://neusoft-master:9000/data/test2";
        Configuration conf = new Configuration();
        try {
            FileSystem fs =FileSystem.get(new URI(uri), conf);
            Path f = new Path(uri);
            //递归删除文件夹下所有文件
            boolean isDelete= fs.delete(f, true);
            //递归删除文件夹下所有文件
            //boolean isDelete= fs.delete(f, false);
            String str=isDelete?"Sucess":"Error";
            System.out.println("删除"+str);
        } catch (Exception e) {
            System.out.println("删除出错~");
        }
    }
}
```

> 1. 将jar包上传到Linux
>
>    请参考第一个程序的导出jar包的过程。 
>
> 2. 程序运行及结果分析
>
>    ```
>    执行程序之后，通过hadoop dfs -ls / 查看是否成功删除HDFS上面的文件
>    ```

###### **实例5：查看文件或目录是否存在**

```
package TestHdfs;
import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
/**
 * @author SimonsZhao
 * 查看文件是否存在
 */
public class CheckFileIsExists {
    public static void main(String[] args) {
        //String uri="hdfs://neusoft-master:9000/data/test2/";//指定目录
        String uri="hdfs://neusoft-master:9000/data/test2/hello";//指定文件
        Configuration conf = new Configuration();
        try {
            FileSystem fs = FileSystem.get(new URI(uri), conf);
            Path path = new Path(uri);
            boolean isExists=fs.exists(path);
            String str=isExists?"Exists":"Not Exists";
            System.out.println("指定文件或目录"+str);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

>  1. 将jar包上传到Linux
>
>     请参考第一个程序的导出jar包的过程。 
>
>  2. 程序运行及结果分析
>
>     ```
>     如果在linux中存在该文件的话，则显示：“指定文件或目录Exists”
>     ```

###### **实例6：列出目录下的文件或目录名称**

```
package TestHdfs;

import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
/**
 * @author SimonsZhao
 * 列出目录下的文件或目录名称
 */
 public class ListFiles {
     public static void main(String[] args) {
         String uri="hdfs://neusoft-master:9000/data";
         Configuration conf = new Configuration();
         try {
             FileSystem fs=FileSystem.get(new URI(uri), conf);
             Path path = new Path(uri);
             FileStatus status[] = fs.listStatus(path);
             for (int i = 0; i < status.length; i++) {
                 System.out.println(status[i].getPath().toString());
             }
         } catch (Exception e) {
             e.printStackTrace();
         }
     }
 }
```

> 1. 将jar包上传到Linux
>
>    请参考第一个程序的导出jar包的过程。 
>
> 2. 程序运行及结果分析
>
>    ![hadoop列目录](C:\Users\Administrator\Desktop\工作-大数据\课件\U2-Hadoop\img\hadoop列目录.png)

###### **实例7：查看文件存储位置**

```
package TestHdfs;

import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.BlockLocation;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
/**
 * @author SimonsZhao
 * 文件存储的位置
 */
public class LoactionFile {
    public static void main(String[] args) {
        String uri="hdfs://neusoft-master:9000/data/demo";//hello为文件
        Configuration conf = new Configuration();
        try {
            FileSystem fs=FileSystem.get(new URI(uri), conf);
            Path path = new Path(uri);
            FileStatus fileStatus = fs.getFileStatus(path);
            BlockLocation blkLocation[]=fs.getFileBlockLocations(fileStatus, 0, fileStatus.getLen());
            for (int i = 0; i < blkLocation.length; i++) {
                String[] hosts=blkLocation[i].getHosts();
            	System.out.println("block_"+i+"_Location:"+hosts[0]);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

> 1. 将jar包上传到Linux
>
>    请参考第一个程序的导出jar包的过程。 
>
> 2. 程序运行及结果分析
>
>    ```
>    由于采用伪分布的环境block块存储均为1，因此这里仅显示1个block块的host主机名显示：block_0_Location:neusoft-master
>    ```

###### **实例8：将本地文件写入到HDFS中**

```
package TestHdfs;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
 
/**
 * @author SimonsZhao
 *将本地文件系统的文件通过java-API写入到HDFS文件
 */
public class FileCopyFromLocal {
    public static void main(String[] args) throws Exception {
        String source="/usr/local/filecontent/demo";//linux中的文件路徑,demo存在一定数据
        String destination="hdfs://neusoft-master:9000/data/test";//HDFS的路徑
        InputStream in = new BufferedInputStream(new FileInputStream(source));
        //HDFS读写的配置文件
        Configuration conf = new Configuration();
        //调用Filesystem的create方法返回的是FSDataOutputStream对象
        //该对象不允许在文件中定位，因为HDFS只允许一个已打开的文件顺序写入或追加
        FileSystem fs = FileSystem.get(URI.create(destination),conf);
        OutputStream out = fs.create(new Path(destination));
        IOUtils.copyBytes(in, out, 4096, true);
    }
}
```

> 1. 将jar包上传到Linux
>
>    请参考第一个程序的导出jar包的过程。 
>
> 2. 程序运行及结果分析
>
>    将本地的demo文件写入HDFS的data目录下的test文件