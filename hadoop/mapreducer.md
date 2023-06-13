Hadoop Apache 开源分布式平台

以hdfs、MapReduce为核心的Hadoop为用户提供了系统底层细节透明的分布式基础架构。

分布式文件系统 6.824



# mapreduce

**MapReduce is a programming model or pattern within the** [**Hadoop**](http://hadoop.apache.org/) **framework that is used to access big data stored in the Hadoop File System (HDFS).** It is a core component, integral to the functioning of the Hadoop framework.

MapReduce facilitates concurrent processing by splitting petabytes of data into smaller chunks, and processing them in parallel on Hadoop commodity servers. In the end, it aggregates all the data from multiple servers to return a consolidated output back to the application.

```
MapReduce 是Hadoop框架内的一种编程模型或模式，用于访问存储在 Hadoop 文件系统 (HDFS) 中的大数据。它是一个核心组件，是 Hadoop 框架功能的组成部分。

MapReduce 通过将数 PB 的数据拆分为更小的数据块并在 Hadoop 商用服务器上并行处理它们来促进并发处理。最后，它聚合来自多个服务器的所有数据，将合并后的输出返回给应用程序。
```



```
使用 MapReduce，不是将数据发送到应用程序或逻辑所在的位置，而是在数据已经驻留的服务器上执行逻辑，以加快处理速度。数据访问和存储是基于磁盘的——输入通常存储为包含结构化、半结构化或非结构化数据的文件，输出也存储在文件中。

MapReduce 曾经是唯一可以检索存储在 HDFS 中的数据的方法，但现在已不再如此。今天，还有其他基于查询的系统，例如 Hive 和 Pig，它们用于使用类似 SQL 的语句从 HDFS 检索数据。但是，这些通常与使用 MapReduce 模型编写的作业一起运行。那是因为 MapReduce 具有独特的优势。
```



## mapreduce如何工作

### Map函数

将来自磁盘的输入作为键值对处理，并声称另一组中间键值对作为输出。 map 是过滤和排序初始数据的强制性步骤

The input data is first split into smaller blocks. Each block is then assigned to a mapper for processing.

For example, if a file has 100 records to be processed, 100 mappers can run together to process one record each. Or maybe 50 mappers can run together to process two records each. The Hadoop framework decides how many mappers to use, based on the size of the data to be processed and the memory block available on each mapper server.

```
输入数据被分割成更小的块。将每个块分配给映射器进行处理
例如，如果一个文件由100个记录需要处理，则100个mapper一起处理，每个处理一条数据。或者也许 50 个映射器可以一起运行，每个映射器处理两个记录。Hadoop framework决定使用多少映射器，根据被处理数据集的大小和每个mapper server可用的内存块
```



### Reduce

After all the mappers complete processing, the framework shuffles and sorts the results before passing them on to the reducers. A reducer cannot start while a mapper is still in progress. All the map output values that have the same key are assigned to a single reducer, which then aggregates the values for that key.

```
在所有映射器完成处理后，框架在将结果传递给缩减器之前对结果进行混洗和排序。当 mapper 仍在进行中时，reducer 无法启动。具有相同键的所有map输出值都分配给一个reducer，然后该reducer聚合该键的值。
```



### Combine and Partition

There are two intermediate steps between Map and Reduce.

```
map与reducer的两个中间步骤
```

**Combine** is an optional process. The combiner is a reducer that runs individually on each mapper server. It reduces the data on each mapper further to a simplified form before passing it downstream.

```
combiner 是一个 reducer 在每个 map server 上单独运行。在将数据传递给下游之前，它将每个mapper上的数据进一步简化
```

This makes shuffling and sorting easier as there is less data to work with. Often, the combiner class is set to the reducer class itself, due to the cumulative and associative functions in the reduce function. However, if needed, the combiner can be a separate class as well.

**Partition** is the process that translates the <key, value> pairs resulting from mappers to another set of <key, value> pairs to feed into the reducer. It decides how the data has to be presented to the reducer and also assigns it to a particular reducer.

The default partitioner determines the hash value for the key, resulting from the mapper, and assigns a partition based on this hash value. There are as many partitions as there are reducers. So, once the partitioning is complete, the data from each partition is sent to a specific reducer.



## A MapReduce Example



### Map

For simplification, let's assume that the Hadoop framework runs just four mappers. Mapper 1, Mapper 2, Mapper 3, and Mapper 4.

The value input to the mapper is one record of the log file. The key could be a text string such as "file name + line number." The mapper, then, processes each record of the log file to produce key value pairs. Here, we will just use a filler for the value as '1.' The output from the mappers look like this:

```
mapper处理日志文件的每个记录以生成键值对。
```

Mapper 1 -> <Exception A, 1>, <Exception B, 1>, <Exception A, 1>, <Exception C, 1>, <Exception A, 1>
Mapper 2 -> <Exception B, 1>, <Exception B, 1>, <Exception A, 1>, <Exception A, 1>
Mapper 3 -> <Exception A, 1>, <Exception C, 1>, <Exception A, 1>, <Exception B, 1>, <Exception A, 1>
Mapper 4 -> <Exception B, 1>, <Exception C, 1>, <Exception C, 1>, <Exception A, 1>

Assuming that there is a combiner running on each mapper—Combiner 1 … Combiner 4—that calculates the count of each exception (which is the same function as the reducer), the input to Combiner 1 will be:

```
假设在每个 mapper 上运行一个组合器——Combiner 1 … Combiner 4——计算每个异常的计数（这与 reducer 的功能相同），Combiner 1 的输入将是：
```

<Exception A, 1>, <Exception B, 1>, <Exception A, 1>, <Exception C, 1>, <Exception A, 1>



### Combine

The output of Combiner 1 will be:

<Exception A, 3>, <Exception B, 1>, <Exception C, 1>

The output from the other combiners will be:

Combiner 2: <Exception A, 2> <Exception B, 2>
Combiner 3: <Exception A, 3> <Exception B, 1> <Exception C, 1>
Combiner 4: <Exception A, 1> <Exception B, 1> <Exception C, 2>

### Partition

After this, the partitioner allocates the data from the combiners to the reducers. The data is also sorted for the reducer.

The input to the reducers will be as below:

Reducer 1: <Exception A> {3,2,3,1}
Reducer 2: <Exception B> {1,2,1,1}
Reducer 3: <Exception C> {1,1,2}

If there were no combiners involved, the input to the reducers will be as below:

Reducer 1: <Exception A> {1,1,1,1,1,1,1,1,1}
Reducer 2: <Exception B> {1,1,1,1,1}
Reducer 3: <Exception C> {1,1,1,1}

Here, the example is a simple one, but when there are terabytes of data involved, the combiner process’ improvement to the bandwidth is significant.

### Reduce

Now, each reducer just calculates the total count of the exceptions as:

Reducer 1: <Exception A, 9>
Reducer 2: <Exception B, 5>
Reducer 3: <Exception C, 4>

The data shows that Exception A is thrown more often than others and requires more attention. When there are more than a few weeks' or months' of data to be processed together, the potential of the MapReduce program can be truly exploited.



## How to Implement MapReduce

MapReduce programs are not just restricted to Java. They can also be written in C, C++, Python, Ruby, Perl, etc. Here is what the main function of a typical MapReduce job looks like:

```java
public static void main(String[] args) throws Exception {
	
    // 创建一个新的JobConf对象，用于配置MapReduce作业。
    JobConf conf = new JobConf(ExceptionCount.class);
    // 设置作业名称为“exceptioncount”。
    conf.setJobName("exceptioncount");

    // 设置输出键类型为Text类。
    conf.setOutputKeyClass(Text.class);
    // 设置输出值类型为IntWritable类。
    conf.setOutputValueClass(IntWritable.class);

    //  设置Mapper类为Map。
    conf.setMapperClass(Map.class);
    // 设置Reducer类为Reduce。
    conf.setReducerClass(Reduce.class);
    // 设置Combiner类为Reduce类（这里指的是Reducer类）。
    conf.setCombinerClass(Reduce.class);

    // 设置输入格式为TextInputFormat类。
    conf.setInputFormat(TextInputFormat.class);
    // 设置输出格式为TextOutputFormat类。
    conf.setOutputFormat(TextOutputFormat.class);

    // 设置输入路径为第一个命令行参数。
    FileInputFormat.setInputPaths(conf, new Path(args[0]));
    // 设置输出路径为第二个命令行参数。
    FileOutputFormat.setOutputPath(conf, new Path(args[1]));

    // 运行MapReduce作业。
    JobClient.runJob(conf);

}
```

The parameters—MapReduce class name, Map, Reduce and Combiner classes, input and output types, input and output file paths—are all defined in the main function. The Mapper class extends MapReduceBase and implements the Mapper interface. The Reducer class extends MapReduceBase and implements the Reducer interface.

To get on with a detailed code example, check out these [Hadoop tutorials](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html).





# Hadoop

The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

































