---
layout: post
title: MapReduce 中的 FileInputFormat 解析和自定义 FileInputFormat
categories: [Big Data]
description: MapReduce 中的 InputFormat 和自定义 InputFormat
keywords: Big Data, Hadoop, MapReduce
---

## FileInputFormat

我们知道，在 MapReduce 计算程序当中，一个任务的文件加载是依靠 FileInputFormat 类，这个类是一个抽象类，而在 Hadoop 中，默认使用的实现类是 TextInputFormat。

我们知道，文件的载入是在每一个 Maptask 当中进行的，所以我们可以进入 Mapper 类当中查看文件是如何进行载入的：

```java

public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {

  public abstract class Context
    implements MapContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {
  }

  protected void setup(Context context
                       ) throws IOException, InterruptedException {
    // NOTHING
  }

  @SuppressWarnings("unchecked")
  protected void map(KEYIN key, VALUEIN value, 
                     Context context) throws IOException, InterruptedException {
    context.write((KEYOUT) key, (VALUEOUT) value);
  }

  protected void cleanup(Context context
                         ) throws IOException, InterruptedException {
    // NOTHING
  }
  
  public void run(Context context) throws IOException, InterruptedException {
    setup(context);
    try {
      while (context.nextKeyValue()) {
        map(context.getCurrentKey(), context.getCurrentValue(), context);
      }
    } finally {
      cleanup(context);
    }
  }
}


```

我们可以看到 Mapper 类中可以大致分为初始化方法、结束方法、map 方法，然后这些方法都是在 run 方法上运行的。

所以我们主要看一下 run 方法的运行流程：
```java
public void run(Context context) throws IOException, InterruptedException {
    setup(context);
    try {
        // context.nextKeyValue() 判断是否还有下一关K,V可以读取
        // textInputFormat 表示的就是是否还有下一行内容进行读取
        while (context.nextKeyValue()) {
        /**
            * context.getCurrentKey() 获取当前的 Key -- textInputFormat 当中就指获取当前行偏移量
            * context.getCurrentValue() 获取当前的 Value -- textInputFormat 当中就指获取当前行的内容
            * context 上下文对象--用于 map 方法中向 reduce 进行写操作
            */
        map(context.getCurrentKey(), context.getCurrentValue(), context);
        }
    } finally {
        cleanup(context);
    }
}
```

我们可以看到这个方法里面大部分内容都与 context 相关，所以我们需要再了解一下 context 是从哪传过来的，我们可以从 run 方法中看到，是谁调用的 run 方法就意味着是谁传过来的 context。

我们知道，一个 maptask 启动一次 map 方法，即一个 maptask 运行一次 run 方法，所以 run 方法是在 MapTask 类当中调用的：
```java

private void runNewMapper(...){
...

// 利用反射机制创建一个 mapper 类，taskContext.getMapperClass()是利用任务的上下问对象获取 Mapper 类
org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE> mapper =
    (org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>)
    ReflectionUtils.newInstance(taskContext.getMapperClass(), job);

// 创建了一个 Mapper.Context 对象，但 mapperContext 指向的是一个 WrappedMapper 实例对象
org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>.Context 
        mapperContext = 
          new WrappedMapper<INKEY, INVALUE, OUTKEY, OUTVALUE>().getMapContext(
              mapContext);

//调用 run 方法，Mapper 类的 run 方法中传的 context 就是这里的 mapperContext
mapper.run(mapperContext);

...
}
```

由上可知，run 方法当中的 context 即是 MapTask 类中的 mapperContext，而 mapperContext 实例化了一个 WrappedMapper 类，所以我们再进入到 WrappedMapper 类中看一下：

```java
public class WrappedMapper<> extends Mapper<> {
  
    public Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>.Context getMapContext(MapContext<KEYIN, VALUEIN, KEYOUT, VALUEOUT> mapContext) {
      return new Context(mapContext);
    }

    @InterfaceStability.Evolving
    public class Context extends Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>.Context{

    protected MapContext<KEYIN, VALUEIN, KEYOUT, VALUEOUT> mapContext;

    public Context(MapContext<KEYIN, VALUEIN, KEYOUT, VALUEOUT> mapContext) {
      this.mapContext = mapContext;
    }

    /**
     * Get the input split for this map.
     */
    public InputSplit getInputSplit() {
      return mapContext.getInputSplit();
    }

    @Override
    public KEYIN getCurrentKey() throws IOException, InterruptedException {
      return mapContext.getCurrentKey();
    }

    @Override
    public VALUEIN getCurrentValue() throws IOException, InterruptedException {
      return mapContext.getCurrentValue();
    }

    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {
      return mapContext.nextKeyValue();
    }

    ...
}
```

