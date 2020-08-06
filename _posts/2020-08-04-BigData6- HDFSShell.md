---
layout: post
title: HDFS 的 操作（一）
categories: [Big Data]
description: Hadoop 之 HDFS
keywords: Big Data, Hadoop, HDFS
---

# 开启 HDFS

## 开启 Hadoop 或 HDFS 的客户端

首先，我们要对 HDFS 进行读写操作的话，我们得先开启一个客户端（Client），因为用户只能通过客户端来进行对 HDFS 的读写操作，如下图所示。

![](/images/posts/hadoop/BigData5-HDFSArchitecture.png)

用命令的方式对 HDFS 进行文件的上传或下载之前，我们得先开启一个客户端。

客户端的命令在 hadoop/bin 目录下。

![](/images/posts/hadoop/BigData6-HDFSClientCommand.png)

**注：.cmd 结尾的命令是 Window 操作系统下对应的命令。**

![](/images/posts/hadoop/BigData6-HadoopFSCommand1.png)

打开 hadoop 命令可以看到这么一句话：

```
fs   run a generic filesystem user client
```

然后我们再进到 hadoop fs 命令中看一下：

![](/images/posts/hadoop/BigData6-HadoopFSCommand2.png)

可以看到，HDFS 对文件或目录的操作与 Linux 对文件或目录的操作基本一样，只是前面添加了 hadoop fs 而已。

------------------------------

## HDFS 的命令操作

从上面的图片中我们可以看到有很多对目录或者对文件操作的命令，这里仅举例在 HDFS 上创建目录的命令，其余命令的操作均类似：

```
hadoop fs -mkdir 路径+目录名
```

**注：所有有关在 HDFS 上的路径均是从根目录开始的绝对路径。**

![](/images/posts/hadoop/BigData6-HDFSMKDIR1.png)

从 hadoop fs -ls / 中我们可以看到 HDFS 的抽象根目录下已经创建了一个 newDir01 子目录，除了使用命令的方式去查看，我们也可以使用 Web 端查看：

![](/images/posts/hadoop/BigData6-HDFSMKDIR2.png)

------------------------------

## Java 操作 HDFS

除了使用命令的方式对 HDFS 进行操作外，我们还可以使用 Java 代码的方式去操作 HDFS，但在使用 Java 去操纵 HDFS 之前，我们需要先准备好操纵 HDFS 所需 Java 依赖包，这里我使用 Maven 去构建项目：

```xml
<!-- hadoop 公共依赖 -->
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.7</version>
</dependency>

<!-- hdfs 依赖 -->
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-hdfs</artifactId>
    <version>2.7.7</version>
</dependency>
```

那 Java 代码是如何操作 HDFS 的呢？举个例子，使用代码在 HDFS 的抽象根目录上创建一个 newDir02 目录：

```java

public static void main(String[] args) throws URISyntaxException, IOException, InterruptedException {


        //创建一个config对象
        Configuration conf = new Configuration();

        //创建一个文件系统对象（HDFS的客户端）
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");

        //在HFDS上创建一个名为newDir02的目录
        fs.mkdirs(new Path("/newDir02/"));

        //关闭文件系统对象
        fs.close();

    }

```

创建完成后，我们可以在 Web 端查看是否创建成功：

![](/images/posts/hadoop/BigData6-HDFSMKDIR3.png)

------------------------------

## HDFS 上传文件后

假设我们已经从本地上传了一个文件 hadoop.tar.gz 到 HDFS 的 /newDir01/ 目录下，这时我们就可以通过 Web 端查看文件的详细信息了：

![](/images/posts/hadoop/BigData6-HDFSPUT1.png)

![](/images/posts/hadoop/BigData6-HDFSPUT2.png)