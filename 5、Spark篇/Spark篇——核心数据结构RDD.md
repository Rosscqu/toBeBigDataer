## Spark篇——核心数据结构RDD

Spark目标是为了基于工作集的应用提供抽象，同时保持MapReduce及其模型的优势特性，即自动容错、位置感知性调度和可伸缩性。弹性分布式数据集（RDD）是Spark的数据结构的基本抽象，表示一个不可变的、可分区的并行操作的集合。本文主要介绍详细介绍RDD。



### 1、RDD基础

#### 1.1 RDD属性

1）**分区列表**

- 数据集的基本组成单位；
- 一个分区对应一个task线程；

2）**计算每个分区的算子**

- 该算子会作用于RDD的每个分区；

3）**RDD的依赖关系**

- 主要用于容错机制，记录血缘（后面会介绍血缘）

4）**KV类型的RDD具有散列函数**

系统默认的散列函数有两种：

- 基于哈希的HashPartitioner：分区逻辑是key.hashcode%分区总数=分区号
- 基于范围的RangePartitioner：

如果业务需要，也可以自定义散列函数。自定义散列函数的步骤：

- 继承Partitioner类；
- 实现numPartitions()方法；
- 实现getPartition()方法

例如自定义实现HashPartitioner散列函数：

```java
public class MyPartitioner extends Partitioner {
    
    private int partitions;

    public MyPartitioner(int partitions) {
        this.partitions = partitions;
    }

    @Override
    public int numPartitions() {
        return partitions;
    }

    @Override
    public int getPartition(Object key) {
        if (key == null) {
            return 0;
        } else {
            return key.hashCode() % numPartitions();
        }
    }
    
}
```

值得注意的是，对于KV的RDD，只有产生shuffle时，才会使用的partition。

5）存储存取每个Partition的优先位置

对于一个HDFS文件来说，这个列表保存的就是每个partition所在的块位置。



#### 1.2 创建RDD的方式 

创建RDD有3种方式：

- **通过并行集合**

  ```java
  public static void main(String[] args) {
      SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("create rdd");
      JavaSparkContext sc = new JavaSparkContext(conf);
      List<String> list = Arrays.asList("a", "b", "c");
      JavaRDD<String> listRdd = sc.parallelize(list);
      System.out.println(listRdd.take(1));
  
  }
  ```

- **从外部数据源获取**

  目前支持的数据源包括：HDFS、HBASE、ES、mysql、本地文件等；

  ```java
  JavaRDD<String> textRdd = sc.textFile("src/main/resources/kv1.txt");
  System.out.println(textRdd.take(2));
  ```

- **从已有的RDD通过转换获取**

```java
JavaRDD<String> textMapRdd = textRdd.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public Iterator<String> call(String s) throws Exception {
        String[] strings = s.split(",");
        List<String> stringList = Arrays.asList(strings);
        return stringList.iterator();
    }
});
System.out.println(textMapRdd.take(4));
```



### 2、RDD的转换和行动算子

算子主要分成两类：转换和行动，转换（transform）算子是根据已有的RDD转换生成一个新的RDD，转换操作是延迟加载不会立刻执行；动作(action)算子顾名思义是触发计算任务的执行，将RDD的计算结果返回Driver或者输出到外部存储介质。下面

#### 2.1 转换transform算子

转换算子包括：

- map
- flatmap
- filter
- mapPartitions
- mapPartitionsWithIndex
- union
- intersection
- distinct
- groupByKey
- reduceByKey：首先生成一个MapPartitionsRDD，起到map端combiner的作用；然后生成一个ShuffledRDD，从上一个RDD的输出读取数据，最为reducer端的开始；最后还会生成一个MapPartitionsRDD，起到reducer端reduce的作用。
- sortByKey
- sortBy
- join
- cogroup
- coalesce
- repartition
- repartitionAndSortWithinPartitions

#### 2.2 动作action算子

动作算子包括：

- reduce
- count
- collect
- first
- take
- takeSample
- saveAsTextFile
- saveAsSequenceFile
- saveAsObjectFile
- countByKey
- foreach
- foreachPartition