从 WrappedMapper 中我们可以知道，run 方法中的 context.nextKeyValue()、context.getCurrentKey()、context.getCurrentValue() 方法都是在这里面调用的，但我们发现，这里面都是去调用了 mapContext 的同名方法。

即现在搜索顺序是 context->mapperContext->mapContext，所以接下来我们要去寻找 mapContext 对象的实现类，由 WrappedMapper 类可知，mapContext 的实现类是依靠 MapTask 中的 WrappedMapper.getMapContext(mapContext) 实现的，所以我们再回到 MapTask 中寻找 mapContext 的定义：

```java
org.apache.hadoop.mapreduce.MapContext<INKEY, INVALUE, OUTKEY, OUTVALUE> 
    mapContext = 
      new MapContextImpl<INKEY, INVALUE, OUTKEY, OUTVALUE>(job, getTaskID(), 
          input, output, 
          committer, 
          reporter, split);
```

这里我们可以看到 mapContext 的实现类是 MapContextImpl 的一个实例，所以我们再进入到 MapContextImpl 类中去找到 nextKeyValue()、getCurrentKey()、getCurrentValue() 方法：

```java
public class MapContextImpl<> extends TaskInputOutputContextImpl<> implements MapContext<> {
  private RecordReader<KEYIN,VALUEIN> reader;
  private InputSplit split;

  public MapContextImpl(Configuration conf, TaskAttemptID taskid,
                        RecordReader<KEYIN,VALUEIN> reader,
                        RecordWriter<KEYOUT,VALUEOUT> writer,
                        OutputCommitter committer,
                        StatusReporter reporter,
                        InputSplit split) {
    super(conf, taskid, writer, committer, reporter);
    this.reader = reader;
    this.split = split;
  }

  /**
   * Get the input split for this map.
   */
  public InputSplit getInputSplit() {
    return split;
  }

  @Override
  public KEYIN getCurrentKey() throws IOException, InterruptedException {
    return reader.getCurrentKey();
  }

  @Override
  public VALUEIN getCurrentValue() throws IOException, InterruptedException {
    return reader.getCurrentValue();
  }

  @Override
  public boolean nextKeyValue() throws IOException, InterruptedException {
    return reader.nextKeyValue();
  }

}
```

现在我们又可以知道 MapContextImpl 类中的 nextKeyValue()、getCurrentKey()、getCurrentValue() 方法是依靠 reader 调用同名方法去实现的，所以我们下一步进入 reader 的实现类中找到这三个方法，reader 的实现类又是依靠调用 MapContextImpl 的构造方法传入的（第三个参数），所以我们又要回到 MapTask 中找到 reader 的实现类：
```java
org.apache.hadoop.mapreduce.RecordReader<INKEY,INVALUE> input =
      new NewTrackingRecordReader<INKEY,INVALUE>
        (split, inputFormat, reporter, taskContext);

...

org.apache.hadoop.mapreduce.MapContext<INKEY, INVALUE, OUTKEY, OUTVALUE> 
    mapContext = 
      new MapContextImpl<INKEY, INVALUE, OUTKEY, OUTVALUE>(job, getTaskID(), 
          input, output, 
          committer, 
          reporter, split);
...


```

至此我们的搜索路径就为 context->mapperContext->mapContext->reader->input，现在我们再进入到 input 的实现类 NewTrackingRecordReader 中找一下上述三个方法（NewTrackingRecordReader 是 MapTask 中的内部类）：

