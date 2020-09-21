---
layout: post
title: HDFS 的 操作（二）
categories: [Big Data]
description: Hadoop 之 HDFS
keywords: Big Data, Hadoop, HDFS
---

# HDFS 的 AIP 操作

## 在 HDFS 上创建文件夹

```java
//在HDFS上创建目录
public void testHDFSMakeDirectory() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //在HFDS上递归创建名为newDir04/newDir0401/newDir040101/的目录
    fs.mkdirs(new Path("/newDir04/newDir0401/newDir040101/"));
    //关闭文件系统对象
    fs.close();
}
```

------------------------------

## HDFS 的文件上传

```java
//文件上传至HDFS
public void testCopyFromLocalFile() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //本地路径既可以使用相对路径也可以使用绝对路径
    Path src = new Path("./src/put/put01");
    //HDFS只能使用绝对路径,可以在上传的时候顺便修改上传的文件名，但不能上传至不存在的目录下
    Path dst = new Path("/");
    //Path dst = new Path("/newDir02/put02");
    //从本地上传指HDFS上
    fs.copyFromLocalFile(src, dst);
    //关闭文件系统对象
    fs.close();
}

```

**注：put01 和 put02 均为文件而不是目录。**

------------------------------

## 从 HDFS 上下载文件
```java
//从HDFS中下载文件
public void testCopyToLocalFile() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //src路径是HDFS的绝对路径
    Path src = new Path("/newDir02/put01");
    //dst路径是本地路径
    Path dst = new Path("./src/get/");
    //Path dst = new Path("./src/get/get01");
    //从HDFS上下载到本地
    fs.copyToLocalFile(src, dst);
    //关闭文件系统对象
    fs.close();
}
```

**注：put01 和 get01 均为文件而不是目录。**

------------------------------

## 删除 HDFS 上的文件或目录
```java
//删除HDFS的文件或目录
public void testDelete() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    /*删除文件或目录，第二个参数代表是否递归删除，
    如果是目录且为true，此目录里的所有文件和目录也会被一同删除，
    如果是目录且为false，则必须此目录是空目录*/
    //fs.delete(new Path("/newDir02/put01"), false);
    fs.delete(new Path("/newDir04/newDir0401/newDir040101/"), false);
    //fs.delete(new Path("/newDir01/"), true);
    //关闭文件系统对象
    fs.close();
}

```

**注：put01 为文件而不是目录。**

------------------------------

## HDFS 上的文件或目录的重命名
```java
//HDFS文件或目录的重命名
public void testRename() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //需要修改的文件名或目录名
    Path src = new Path("/newDir02/put01");
    //修改后的文件名或目录名
    Path dst = new Path("/newDir02/put02");
    //重命名文件或目录
    fs.rename(src, dst);
    //关闭文件系统对象
    fs.close();
}

```

------------------------------

## HDFS 上的文件详情的查看
```java
//HDFS文件详情的查看
public void testListFiles() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //获取此路径下的所有文件状态的迭代器，只获得文件的详细信息而不获得此目录下的目录
    RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);

    while (listFiles.hasNext()){
        //获取迭代器中的一个文件状态
        LocatedFileStatus fileStatus = listFiles.next();
        //打印此文件的所有详细信息
        System.out.println(fileStatus);
        //逐个信息打印
        System.out.println("文件路径及文件名：" + fileStatus.getPath());
        System.out.println("文件名：" + fileStatus.getPath().getName());
        System.out.println("文件大小：" + fileStatus.getLen());
        System.out.println("文件副本数：" + fileStatus.getReplication());
        System.out.println("文件权限：" + fileStatus.getPermission());

        BlockLocation[] blockLocations = fileStatus.getBlockLocations();

        for (BlockLocation b:blockLocations) {
            System.out.println("文件数据块信息：" + b);
        }
        System.out.println("-------分割线-------");
    }
    //关闭文件系统对象
    fs.close();
}

```

------------------------------

## HDFS 上的文件和目录的判断
```java
//HDFS文件和目录的判断
public void testListStatus() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //获取路径下的所有文件和目录的状态表，不递归
    FileStatus[] listStatus = fs.listStatus(new Path("/"));
    //迭代器迭代过程，判断是文件还是目录
    for (FileStatus fileStatus:listStatus) {
        if (fileStatus.isDirectory()){
            System.out.println("目录：" + fileStatus.getPath().getName());
        }else if (fileStatus.isFile()){
            System.out.println("文件：" + fileStatus.getPath().getName());
        }
    }
    //关闭文件系统对象
    fs.close();
}

```

------------------------------

# HDFS 的 IO 流操作

## 以 IO 流的方式传输文件到 HDFS 上
```java
//以IO流的方式传输文件到HDFS上
public void testPutFileToHDFS() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //获取输入流
    FileInputStream fis = new FileInputStream(new File("./src/put/put01"));
    //获取输出流
    FSDataOutputStream fos = fs.create(new Path("/put02"));
    //文件传输
    IOUtils.copyBytes(fis, fos, conf);
    //关闭资源
    IOUtils.closeStream(fos);
    IOUtils.closeStream(fis);
    fs.close();
}

```

------------------------------

## 以 IO 流的方式从 HDFS 上下载文件
```java
//以IO流的方式从HDFS上下载文件
public void testGetFileFromHDFS() throws URISyntaxException, IOException, InterruptedException {
    //创建一个config对象
    Configuration conf = new Configuration();
    //创建一个文件系统对象（HDFS的客户端）
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop01:9000"), conf, "hadoop");
    //获取输入流
    FSDataInputStream fis = fs.open(new Path("/put02"));
    //获取输出流
    FileOutputStream fos = new FileOutputStream(new File("./src/get/get02"));
    //文件传输
    IOUtils.copyBytes(fis, fos, conf);
    //关闭资源
    IOUtils.closeStream(fis);
    IOUtils.closeStream(fos);
    fs.close();
}

```