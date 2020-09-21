---
layout: post
title: MapReduce 中自定义 OutputFormat
categories: [Big Data]
description: MapReduce 中自定义 OutputFormat
keywords: Big Data, Hadoop, MapReduce
---

## 自定义 OutputFormat

我们知道，默认的文件加载类是：TextOutputFormat，默认的文件读取是：LineRecordWriter，所以如果要自定义 OutputFormat，我们需要重写这两个类。

我们以一个例子来自定义 OutputFormat。

需求：将考试成绩及格的输出到一个文件夹（/kaoshi/jige/），将考试成绩不及格的输出到另一个文件夹（/kaoshi/bujige/）,成功与否标志文件输出的到 /scoreout。

**注：与分区不同，分区只能输出到同一个文件夹下的不同文件下。**

输入：

    /scorein/score.txt
    数学,涂涂,88,99,100,60,70
    语文,憨憨,55,56,57,58,59
    计算机,涂涂,99,99,99,99,99
    计算机,憨憨,59,59,59,59,59
    语文,涂涂,30,40,50,60,70
    数学,憨憨,50,60,70,80,90

输出：

    /kaoshi/jige/jige.txt
    数学	憨憨	70.0
    数学	涂涂	83.4
    计算机	涂涂	99.0

    /kaoshi/bujige/bujige.txt
    计算机	憨憨	59.0
    语文	憨憨	57.0
    语文	涂涂	50.0


DiffDic -- Mapper、Reducer、Driver：
```java
/**
 * 需求：不同结果输出到不同的文件夹下
 * 统计平均成绩   成绩低于60分的写到/chengji/bujige/bujige.txt    成绩不低于60分的写到/chengji/jige/jige.txt
 */
public class DiffDic {

    static class MyMapper extends Mapper<LongWritable, Text, Text, IntWritable>{

        Text mapKey = new Text();
        IntWritable mapValue = new IntWritable();

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String[] datas = value.toString().split(",");
            String k = datas[0] + "\t" + datas[1];
            mapKey.set(k);

            for (int i=2; i<datas.length; i++){
                mapValue.set(Integer.parseInt(datas[i]));
                context.write(mapKey, mapValue);
            }
        }
    }

    static class MyReducer extends Reducer<Text, IntWritable, Text, DoubleWritable>{

        DoubleWritable value = new DoubleWritable();

        @Override
        protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int sum = 0;
            int count = 0;
            for (IntWritable v:values) {
                sum+=v.get();
                count++;
            }
            double avg = (double)sum/count;
            value.set(avg);
            context.write(key,value);
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        args = new String[]{"/scorein","/scoreout"};

        Configuration conf = new Configuration();

        //1 获取 Job 对象
        Job job = Job.getInstance(conf);

        //2 设置 jar 存储位置
        job.setJarByClass(DiffDic.class);

        //3 关联 Map 和 Reduce 类
        job.setMapperClass(MyMapper.class);
        job.setReducerClass(MyReducer.class);

        //4 设置 Mapper 阶段输出数据的 key 和 value 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        //5 设置最终数据输出的 key 和 value 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(DoubleWritable.class);

        // 指定自定义输出
        job.setOutputFormatClass(DiffDicOutputFormat.class);

        //6 设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //7 提交 job
        //job.submit();//此方法不会打印日志
        boolean result = job.waitForCompletion(true);

        System.exit(result?0:1);
    }
}

```

DiffDicOutputFormat 继承 FileOutputFormat 并重写其方法：
```java
public class DiffDicOutputFormat extends FileOutputFormat<Text, DoubleWritable> {

    //获取 RecordWriter 对象
    @Override
    public RecordWriter<Text, DoubleWritable> getRecordWriter(TaskAttemptContext job) throws IOException, InterruptedException {
        FileSystem fs = FileSystem.get(job.getConfiguration());

        return new DiffRecordWriter(fs);
    }
}
```

DiffRecordWriter 继承 RecordWriter 并重写其方法：
```java
public class DiffRecordWriter extends RecordWriter<Text, DoubleWritable> {
    FSDataOutputStream out01 = null;
    FSDataOutputStream out02 = null;
    FileSystem fs;


    public DiffRecordWriter(FileSystem fs) throws IOException {
        this.fs = fs;
        out01 = fs.create(new Path("/kaoshi/jige/jige.txt"));
        out02 = fs.create(new Path("/kaoshi/bujige/bujige.txt"));

    }

    // 真正向外写出的方法

    /**
     * 需求：不同成绩输出到不同文件夹。--则需要两个输出流
     * 流可通过 filesystem 对象，但 filesystem 对象需要通过配置文件获取
     * @param key
     * @param value
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public void write(Text key, DoubleWritable value) throws IOException, InterruptedException {
        double score = value.get();
        if (score >= 60){
            out01.write((key.toString() + "\t" + Double.toString(score) + "\n").getBytes("utf-8"));
        }else {
            out02.write((key.toString() + "\t" + Double.toString(score) + "\n").getBytes("utf-8"));
        }
    }

    //关闭资源的方法
    @Override
    public void close(TaskAttemptContext context) throws IOException, InterruptedException {
        out01.close();
        out02.close();
    }
}
```