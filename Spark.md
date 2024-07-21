# Spark

## 概念

### Spark 的技术栈有哪些？

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOiaURKKplpMp4EVzj107TSGeQwkEYPya7vicicJeFRxSAmAEUBXAziax2y2ABuibCq2HAef2EluzWxAOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. **Spark Core**：Spark 的基础组件，提供了任务调度、内存管理和错误恢复等功能。它还定义了 RDD（Resilient Distributed Datasets）数据结构，用于在集群上进行分布式计算。
2. **Spark SQL**：用于处理结构化数据的组件，支持使用 SQL 查询数据。它提供了 DataFrame 和 Dataset 两个 API，可以方便地进行数据处理和分析。适合处理大规模的结构化数据。
3. **Spark Streaming**：用于实时数据处理的组件，可以将实时数据流划分为小批次进行处理。它支持各种数据源，如 Kafka、Flume 和 HDFS，并提供了窗口操作和状态管理等功能。适合实时数据分析和流式处理。
4. **Spark MLlib**：用于机器学习的组件，提供了常见的机器学习算法和工具。它支持分类、回归、聚类和推荐等任务，并提供了特征提取、模型评估和模型调优等功能。适合大规模的机器学习任务。
5. **Spark GraphX**：用于图计算的组件，提供了图结构的抽象和常见的图算法。它支持图的构建、遍历和计算，并提供了图分析和图挖掘等功能。适合社交网络分析和图计算任务。

我们主要讨论的是 Spark Core 的部分。

### Spark 为什么快？

#### 为什么比MapReduce 快？

Spark的DAGScheduler相当于一个改进版的MapReduce，如果计算不涉及与其他节点进行数据交换，Spark可以在内存中一次性完成这些操作，也就是**中间结果**无须落盘，减少了磁盘IO的操作。但是，如果计算过程中涉及数据交换，Spark也是会把shuffle的数据写磁盘的！

有同学提到，Spark是基于内存的计算，所以快，这也不是主要原因，要对数据做计算，必然得加载到内存，Hadoop也是如此，只不过Spark支持将需要反复用到的数据给Cache到内存中，减少数据加载耗时，所以Spark跑机器学习算法比较在行（需要对数据进行反复迭代）。Spark基于磁盘的计算依然也是比Hadoop快。

#### Spark为什么比 Hive 快？

1. **消除了冗余的 HDFS 读写**: Hadoop 每次 shuffle 操作后，必须写到磁盘，而 Spark 在 shuffle 后不一定落盘，可以 cache 到内存中，以便迭代时使用。如果操作复杂，很多的 shufle 操作，那么 Hadoop 的读写 IO 时间会大大增加，也是 Hive 更慢的主要原因了
2. **消除了冗余的 MapReduce 阶段**: Hadoop 的 shuffle 操作一定连着完整的 MapReduce 操作，冗余繁琐。而 Spark 基于 RDD 提供了丰富的算子操作，且 reduce 操作产生 shuffle 数据，可以缓存在内存中
3. **JVM 的优化**: Hadoop 每次 MapReduce 操作，启动一个 Task 便会启动一次 JVM，基于进程的操作。而 Spark 每次 MapReduce 操作是基于线程的，只在启动 Executor 是启动一次 JVM，内存的 Task 操作是在线程复用的。每次启动 JVM 的时间可能就需要几秒甚至十几秒，那么当 Task 多了，这个时间 Hadoop 不知道比 Spark 慢了多少.
4. 但是Hive 2.X版本默认使用 MapReduce 作为查询引擎。比较新的 Hive 也是用 Tez, Spark 作为查询引擎，采用了DAG 的执行模型。

#### Flink与Spark 的技术选型？

1. **批处理任务**：如果主要任务是大规模的批处理，Spark依然是一个强大的选择，特别是其内存计算模型和Spark SQL的能力。
2. **实时流处理**：对于低延迟、高吞吐量的实时流处理任务，Flink通常是更好的选择。
3. **混合任务**：如果需要同时处理批和流数据，并且希望使用统一的API，Flink的批流统一模型可能更具优势。
4. **机器学习**：在机器学习领域，Spark MLlib仍然是一个强有力的工具，特别是在批处理和离线训练场景中。

### Spark 3.X 有什么新特性？