```java
  static class NewTrackingRecordReader<K,V> 
    extends org.apache.hadoop.mapreduce.RecordReader<K,V> {
    private final org.apache.hadoop.mapreduce.RecordReader<K,V> real;
    private final org.apache.hadoop.mapreduce.Counter inputRecordCounter;
    private final org.apache.hadoop.mapreduce.Counter fileInputByteCounter;
    private final TaskReporter reporter;
    private final List<Statistics> fsStats;
    
    NewTrackingRecordReader(org.apache.hadoop.mapreduce.InputSplit split,
        org.apache.hadoop.mapreduce.InputFormat<K, V> inputFormat,
        TaskReporter reporter,
        org.apache.hadoop.mapreduce.TaskAttemptContext taskContext)
        throws InterruptedException, IOException {
      this.reporter = reporter;
      this.inputRecordCounter = reporter
          .getCounter(TaskCounter.MAP_INPUT_RECORDS);
      this.fileInputByteCounter = reporter
          .getCounter(FileInputFormatCounter.BYTES_READ);

      List <Statistics> matchedStats = null;
      if (split instanceof org.apache.hadoop.mapreduce.lib.input.FileSplit) {
        matchedStats = getFsStatistics(((org.apache.hadoop.mapreduce.lib.input.FileSplit) split)
            .getPath(), taskContext.getConfiguration());
      }
      fsStats = matchedStats;

      long bytesInPrev = getInputBytes(fsStats);
      //real 是通过 inputFormat 获取的
      this.real = inputFormat.createRecordReader(split, taskContext);
      long bytesInCurr = getInputBytes(fsStats);
      fileInputByteCounter.increment(bytesInCurr - bytesInPrev);
    }

    @Override
    public void close() throws IOException {
      long bytesInPrev = getInputBytes(fsStats);
      real.close();
      long bytesInCurr = getInputBytes(fsStats);
      fileInputByteCounter.increment(bytesInCurr - bytesInPrev);
    }

    @Override
    public K getCurrentKey() throws IOException, InterruptedException {
      return real.getCurrentKey();
    }

    @Override
    public V getCurrentValue() throws IOException, InterruptedException {
      return real.getCurrentValue();
    }

    @Override
    public float getProgress() throws IOException, InterruptedException {
      return real.getProgress();
    }

    @Override
    public void initialize(org.apache.hadoop.mapreduce.InputSplit split,
                           org.apache.hadoop.mapreduce.TaskAttemptContext context
                           ) throws IOException, InterruptedException {
      long bytesInPrev = getInputBytes(fsStats);
      real.initialize(split, context);
      long bytesInCurr = getInputBytes(fsStats);
      fileInputByteCounter.increment(bytesInCurr - bytesInPrev);
    }

    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {
      long bytesInPrev = getInputBytes(fsStats);
      boolean result = real.nextKeyValue();
      long bytesInCurr = getInputBytes(fsStats);
      if (result) {
        inputRecordCounter.increment(1);
      }
      fileInputByteCounter.increment(bytesInCurr - bytesInPrev);
      reporter.setProgress(getProgress());
      return result;
    }

    private long getInputBytes(List<Statistics> stats) {
      if (stats == null) return 0;
      long bytesRead = 0;
      for (Statistics stat: stats) {
        bytesRead = bytesRead + stat.getBytesRead();
      }
      return bytesRead;
    }
  }
```

然后我们叕叒双又发现，上面所述的三个方法是 real 调用的，real 是通过 inputFormat.createRecordReader(split, taskContext) 获取的，现在我们再进入到 inputFormat 中去找一下这三个方法（inputFormat 是 NewTrackingRecordReader 构造方法中的第二个参数）：

```java
...
// make the input format
org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE> inputFormat =
    (org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE>)
    ReflectionUtils.newInstance(taskContext.getInputFormatClass(), job);

...

org.apache.hadoop.mapreduce.RecordReader<INKEY,INVALUE> input =
      new NewTrackingRecordReader<INKEY,INVALUE>
        (split, inputFormat, reporter, taskContext);
...
```

最后我们发现，inputFormat 是利用反射机制创建的，然后我们进入到 taskContext.getInputFormatClass() 中看一下到底是反射了哪个类，进入到 taskContext.getInputFormatClass() 中后发现是一个未被实现的类，所以我们找一下他的实现类（JobContextImpl）中的getInputFormatClass()：

