# Spark

## 前言

Spark 的技术栈有哪些？

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOiaURKKplpMp4EVzj107TSGeQwkEYPya7vicicJeFRxSAmAEUBXAziax2y2ABuibCq2HAef2EluzWxAOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. **Spark Core**：Spark 的基础组件，提供了任务调度、内存管理和错误恢复等功能。它还定义了 RDD（Resilient Distributed Datasets）数据结构，用于在集群上进行分布式计算。
2. **Spark SQL**：用于处理结构化数据的组件，支持使用 SQL 查询数据。它提供了 DataFrame 和 Dataset 两个 API，可以方便地进行数据处理和分析。适合处理大规模的结构化数据。
3. **Spark Streaming**：用于实时数据处理的组件，可以将实时数据流划分为小批次进行处理。它支持各种数据源，如 Kafka、Flume 和 HDFS，并提供了窗口操作和状态管理等功能。适合实时数据分析和流式处理。
4. **Spark MLlib**：用于机器学习的组件，提供了常见的机器学习算法和工具。它支持分类、回归、聚类和推荐等任务，并提供了特征提取、模型评估和模型调优等功能。适合大规模的机器学习任务。
5. **Spark GraphX**：用于图计算的组件，提供了图结构的抽象和常见的图算法。它支持图的构建、遍历和计算，并提供了图分析和图挖掘等功能。适合社交网络分析和图计算任务。

我们主要讨论的是 Spark Core 的部分。



Spark 为什么快？

为什么不用 Flink？

Spark 3.X 有什么新特性？







## Spark如何运行（一个Spark job是怎么跑起来的）？

![1721142303398](C:\Users\wang2\AppData\Roaming\Typora\typora-user-images\1721142303398.png)

**Spark 有三大组件组成：**

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

**运行过程如下：**

1. **提交任务**：用户通过Client提交一个Spark任务。这通常涉及到编写一个Spark应用程序，并使用`spark-submit`命令来提交。
2. **初始化SparkContext**：在Spark应用程序中，首先会初始化一个`SparkContext`对象。`SparkContext`是Spark应用程序的入口点，负责与Cluster Manager通信。
3. **资源申请**：`SparkContext`向Cluster Manager注册并申请Executor资源。Cluster Manager负责资源的分配和管理。
4. **启动Executors**：Cluster Manager在各个工作节点上启动Executors。Executor是Spark应用程序在工作节点上的执行环境，负责执行任务。Executor 对 Driver 发送心跳。
5. **构建DAG**：Spark应用程序中的RDD操作会构建一个有向无环图（DAG），这个图表示了RDD之间的依赖关系。
6. **划分Stage**：DAG Scheduler负责将DAG划分为多个Stage。每个Stage包含一组可以并行执行的任务（Task）。
7. **任务分配**：Task Scheduler负责将任务分配给Executor执行。Executor会向SparkContext申请任务。
8. **执行任务**：Executor接收到任务后，会执行相应的代码。在执行过程中，可能会涉及到数据的读取、转换和写入。
9. **结果返回**：任务执行完成后，结果会返回给Driver。在某些情况下，结果可能会直接写入外部存储系统，如HDFS。
10. **动态分区**：如果使用了动态分区，Spark会在执行过程中动态创建分区，并在执行完成后将结果写入外部存储系统。
11. **关闭SparkContext**：在所有任务执行完毕后，`SparkContext`会被关闭，释放资源。

**任务是如何被划分的？**





## 运行模式

（1）、Spark On Standalone模式为：TaskSchedule。

（2）、Yarn Client模式为：YarnClientClusterScheduler。

（3）、Yarn Cluster模式为：YarnClusterScheduler。

#### 几种部署方式：spark on yarn-client/ spark on yarn-cluster/spark on standalone

Yarn是什么？做什么的？



## 





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



- 1.Application：用户编写的spark的程序，其中包括一个Driver功能的代码块和分布在集群中多个节点上运行的Executor代码。
- Driver：运行上述的Application的main函数并创建SparkContext，目的是为了准备spark的运行环境，在spark中有SparkContext负责和ClusterManager通信，进行资源的申请、任务的分配和监控等，当Exectutor运行完毕的时候，负责把SparkContext关闭。
- Executor：某个Application运行在Worker节点的一个**进程**，该进程负责某些Task，并且负责将数据存到内存或磁盘上，每个Application都有各自独立的一批Executor，在Spark on yarn模式下，该进程被称为CoarseGrainedExecutor Backend。一个CoarseGrainedExecutor
  Backend有且仅有一个Executor对象，负责将Task包装成taskRuuner，并从线程池中抽取一个空闲线程运行Task，每一个CoarseGrainedExecutor Backend能够运行的Task数量取决于cpu数量。
- Cluster Manager：指的是在集群上获取资源的外部服务，目前有三种类型：
  - Sparkalone：spark的原生的资源管理，由Master负责资源的分配。
  - Apache Mesos：与Hadoop MR兼容性良好的一种资源调度框架。
  - Hadoop Yarn：只要指yarn中的ResourceManager
- Worker：集群中可以运行Application代码的节点，在sparkstandalone模式中是通过slave文件配置的worker节点，在Spark on yarn模式下就是NodeManager节点。
- Task：被送到某个Executor上的工作单元，和HadoopMR中的MapTask、ReduceTask概念一样，是运行Application的基本单位。多个Task组成一个Stage，而Task的调度和管理等是由TaskScheduler负责。