#### 2.3 RDD缓存机制

缓存的作用是当其他job需要使用该RDD的结果数据，可以直接从缓存中直接读取，避免重复计算。例如下图中，将RDD2进行缓存，然后RDD3和RDD4进行计算时可以直接从缓存中读取，可以提高计算效率。

<img src="img/RDD缓存示意图.png" alt="image-20201118002520354" style="zoom:50%;" />

1）RDD缓存机制实现方式：

- 方法persist()

  - 原理：调用persist()待参数的方法，默认存储级别是MEMORY_ONLY
  - 其他的缓存级别：

  ```scala
  val NONE = new StorageLevel(false, false, false, false)
  val DISK_ONLY = new StorageLevel(true, false, false, false)
  val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
  val MEMORY_ONLY = new StorageLevel(false, true, false, true)
  val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
  val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
  val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
  val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
  val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
  val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
  val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
  val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
  ```

  StorageLevel对象参数参数分别是：

  ```scala
  StorageLevel private(
      private var _useDisk: Boolean,
      private var _useMemory: Boolean,
      private var _useOffHeap: Boolean,
      private var _deserialized: Boolean,
      private var _replication: Int = 1)
  ```

- 方法cache()

  - 原理：调用persist()方法，固定的存储级别是MEMORY_ONLY。

2）可以创建缓存，也可以清除缓存，清除缓存的方法有：

- 自动清除：应用结束会自动清除内存的缓存；
- 手动清除：调用unpersist()方法

3）什么时候适合使用缓存？

- 某个RDD被多次使用；
- 为了获取一个RDD的结果，需要经过一系列复杂的算子操作或计算才能获取。

#### 2.4 RDD的checkpoint机制

通过缓存，spark避免了RDD上重新计算，能够极大地提升计算速度。但是如果缓存丢失，那么就需要重新计算，如果计算特别复杂或者计算特别耗时，那么会影响整个job的计算。为了避免由于RDD缓存丢失重新计算带来的开销，RDD引进了**检查点（checkpoint）机制**。

检查点是在计算后，重新建立一个job来计算。检查点的使用方式：

```java
public class CheckpointJob {

    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("checkpoint job").setMaster("local[2]");
        JavaSparkContext sc = new JavaSparkContext(conf);
        // 设置检查点目录
        sc.setCheckpointDir("file:///Users/chengxi/workspace/sourceWorkspace/spark-2.4.0/examples/src/main/resources/checkpoint");

        JavaRDD<String> dataSource = sc.textFile("/Users/chengxi/workspace/sourceWorkspace/spark-2.4.0/examples/src/main/resources/people.csv");
        // 在checkpoint前先缓存，可以提高效率
        dataSource.cache();
        dataSource.checkpoint();

        System.out.println(dataSource.count());

    }
}
```

检查点执行成功后，在检查点目录下会出现文件：

<img src="img/checkpoint文件示意图.png" alt="image-20201216223552208" style="zoom:50%;" />

在使用checkpoint时可以先使用cache()方法，可以直接从缓存持久化到checkpoint中。



#### 2.5 RDD的数据共享机制



#### 2.6 RDD序列化



### 3、RDD依赖关系

Spark会根据用户提交的计算逻辑中的RDD的转换和动作来生成RDD之间的依赖关系，依赖的维度包括：RDD的parent RDD是什么；依赖于parent RDD的哪些Partition。根据依赖的parent RDD的Partition的不同情况，将依赖关系分为：宽依赖和窄依赖。下面分别介绍这两种依赖关系。

#### 3.1 窄依赖

窄依赖是指每个Parent RDD的Partition最多被子RDD的一个Partition使用。例如：

<img src="img/窄依赖示意图.png" alt="image-20201216225253949" style="zoom:50%;" />

常见的窄依赖算子有：map、flatMap、filter等。

所有的类都要实现Dependency类，具体类图如下：

<img src="../../../Library/Application Support/typora-user-images/image-20201216231752375.png" alt="image-20201216231752375" style="zoom:50%;" />

其中NarrowDependency就是窄依赖。窄依赖又有两种实现：一对一依赖（OneToOneDependency）和范围的依赖(RangeDependency)。