```java
@SuppressWarnings("unchecked")
public Class<? extends InputFormat<?,?>> getInputFormatClass() 
    throws ClassNotFoundException {
        //conf.getClass(属性,默认值);
return (Class<? extends InputFormat<?,?>>) 
    conf.getClass(INPUT_FORMAT_CLASS_ATTR, TextInputFormat.class);
}
```

所以我们可以看到这个方法返回的返回值是调用了 conf.getClass(INPUT_FORMAT_CLASS_ATTR, TextInputFormat.class); 方法，这里我们可以看到我们熟悉的 inputFormat 的默认实现类是 TextInputFormat，至此我们就知道 Hadoop 中的 inputFormat 的默认实现类是在哪定义的了，除此之外我们还发现有另一个参数 INPUT_FORMAT_CLASS_ATTR，我们可以点进这个参数中看一下：
```java
public static final String INPUT_FORMAT_CLASS_ATTR = "mapreduce.job.inputformat.class";
```

然而我们去配置文件中是找不到 mapreduce.job.inputformat.class 的属性的，所以 inputFormat 默认是实现 TextInputFormat 对象。

**我们再梳理一下流程顺序，run 方法中调用的三个方法（context.nextKeyValue()、context.getCurrentKey()、context.getCurrentValue()）分别是在：**

**context->mapperContext->mapContext->reader->input->real**

**real 是通过 inputFormat 的 createRecordReader(split, taskContext); 方法获取的实例对象。**

而 inputFormat 的默认实现类是 TextInputFormat，所以我们再去 TextInputFormat 中找一下 createRecordReader 方法：
```java
public class TextInputFormat extends FileInputFormat<LongWritable, Text> {

  //createRecordReader 方法返回的是 RecordReader 对象
  @Override
  public RecordReader<LongWritable, Text> 
    createRecordReader(InputSplit split,
                       TaskAttemptContext context) {
    String delimiter = context.getConfiguration().get(
        "textinputformat.record.delimiter");
    byte[] recordDelimiterBytes = null;
    if (null != delimiter)
      recordDelimiterBytes = delimiter.getBytes(Charsets.UTF_8);
    return new LineRecordReader(recordDelimiterBytes);
  }

  @Override
  protected boolean isSplitable(JobContext context, Path file) {
    final CompressionCodec codec =
      new CompressionCodecFactory(context.getConfiguration()).getCodec(file);
    if (null == codec) {
      return true;
    }
    return codec instanceof SplittableCompressionCodec;
  }

}
```

我们可以看到 TextInputFormat 的 createRecordReader 方法返回的是一个 RecordReader 类，但 RecordReader 是一个抽象类，所以 TextInputFormat 的 createRecordReader 方法返回的是一个实现类（LineRecordReader）。

至此，我们就明白了为什么文件加载 FileInputFormat 的默认实现类是 TextInputFormat，文件读取 RecordReader 默认实现的是 LineRecordReader 了。

那么 LineRecordReader 中肯定就实现了 nextKeyValue()、getCurrentKey()、getCurrentValue() 方法：
```java
public class LineRecordReader extends RecordReader<LongWritable, Text> {

  ...
  
  public LineRecordReader() {
  }

  public LineRecordReader(byte[] recordDelimiter) {
    this.recordDelimiterBytes = recordDelimiter;
  }

  public void initialize(InputSplit genericSplit,
                         TaskAttemptContext context) throws IOException {
    ...
  }
  
  ...

  // nextKeyValue 的实现方法
  public boolean nextKeyValue() throws IOException {
    //省略内部实现细节
    ...
  }

  //getCurrentKey() 的实现方法
  @Override
  public LongWritable getCurrentKey() {
    return key;
  }

  //getCurrentValue() 的实现方法
  @Override
  public Text getCurrentValue() {
    return value;
  }

  public float getProgress() throws IOException {
    ...
  }
  
  public synchronized void close() throws IOException {
    ...
  }
}
```

## MapReduce 中自定义文件输入

文件的输入是计算程序从文件系统将文件输入到 map 当中的运算过程，由上我们可以得知，map 的运行主要和 run 方法有关，run 方法中的逻辑关系主要跟其中的三个方法有关（nextKeyValue()、getCurrentKey()、getCurrentValue()）,并且这三个方法的实现本质上是与 FileInputFormat 和 RecordReader 有关，所以我们只要重新实现 FileInputFormat 和 RecordReader 抽象类当中的方法即可。

