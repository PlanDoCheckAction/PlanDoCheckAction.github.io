---
layout: post
title: MapReduce 自定义可序列化对象
categories: [Big Data]
description: MapReduce 序列化
keywords: Big Data, Hadoop, MapReduce
---

## 什么是序列化

通俗的讲，序列化就是将内存当中的数据对象转换成字节序列（或其他数据传输协议），以便于存储（持久化）到磁盘或者进行网络传输。

除了序列化以外，还有一个名词为反序列化，很容易理解，反序列化就是将收到的字节序列（或其他数据传输协议）或者是磁盘的持久化数据转换成内存当中的数据对象。

## 为什么要序列化

一般来说，内存当中的数据对象，在我们机器关机断电后就没有了，并且内存当中的数据对象只能由本地的进程进行使用，不能被发送到网络上的另一台计算机上，然后序列化可以将内存当中的数据对象写到磁盘，然后我们就可以以文件的形式将数据对象发送到远程的计算机上。

## MapReduce 为什么不用 Java 自带的序列化方法

Java 自带的序列化是一个重量级的序列化框架（ Serializable ），一个对象被序列化后，会附带很多额外的信息（各种校验信息，Header，继承体系等等），这不便于在网络中高效的传输，所以，Hadoop 自己开发了一套序列化机制（ Writable ）。

Hadoop 序列化特点：

1. 紧凑：高效的使用存储空间；
2. 快速：读写数据的额外开销小；
3. 可扩展：随着通信协议的升级而可升级；
4. 互操作：支持多语言的交互。

## 自定义 bean 对象实现序列化接口（ Writable ）

在项目开发过程中，往往常用的 Hadoop 基本序列化类型不能满足所有需求，比如在 Hadoop 框架内部传递一个 bean 对象，那么该对象就需要实现序列化接口。

具体实现 bean 对象序列化步骤如下 7 步：

1. 必须实现 Writable 接口；
2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造函数；
3. 实现序列化方法；
4. 实现反序列化方法；
5. **注意反序列化属性参数的顺序和序列化属性参数的顺序完全一致；**
6. 要想把结果显示在文件中，需要重写 toString() 方法，可用「 \t 」分开，方便后续使用；
7. 如果需要将自定义的 bean 放在 key 当中进行传输，则还需要实现 Comparable 接口，因为 MapReduce 框架中的 Shuffle 过程要求对 key 必须能排序。

## 序列化实例

我们以统计每个手机号耗费的总上行流量、总下行流量和总流量为例来说明如何自定义可序列化的 bean 对象。

### 需求分析

此例的需求：统计每一个手机耗费的总上行流量、总下行流量、总流量。

1. 输入数据：
![](/images/posts/hadoop/BigData11-PhoneData.png)

2. 输入数据的格式：
```
   手机号码              网络ip      上行数据包  下行数据包  上行流量 下行流量 网络状态码
   11111111111    120.196.100.82       24          27       2481    24681     200
```
3. 期望输出数据格式：
```
   手机号码          总上行流量          总下行流量        总流量
   11111111110	        4088	        190572	        194660
```
4. Map 阶段

读取每一行数据并切分出我们所需要的信息（手机号码、上行流量、下行流量），然后以手机号码为输出的 key，bean 对象为输出的 value，即 context.write(手机号码, bean对象); 。

**注：bean 对象要想能够在 MapReduce 中传输则必须实现 Hadoop 的序列化接口；bean 对象若作为 key，则还必须实现 Comparable 接口。**

5. Reduce 阶段

将每个相同 key 的一组 value 值（即一组 bean 对象）按需求进行操作，这里我们是将 key 对应的一组 bean 对象内部的上行流量、下行流量分别求和，最后再求总和得到总流量。

### 编写对应的 MapReduce 程序

1. 编写流量统计的 bean 对象对应的类：
```java
public class FlowBean implements Writable {

    private long upFlow;//上行流量
    private long downFlow;//下行流量
    private long sumFlow;//总流量

    // 反序列化时，需要反射调用空参构造函数，所以必须有空参构造函数
    public FlowBean() {
        super();
    }

    public FlowBean(long upFlow, long downFlow) {
        super();
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = upFlow + downFlow;
    }

    // 序列化方法
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFlow);
        out.writeLong(sumFlow);
    }

    // 反序列方法，反序列化方法读的顺序必须和序列化方法写的顺序一致
    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow = in.readLong();
        this.downFlow = in.readLong();
        this.sumFlow = in.readLong();
    }

    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }
}
```

2. 编写 Mapper 类：
```java
public class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean> {

    FlowBean v = new FlowBean();;
    Text k = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        // 获取文件中每一行的内容并按照制表符切分
        String[] fields = value.toString().split("\t");

        String phoneNumber = fields[0];

        Long upFlow = Long.parseLong(fields[fields.length-3]);
        Long downFlow = Long.parseLong(fields[fields.length-2]);

        k.set(phoneNumber);
        v.setUpFlow(upFlow);
        v.setDownFlow(downFlow);
        v.setSumFlow(upFlow + downFlow);

        context.write(k, v);
    }
}
```

3. 编写 Reducer 类：
```java
public class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean> {

    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {
        long sum_upFlow = 0;
        long sum_downFlow = 0;

        // 计算总上行流量和总下行流量
        for (FlowBean value:values) {
            sum_upFlow += value.getUpFlow();
            sum_downFlow += value.getDownFlow();
        }

        // 封装对象
        FlowBean resultFlow = new FlowBean(sum_upFlow, sum_downFlow);

        context.write(key,resultFlow);
    }
}
```

4. 编写驱动类：
```java
public class FlowCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();

        //1 获取 Job 对象
        Job job = Job.getInstance(conf);

        //2 设置 jar 存储位置
        job.setJarByClass(FlowCountDriver.class);

        //3 关联 Map 和 Reduce 类
        job.setMapperClass(FlowCountMapper.class);
        job.setReducerClass(FlowCountReducer.class);

        //4 设置 Mapper 阶段输出数据的 key 和 value 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);

        //5 设置最终数据输出的 key 和 value 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //6 设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));

        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //7 提交 job
        //job.submit();//此方法不会打印日志
        job.waitForCompletion(true);
    }
}

```

### 输出结果

![](/images/posts/hadoop/BigData11-Result.png)