1）一对一依赖

```java
class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd) {
  override def getParents(partitionId: Int): List[Int] = List(partitionId)
}
```

从源码中可以看出，RDD仅依赖于Parent RDD相同ID的Partition，即如上图所示。



2）范围的依赖

```java
/**
 * @param rdd the parent RDD
 * @param inStart parent RDD的起始点
 * @param outStart 子RDD的起始点
 * @param length parent RDD的Partition的数量
 */
class RangeDependency[T](rdd: RDD[T], inStart: Int, outStart: Int, length: Int)
  extends NarrowDependency[T](rdd) {

  override def getParents(partitionId: Int): List[Int] = {
    if (partitionId >= outStart && partitionId < outStart + length) {
      List(partitionId - outStart + inStart)
    } else {
      Nil
    }
  }
}
```

从源码中可以看出，范围的依赖是将多个RDD合并成一个RDD，这些RDD是被拼接起来的，每个parent RDD的Partition的相对顺序不会变。

目前该依赖仅仅被UnionRDD使用。



#### 3.2 宽依赖与shuffle依赖

宽依赖是指多个子RDD的Partition会依赖于同一个parent RDD的Partition，例如：

<img src="img/宽依赖示意图.png" alt="image-20201216230105646" style="zoom:50%;" />

宽依赖只有1种实现：ShuffleDependency：

```java
class ShuffleDependency[K: ClassTag, V: ClassTag, C: ClassTag](
    @transient private val _rdd: RDD[_ <: Product2[K, V]],
    val partitioner: Partitioner,
    val serializer: Serializer = SparkEnv.get.serializer,
    val keyOrdering: Option[Ordering[K]] = None,
    val aggregator: Option[Aggregator[K, V, C]] = None,
    val mapSideCombine: Boolean = false)
  extends Dependency[Product2[K, V]] {

  if (mapSideCombine) {
    require(aggregator.isDefined, "Map-side combine without Aggregator specified!")
  }
  override def rdd: RDD[Product2[K, V]] = _rdd.asInstanceOf[RDD[Product2[K, V]]]

  private[spark] val keyClassName: String = reflect.classTag[K].runtimeClass.getName
  private[spark] val valueClassName: String = reflect.classTag[V].runtimeClass.getName
  // Note: It's possible that the combiner class tag is null, if the combineByKey
  // methods in PairRDDFunctions are used instead of combineByKeyWithClassTag.
  private[spark] val combinerClassName: Option[String] =
    Option(reflect.classTag[C]).map(_.runtimeClass.getName)
  // 获取新的shuffleID
  val shuffleId: Int = _rdd.context.newShuffleId()

  // 向shuffleManager注册shuffle信息
  val shuffleHandle: ShuffleHandle = _rdd.context.env.shuffleManager.registerShuffle(
    shuffleId, _rdd.partitions.length, this)

  _rdd.sparkContext.cleaner.foreach(_.registerShuffleForCleanup(this))
}
```

从源码可以看出，子RDD依赖parent RDD的所有Partition，所以需要shuffle过程。宽依赖支持两种shuffle Manager：1）基于Hash的Shuffle机制的HashShuffleManager；2）基于排序的Shuffle机制的SortShuffleManager。具体的shuffle机制后面专门讲解。

### 4、DAG有向无环图

Spark生成RDD间的依赖关系后，同时这个计算链也就生成了逻辑上的DAG。

#### 4.1 Lineage血统

DAG之间的依赖关系包含RDD由哪些Parent RDD转换而来和它依赖Parent RDD的哪些Partition，这是DAG的重要属性。借助这些依赖关系，DAG可以认为这些RDD之间形成了Lineage（血统）。

Lineage在Spark中具有重要作用，不仅借能保证一个RDD计算前，它所依赖的parent RDD都已经完成计算；而且实现RDD的容错性，如果一个RDD的部分或全部计算结果丢失，那么只需要重新计算这部饿呜呜呜呜呜-=分丢失的数据，不需要全量的重新计算。



#### 4.2 DAG有向无环图生成



#### 4.3 stage划分







