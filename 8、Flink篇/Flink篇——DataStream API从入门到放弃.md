## Flink篇——DataStream API从入门到放弃



DataStream API主要分为三个部分：DataSource模块、Transformation模块以及DataSink模块：

- DataSource模块主要定义数据接入功能，主要将各种外部数据接入到Flink系统中，并将数据转换成对应的DataStream数据集；
- Transformation模块定义了对DataStream数据集的各种转换操作；
- DataSink模块将数据写出到外部存储介质中，例如将数据输出到文件或Kafka消息中间件中。

下面分别介绍Flink内置的API，以及如何自定义算子。



### 1、DataSource数据输入

Flink的数据源主要分为内置数据源和第三方数据源两种类型，内置数据源可以直接使用，而第三方数据源需要引入相应的maven依赖，目前第三方数据源有Apache Kafka Connector、ElasticSearch Connector等。当然也可以自定义实现数据接入函数SourceFunction，并封装成第三方数据源的Connector。

#### 1.1 内置数据源

内置数据源函数都定义在org.apache.flink.streaming.api.environment.StreamExecutionEnvironment类中，主要内置数据源如下表：

1）文件数据源

```java
DataStreamSource<String> readTextFile(String filePath)

DataStreamSource<String> readTextFile(String filePath, String charsetName)

DataStreamSource<OUT> readFile(FileInputFormat<OUT> inputFormat, String filePath)

DataStreamSource<OUT> readFile(FileInputFormat<OUT> inputFormat, String filePath,
												FileProcessingMode watchType, long interval)
								
DataStreamSource<OUT> readFile(FileInputFormat<OUT> inputFormat, String filePath,
												FileProcessingMode watchType, long interval, TypeInformation<OUT> typeInformation)
```

其中readTextFile直接读取文本文件，readFile方法通过指定文件FileInputFormat读取特定数据类型的文件，其中FileInputFormat可以是系统定义的FileInputFormat，也可以是自定义实现FileInputFormat接口类。

参数解释：

- filePath：文件路径；
- charsetName：编码方式，默认是"utf-8"；
- Inputformat：文件；
- watchType：文件读取类型，FileProcessingMode是枚举类，其中PROCESS_ONCE是指文件内容发生变化时，只会将变化的数据读取到Flink中；PROCESS_CONTINUOUSLY是指文件内容发生变化时，Flink会将文件全部内容加载到Flink中，所以不能保证Excatly-once语义。
- Interval：检测文件变化时间间隔；
- typeInformation：输入文件元素的类型信息



2）Socket数据源

```java
DataStreamSource<String> socketTextStream(String hostname, int port, String delimiter, long maxRetry)

DataStreamSource<String> socketTextStream(String hostname, int port, String delimiter)

DataStreamSource<String> socketTextStream(String hostname, int port)
```

后面两个重载方法都是调用第一个方法。

参数解释：

- hostname：IP地址
- port：端口
- delimiter：分割符号，默认"\n"
- maxRetry：重试次数

3）集合数据源

可以将集合类转换为DataStream数据集，本质是将本地集合中的数据分发到远端并行执行的节点中。目前支持继承Collection和Iterator类转换成DataStream。

```java
DataStreamSource<OUT> fromElements(OUT... data)

DataStreamSource<OUT> fromElements(Class<OUT> type, OUT... data)

DataStreamSource<OUT> fromCollection(Collection<OUT> data)

DataStreamSource<OUT> fromCollection(Collection<OUT> data, TypeInformation<OUT> typeInfo)

DataStreamSource<OUT> fromCollection(Iterator<OUT> data, Class<OUT> type)

DataStreamSource<OUT> fromCollection(Iterator<OUT> data, TypeInformation<OUT> typeInfo)

DataStreamSource<OUT> fromParallelCollection(SplittableIterator<OUT> iterator, Class<OUT> type)

DataStreamSource<OUT> fromParallelCollection(SplittableIterator<OUT> iterator, TypeInformation<OUT> typeInfo)
```



#### 1.2 外部数据源



### 2、DataStream转换操作





### 3、DataSink数据输出[=]