## RDD(Resilient Distributed Datasets,弹性分布式数据集)

### 为什么使用 RDD？

在RDD之前的计算框架，不善于处理2种应用：迭代式的算法（iterative algorithms）和交互式的数据挖掘工具（ineractive data mining tools）。

> 迭代式的算法: 下一阶段的计算会频繁使用到之前的数据， 例如PageRank，K-means集群算法，逻辑回归算法。
>
> 交互式的程序：程序运行过程中依赖用户的输入来确定下一步的计算。

RDD之前的计算框架，没有很好地复用计算阶段之间的数据。例如，MapReduce框架，计算完一个阶段的数据，将结果输出到外部的静态储存系统，例如，分布式文件系统，这造成数据复制，磁盘I/O，序列化的开销，并且占据了程序运行的大部分时间。而且，这些框架的容错，是通过不同机器之间同步数据/更新日志来实现的。对于数据密集型的应用，这些数据同步的操作使用更多的网络带宽，速度远慢于内存读取。

### RDD的特点

RDD是一个只读的，分片的记录集合。RDD只能通过(1)静态存储中的数据（2）其他RDD来创建。创建RDD的操作被称为 `Transformation`，例如 `map`, `filter`。

`org.apache.spark.rdd/RDD.Scala`中描述, RDD有以下特性：

* 一个partition列表，是数据集的一部分。
* 一个其他RDD的依赖列表。
* 一个函数，用来计算partition的分片。
* partitioner，返回RDD是否hash/范围分片的元数据（可选）
* 列出Partition p能快速访问的节点，基于数据局部性（可选）

### RDD的操作（算子）

RDD的操作分为 `Transformation`和 `Action`，`Transformation`是数据的转换，是延时执行的，只有遇到后面的 `Action`操作才会真正执行。

下面的代码中，`lines`创建了一个来自HDFS文件的RDD，errors是 `lines`创建的RDD。代码运行到到了 Collect( ) 才真正开始进行,  因为他是唯一的 Action 算子。

```
lines = spark.textFile("hdfs://...")
errors = lines.filter(_.startsWith("ERROR"))
errors.persist()
errors.count()


// Count errors mentioning MySQL:
errors.filter(_.contains("MySQL")).count()
// Return the time fields of errors mentioning
// HDFS as an array (assuming time is field
// number 3 in a tab-separated format):
errors.filter(_.contains("HDFS"))
      .map(_.split(’\t’)(3))
      .collect()
```