假设我们要实现一次输入一个文件到 Map 端处理，那我们就要重新实现自定义的 FileInputFormat 和 RecordReader，我们以一个「将多个文件合并成一个文件」的例子来说明如何重新实现 FileInputFormat 和 RecordReader：

MergeFileInputFormat 继承 FileInputFormat 并重写其方法：

```java
/**
 * K:Mapper输入的key  默认指的是偏移量
 * V:Mapper输入的value  默认指的是一行的内容
 *
 * 现在需求是每次读取一个文件的内容，所以现在我们假设Key是整个文件的内容，Value无所谓是什么
 * Key:整个文件的内容
 * Value:NullWritable
 */
public class MergeFileInputFormat extends FileInputFormat<Text, NullWritable> {

    // 构建 RecordReader 的方法

    /**
     *
     * @param split     文件切片    文件切片信息
     * @param context   上下文对象
     * @return
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public RecordReader<Text, NullWritable> createRecordReader(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        MergeFileRecordReader mergeFileRecordReader = new MergeFileRecordReader();
        mergeFileRecordReader.initialize(split, context);


        return mergeFileRecordReader;
    }
}

```

MergeFileRecordReader 继承 RecordReader 并重写其方法：
```java
public class MergeFileRecordReader extends RecordReader<Text, NullWritable> {

    FSDataInputStream open = null;
    //标识文件是否读取完成
    boolean isRead = false;
    Text key = new Text();
    long len = 0;

    // 主要进行文件读取  整个文件进行读取  创建一个流->创建流需要文件路径->可通过 split 切片进行获取
    // 这个方法是初始化方法   类似于 setup 方法    用于对属性或链接或流进行初始化
    @Override
    public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        FileSplit fileSplit = (FileSplit)split;
        Path path = fileSplit.getPath();
        //创建文件系统
        FileSystem fs = FileSystem.get(context.getConfiguration());
        //创建输入流
        open = fs.open(path);
        len = fileSplit.getLength();
    }

    //文件是否读取结束
    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {
        if(!isRead){
            byte[] b = new byte[(int)len];
            open.readFully(0,b);

            key.set(b);

            isRead = true;
            return isRead;
        }else {
            return false;
        }
    }

    @Override
    public Text getCurrentKey() throws IOException, InterruptedException {
        return this.key;
    }

    @Override
    public NullWritable getCurrentValue() throws IOException, InterruptedException {
        return NullWritable.get();
    }

    // 获取执行进度的方法
    @Override
    public float getProgress() throws IOException, InterruptedException {
        return isRead ? 1.0f : 0.0f;
    }

    //进行一些连接或者流的关闭
    @Override
    public void close() throws IOException {
        open.close();
    }
}

```

MergeFile -- Mapper、Reducer、Driver：

```java
public class MergeFile {
    static class MyMapper extends Mapper<Text, NullWritable, Text, NullWritable>{


        /**
         *
         * @param key 整个文件的内容
         * @param value NullWritable
         * @param context 上下文对象
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void map(Text key, NullWritable value, Context context) throws IOException, InterruptedException {
            context.write(key, NullWritable.get());
        }
    }

    static class MyReducer extends Reducer<Text, NullWritable, Text, NullWritable>{

        //每组相同的Key调用一次
        @Override
        protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
            context.write(key, NullWritable.get());
        }
    }

    public static void main(String[] args) throws InterruptedException, IOException, ClassNotFoundException {
        args = new String[]{"/mergein","/mergeout"};

        Configuration conf = new Configuration();

        //1 获取 Job 对象
        Job job = Job.getInstance(conf);

        //2 设置 jar 存储位置
        job.setJarByClass(MergeFile.class);

        //3 关联 Map 和 Reduce 类
        job.setMapperClass(MyMapper.class);
        job.setReducerClass(MyReducer.class);

        //4 设置 Mapper 阶段输出数据的 key 和 value 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        //5 设置最终数据输出的 key 和 value 类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        // 指定自定义输入
        job.setInputFormatClass(MergeFileInputFormat.class);

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

**总结：要自定义输入即需要自定义类继承 FileInputFormat 和 RecordReader 并实现其中的方法。**