![rdd_example](https://liangyaopei.github.io/2021/01/16/apache-spark-rdd-intro/rdd_example.jpg)

### 算子的分类

#### 转换算子（transformations）

| **操作**                                                                               | **含义**                                                                                                                                                                                                               |
| -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **filter** ( *func* )                                                                | 筛选出满足条件的元素，并返回一个新的数据集                                                                                                                                                                                   |
| **map** ( *func* )                                                                   | 将每个元素传递到函数*func* 中，返回一个新的数据集，每个输入元素会映射到 **1 个输出结果**                                                                                                                             |
| **flatMap** ( *func* )                                                               | 与 map 相似，但每个输入元素都可以映射到**0 或多个输出结果**                                                                                                                                                            |
| **mapPartitions** ( *func* )                                                         | 与 map 相似，但是传递给函数*func* 的是每个分区数据集对应的迭代器                                                                                                                                                           |
| **distinct** ( *func* )                                                              | 对原数据集进行去重，并返回新的数据集                                                                                                                                                                                         |
| **groupByKey** ( *[numPartitions]* )                                                 | 应用于 (K, V) 形式的数据集，返回一个新的 (K, Iterable `<V>`) 形式的数据集，可通过 *numPartitions* 指定新数据集的分区数                                                                                                   |
| **reduceByKey** ( *func* ,  *[numPartitions]* )                                    | 应用于 (K, V) 形式的数据集，返回一个新的 (K, V) 形式的数据集，新数据集中的 V 是原有数据集中每个 K 对应的 V 传递到 *func *中进行聚合后的结果                                                                                  |
| **aggregateByKey** ( *zeroValue* )( *seqOp* ,  *combOp* ,  *[numPartitions]* ) | 应用于 (K, V) 形式的数据集，返回一个新的 (K, U) 形式的数据集，新数据集中的 U 是原有数据集中每个 K 对应的 V 传递到*seqOp（partition内的聚合）* 与 *combOp（Partition间的聚合）* 的联合函数且与 *zeroValue* 聚合后的结果 |
| **sortByKey** ( *[ascending]* ,  *[numPartitions]* )                               | 应用于 (K, V) 形式的数据集，返回一个根据 K 排序的数据集，K 按升序或降序排序由*ascending* 指定                                                                                                                              |
| **union** ( *func* )                                                                 | 将两个数据集中的元素合并到一个新的数据集                                                                                                                                                                                     |
| **join** ( *func* )                                                                  | 表示内连接，对于给定的两个形式分别为 (K, V) 和 (K, W) 的数据集，只有在两个数据集中都存在的 K 才会被输出，最终得到一个 (K, (V, W)) 类型的数据集                                                                               |
| **repartition** ( *numPartitions* )                                                  | 对数据集进行重分区，新的分区数由*numPartitions* 指定                                                                                                                                                                       |

**关于 `map` 和 `mapPartitions`：**

* 本质区别在于输入给函数 func 的对象不同，前者是 RDD 中的元素，后者是 RDD 分区数据集对应的迭代器。
* 假如某个 RDD 有 n 个分区，所有分区共有 m 个元素，那么该 RDD 调用 `map` 算子时会触发 m 次 func，而调用 `mapPartition` 时只会触发 n 次 func。在这种区别下，当函数 func 中存在创建连接、读取文件等耗费资源的步骤时，`mapPartition` 的性能会比 `map` 更好。

#### 行动算子（Action）

| **操作**                            | **含义**                                                                                                                    |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **count** ()                        | 返回数据集中的元素个数                                                                                                            |
| **countByKey** ()                   | 仅适用于 (K, V) 形式的数据集，以 (K, Int) 形式的 Map 返回每个 K 的元素个数                                                        |
| **collect** ()                      | 以数组的形式返回数据集中的所有元素                                                                                                |
| **first** ()                        | 返回数据集中的第一个元素                                                                                                          |
| **take** ( *n* )                  | 以数组的形式返回数据集中的前*n* 个元素                                                                                          |
| **reduce** ( *func* )             | 通过函数*func* （输入两个参数并返回一个值）聚合数据集中的元素                                                                   |
| **foreach** ( *func* )            | 将数据集中的每个元素传递到函数 *func *中运行                                                                                      |
| **saveAsTextFile** ( *path* )     | 将数据集以文本格式写到本地磁盘或 HDFS 的指定目录下                                                                                |
| **saveAsSequenceFile** ( *path* ) | 将数据集以 SequenceFile 格式写到本地磁盘或 HDFS 的指定目录下，仅适用于 (K, V) 形式且 K 和 V 均实现了 Hadoop Writable 接口的数据集 |
| **saveAsObjectFile** ( *path* )   | 将数据集序列化成对象保存至本地磁盘或 HDFS 的指定目录下                                                                            |

#### RDD 的依赖关系

在计算逻辑中，不可避免的会出现 RDD 转换的过程，即旧的 RDD 调用转换算子生成新的 RDD。

通常，我们称旧 RDD 为父 RDD，新 RDD 为子 RDD。在这个转换的过程里，新旧 RDD 自然会建立起类似父子的联系，这个联系从概念来说便是 RDD 的依赖关系，在代码层面由抽象类 **Dependency** 表示。

##### 宽窄依赖

*掌握宽窄依赖，是后续学习 RDD 阶段划分的基础，而 RDD 阶段划分，又是学习 Spark 任务划分的前提。这一系列知识，将有助于我们了解一个计算应用在提交后的执行过程。*

1. 宽依赖(ShuffleDependency)：多个子RDD的Partition依赖一个父RDD的Partition。例如一个 reduceBykey 就是一次造成宽依赖的shuffle计算：
2. 窄依赖(NarrowDependency)：每一个父RDD的一个Partition只被一个子RDD的Partition使用，或者多个父RDD指向一个子RDD分区。

![宽依赖示例](https://img-blog.csdnimg.cn/dea4c38ba23f45508d5aeaf412e9c35f.png)

下图中的蓝色方框表示的是 partition。

![rdd_dependencies](https://liangyaopei.github.io/2021/01/16/apache-spark-rdd-intro/rdd_dependencies.jpg)

窄依赖的RDD，支持同一个节点内的数据的流水线执行，不需要数据在节点之间的传输。与MapReduce比较，节省了数据传输的花销。而宽依赖的RDD，需要多个父RDD的数据，需要多个节点之间数据的shuffle，来传输数据。窄依赖的RDD，节点故障后的恢复更为高效，只需要重新计算丢失的RDD数据。而宽依赖的RDD，一个节点故障后，需要从宽依赖的继承图中的祖先节点，开始重新计算。

### RDD的血缘关系（Lineage）

血缘关系是 RDD 的重要特性之一，基于 RDD 核心属性 `dependencies` 实现，它描述了一个 RDD 是如何从初始 RDD 计算得来的。

![](https://magicpenta.github.io/assets/images/lineage-5e980c3ac6ec7d1daa2802cf7d23efd6.svg)



图例展现了一个 Spark 应用从输入到输出的过程。在该过程中，存在着一系列 RDD 的创建与转换，Spark 会记录转换过程中各个 RDD 的依赖关系，并在 RDD F 调用行动算子后构建 DAG 图，触发真正的计算。

将上述过程中 RDD 的依赖关系串联起来，便形成一个血缘关系（Lineage）。在血缘关系中，下一代的 RDD 依赖于上一代的 RDD。以图例说明，B 依赖于 A，D 依赖于 C，E 依赖于 B 和 D。

*血缘关系的存在使 RDD 具备了容错性。当 RDD 的部分分区数据丢失时，Spark 可以通过血缘关系获取足够的关联信息，进而重新计算并恢复丢失的分区。*

### RDD的持久化


### RDD 的检查点



## Spark如何运行/一个Spark job是怎么跑起来的/Spark 任务调度？

### Spark 有三大组件组成：

![img](https://pic2.zhimg.com/80/v2-2390f979cb9144e89ba1e95dddfa1f15_720w.webp)

1. Cluster Manager：

   1. 负责管理集群资源，包括节点的分配和监控。
   2. 它接受来自Driver的资源请求，并在集群中分配相应的Executor进程。
   3. Cluster Manager可以是Spark自带的Standalone模式，也可以是其他资源管理器，如YARN、Mesos或Kubernetes。
2. Driver（SparkContext）

   1. 作为Spark应用程序的驱动程序，负责提交Spark作业，并且是用户程序与Spark集群之间的桥梁。
   2. 它包含SparkContext对象，该对象初始化作业，并负责构建DAG（有向无环图）。
   3. Driver程序还负责划分DAG为多个Stage，并将任务分配给Executor执行。
3. Executor（进程）与task（线程）

   1. Executor是运行在集群节点上的进程，负责执行Driver分配的任务。
   2. 每个Executor包含多个Task线程，每个Task线程执行一个任务。
   3. Executor还负责管理其节点上的内存和存储资源，以及与外部存储系统（如HDFS）的交互。

### 运行过程如下：

![Spark job](https://pic4.zhimg.com/80/v2-9cfaf397c4cf2be0ea7909a90661971f_720w.webp)

1. **提交任务**：用户通过Client提交一个Spark任务。这通常涉及到编写一个Spark应用程序，并使用 `spark-submit`命令来提交。
2. **初始化SparkContext**：在Spark应用程序中，首先会初始化一个 `SparkContext`对象。`SparkContext`是Spark应用程序的入口点，负责与Cluster Manager通信。
3. **资源申请**：`SparkContext`向Cluster Manager注册并申请Executor资源。Cluster Manager负责资源的分配和管理。
4. **启动Executors**：Cluster Manager在各个工作节点上启动Executors。Executor是Spark应用程序在工作节点上的执行环境，负责执行任务。Executor 对 Driver 发送心跳。
5. **构建DAG**：Spark应用程序中的RDD操作会构建一个有向无环图（DAG），这个图表示了RDD之间的依赖关系。
6. **划分Stage**：DAG Scheduler负责将DAG划分为多个Stage。每个Stage包含一组可以并行执行的任务（Task）。
7. **任务分配**：Task Scheduler负责将任务分配给Executor执行。Executor会向SparkContext申请任务。
8. **执行任务**：Executor接收到任务后，会执行相应的代码。在执行过程中，可能会涉及到数据的读取、转换和写入。
9. **结果返回**：任务执行完成后，结果会返回给Driver。在某些情况下，结果可能会直接写入外部存储系统，如HDFS。
10. **动态分区**：如果使用了动态分区，Spark会在执行过程中动态创建分区，并在执行完成后将结果写入外部存储系统。
11. **关闭SparkContext**：在所有任务执行完毕后，`SparkContext`会被关闭，释放资源。

### 任务是如何被划分的？（[参考](https://blog.csdn.net/benjam1n77/article/details/126513983)）

#### Job,Stage,Task

![任务划分](https://pic2.zhimg.com/80/v2-dada5b5cb068774daf44dd2ac4e2ee15_720w.webp)

首先，Job=多个stage，Stage=多个同种task, Task分为ShuffleMapTask和ResultTask，Dependency分为宽依赖（ShuffleDependency）和窄依赖（NarrowDependency）。

* Job：Spark 中的算子分为 transformation 和 action，一个 action就会触发一个 Job。
* Stage: 一个Job会被划分为多个 Stage， Stage 以宽依赖为划分的依据。Shuffle前后的 RDD 属于不同的stage。
* Task：一个 Stage 包含一个或者多个 Task，一个stage的task数量由最后一个 RDD的 partition 数量决定。

**DAGScheduler，TaskScheduler, Schedulerbacked**

* DAGScheduler：根据RDD的依赖关系，将Job划分为一个或多个Stage，每个Stage会依据最后一个RDD的partition的数量生成一个或多个Task，同一Stage的Task属于同一TaskSet（任务集），DAGScheduler向TaskScheduler提交任务是以TaskSet为单位。
* TaskScheduler：接收来自DAGScheduler提交的TaskSet，向Executor分发Task。
* SchedulerBackend：TaskScheduler与Executor进行RPC通信的后台。

![img](https://img-blog.csdnimg.cn/41d9c067535b49fda075238bae3ef083.png#pic_center)

## 运行模式

（1）、Spark On Standalone模式为：TaskSchedule。

（2）、Yarn Client模式为：YarnClientClusterScheduler。

（3）、Yarn Cluster模式为：YarnClusterScheduler。

#### 几种部署方式：spark on yarn-client/ spark on yarn-cluster/spark on standalone

Yarn是什么？做什么的？

## RDD与算子

#### RDD是什么，有什么特性

RDD的宽窄依赖

任务切分-DAG图

 Spark RDD是怎么容错

Shuffle

 transform算子、action算子

#### 说说map和mapPartitions的区别

#### 说说RDD.cache()和RDD.persist()的区别:

## 性能调优

1. #### 内存溢出
2. 数据倾斜
3. 

## 名词术语

- Application：用户编写的spark的程序，其中包括一个Driver功能的代码块和分布在集群中多个节点上运行的Executor代码。
- Driver：运行上述的Application的main函数并创建SparkContext，目的是为了准备spark的运行环境，在spark中有SparkContext负责和ClusterManager通信，进行资源的申请、任务的分配和监控等，当Exectutor运行完毕的时候，负责把SparkContext关闭。
- Executor：某个Application运行在Worker节点的一个**进程**，该进程负责某些Task，并且负责将数据存到内存或磁盘上，每个Application都有各自独立的一批Executor，在Spark on yarn模式下，该进程被称为CoarseGrainedExecutor Backend。一个CoarseGrainedExecutor
  Backend有且仅有一个Executor对象，负责将Task包装成taskRuuner，并从线程池中抽取一个空闲线程运行Task，每一个CoarseGrainedExecutor Backend能够运行的Task数量取决于cpu数量。
- Cluster Manager：指的是在集群上获取资源的外部服务，目前有三种类型：
  - Sparkalone：spark的原生的资源管理，由Master负责资源的分配。
  - Apache Mesos：与Hadoop MR兼容性良好的一种资源调度框架。
  - Hadoop Yarn：只要指yarn中的ResourceManager
- Worker：集群中可以运行Application代码的节点，在sparkstandalone模式中是通过slave文件配置的worker节点，在Spark on yarn模式下就是NodeManager节点。
- Task：被送到某个Executor上的工作单元，和HadoopMR中的MapTask、ReduceTask概念一样，是运行Application的基本单位。多个Task组成一个Stage，而Task的调度和管理等是由TaskScheduler负责。
