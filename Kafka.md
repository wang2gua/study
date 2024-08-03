## Kafka

### 基本概念

#### 生产者消费者

众所周知，Kafka是一个消息队列，把消息放到队列里边的叫**生产者**，从队列里边消费的叫**消费者**。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSCcPLLczkhtSJOjsKdrYTdXGzrh4m09FtjaHNQsEV9vbe8rOKhQTSOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### Topic

一个消息中间件，队列不单单只有一个，我们往往会有多个队列，而我们生产者和消费者就得知道：把数据丢给哪个队列，从哪个队列消息。我们需要给队列取名字，叫做**topic**(相当于数据库里边**表**的概念)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSAQlIDjwPwS22av55eB8wtGoTS00WwAzrBHiaoK0f5o1mGib9EsnLK5IA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

现在我们给队列取了名字以后，生产者就知道往哪个队列丢数据了，消费者也知道往哪个队列拿数据了。我们可以有多个生产者**往同一个队列(topic)**丢数据，多个消费者**往同一个队列(topic)**拿数据

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSKC8E9qbOX0CbfKE2zib77wzOicT6GWZxv4nushlFQrFUbv98P68o4TEg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### Partition

为了提高一个队列(topic)的**吞吐量**，Kafka会把topic进行分区(**Partition**)

其中分区路由可以简单理解成一个 Hash 函数，生产者在发送消息时，完全可以自定义这个函数来决定分区规则。如果分区规则设定合理，所有消息将均匀地分配到不同的分区中。

先通过 Topic 对消息进行逻辑分类，然后通过 Partition 进一步做物理分片，最终多个 Partition 又会均匀地分布在集群中的每台机器上，从而很好地解决了存储的扩展性问题。

因此，Partition 是 Kafka 最基本的部署单元。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSAPbaicgRorFWGg4DQBTmFJwlzbIiczsVAYBdtjvqDXAL5LiawocvmI98g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Kafka分区

所以，生产者实际上是往一个topic名为Java3y中的分区(**Partition**)丢数据，消费者实际上是往一个topic名为Java3y的分区(**Partition**)取数据

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSoMDSug06DTcXR5vkBAZ0FKqg277rlw5sWRQqN6ejkceZhDHe3boJag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

##### 分片部署

同一个 Topic 的两个 Partition 分布在不同的消息服务器上，能做到消息的分布式存储了。提高可扩展性。

![图片](https://mmbiz.qpic.cn/mmbiz_png/AaabKZjib2kYoV8r1cz8iakcS18uiaPaicUZtcz05mlq35knW1zjuskluicluzH8JJPkgsxa1iaEQibbX16geiajTecUjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### Broker

一台Kafka服务器叫做**Broker**，Kafka集群就是多台Kafka服务器：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSiaWNEPEIq117QqjJJjROVZFFbkHchXgCuHxicVYKZrZcu8RzUPUSoWyA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

一个topic会分为多个partition，实际上partition会**分布**在不同的broker中，举个例子：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSzFg8c2RMeOSllhV91sIibY9V9YXhGOYVqETSn1csLElrZRULjmjNfRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

由此得知：**Kafka是天然分布式的**。

#### 备份

现在我们已经知道了往topic里边丢数据，实际上这些数据会分到不同的partition上，这些partition存在不同的broker上。分布式肯定会带来问题：“万一其中一台broker(Kafka服务器)出现网络抖动或者挂了，怎么办？”

Kafka是这样做的：我们数据存在不同的partition上，那kafka就把这些partition做**备份**。比如，现在我们有三个partition，分别存在三台broker上。每个partition都会备份，这些备份散落在**不同**的broker上。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSXaCIbAPDYwpwnGB8jmzzLianqd2ibdatNdcLxicnuvUtaTPUrKVRBxIWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

红色块的partition代表的是**主**分区，紫色的partition块代表的是**备份**分区。生产者往topic丢数据，是与**主**分区交互，消费者消费topic的数据，也是与主分区交互。

副本之间是 “一主多从” 的关系，其中 leader 副本负责读写请求，**备份分区仅仅用作于备份，不做读写。**如果某个Broker挂了，那就会选举出其他Broker的partition来作为主分区，这就实现了**高可用**。

另外值得一提的是：当生产者把数据丢进topic时，我们知道是写在partition上的，那partition是怎么将其持久化的呢？（不持久化如果Broker中途挂了，那肯定会丢数据嘛)。

Kafka是将partition的数据写在**磁盘**的(消息日志)，不过Kafka只允许**追加写入**(顺序访问)，避免缓慢的随机 I/O 操作。

- Kafka也不是partition一有数据就立马将数据写到磁盘上，它会先**缓存**一部分，等到足够多数据量或等待一定的时间再批量写入(flush)。

上面balabala地都是讲生产者把数据丢进topic是怎么样的，下面来讲讲消费者是怎么消费的。既然数据是保存在partition中的，那么**消费者实际上也是从partition中取**数据。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHS4WMrj7ibTDBIYgDs9txFNXl6Y030Fh9N7FibcpkT7tVr9mkRBP1ZSKpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 消费者组

生产者可以有多个，消费者也可以有多个。像上面图的情况，是一个消费者消费三个分区的数据。多个消费者可以组成一个**消费者组**。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSdCcibxaHvX94Qk3AjMEZj1aCzWsXf0d1PvGESMM7kkic8GyUzNibribeDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

本来是一个消费者消费三个分区的，现在我们有消费者组，就可以**每个消费者去消费一个分区**（也是为了提高吞吐量）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHSnQyheGN9ib499oicxic5mMebSFePJCIjy6CUxJGpH03blxuo3xRUQzByg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

按图上所示的情况，这里想要说明的是：

- 如果消费者组中的某个消费者挂了，那么其中一个消费者可能就要消费两个partition了
- 如果只有三个partition，而消费者组有4个消费者，那么一个消费者会空闲
- 每个 Partition 只能由消费组中的一个消费者进行消费。所以想提升的Kafka的吞吐量仅增加消费者数量是不行的，一定还得增加partition数量。
- 如果多加入一个**消费者组**，无论是新增的消费者组还是原本的消费者组，都能消费topic的全部数据。（消费者组之间从逻辑上它们是**独立**的）

#### 零拷贝

前面讲解到了生产者往topic里丢数据是存在partition上的，而partition持久化到磁盘是IO顺序访问的，并且是先写缓存，隔一段时间或者数据量足够大的时候才批量写入磁盘的。

消费者在读的时候也很有讲究：正常的读磁盘数据是需要将内核态数据拷贝到用户态的，而Kafka 通过调用`sendfile()`直接从内核空间（DMA的）到内核空间（Socket的），**少做了一步拷贝**的操作。（零拷贝）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/2BGWl1qPxib1fzJ5GDcNhdf30yoUqxGHS8NYxibXm9GUGORz886o1V3Kpiam5rH4icwNhqzgVQeXEkhAAG5Alrb2wg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### Offset

有的同学可能会产生疑问：消费者是怎么知道自己消费到哪里的呀？Kafka不是支持**回溯**吗？那是怎么做的呀？

- 比如上面也提到：如果一个消费者组中的某个消费者挂了，那挂掉的消费者所消费的分区可能就由存活的消费者消费。那**存活的消费者是需要知道挂掉的消费者消费到哪了**，不然怎么玩。

这里要引出`offset`了，Kafka就是用`offset`来表示消费者的消费进度到哪了，每个消费者会都有自己的`offset`。说白了`offset`就是表示消费者的**消费进度**。

在以前版本的Kafka，这个`offset`是由Zookeeper来管理的，后来Kafka开发者认为Zookeeper不合适大量的删改操作，于是把`offset`在broker以内部topic(`__consumer_offsets`)的方式来保存起来。

每次消费者消费的时候，都会提交这个`offset`，Kafka可以让你选择是自动提交还是手动提交。

既然提到了Zookeeper，那就多说一句。Zookeeper虽然在新版的Kafka中没有用作于保存客户端的`offset`，但是Zookeeper是Kafka一个重要的依赖。

- 探测broker和consumer的添加或移除。
- 负责维护所有partition的领导者/从属者关系（主分区和备份分区），如果主分区挂了，需要选举出备份分区作为主分区。
- 维护topic、partition等元配置信息

Kafka 通过巧妙的模型设计，将自己退化成一个海量消息的存储系统。

### 优化

#### 磁盘优化

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1ND6Kkicwib1YlXRcc5Sv36MrIFEfT9hRKKCq6SfooY4CibbMIic3IAUEaibIw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

完成一次磁盘 IO，需要经过`寻道`、`旋转`和`数据传输`三个步骤。

影响磁盘 IO 性能的因素也就发生在上面三个步骤上，因此主要花费的时间就是：

1. 寻道时间：Tseek 是指将读写磁头移动至正确的磁道上所需要的时间。寻道时间越短，I/O 操作越快，目前磁盘的平均寻道时间一般在 3-15ms。
2. 旋转延迟：Trotation 是指盘片旋转将请求数据所在的扇区移动到读写磁盘下方所需要的时间。旋转延迟取决于磁盘转速，通常用磁盘旋转一周所需时间的 1/2 表示。比如：7200rpm 的磁盘平均旋转延迟大约为 60*1000/7200/2 = 4.17ms，而转速为 15000rpm 的磁盘其平均旋转延迟为 2ms。
3. 数据传输时间：Ttransfer 是指完成传输所请求的数据所需要的时间，它取决于数据传输率，其值等于数据大小除以数据传输率。目前 IDE/ATA 能达到 133MB/s，SATA II 可达到 300MB/s 的接口数据传输率，数据传输时间通常远小于前两部分消耗时间。简单计算时可忽略。

因此，如果在写磁盘的时候省去`寻道`、`旋转`可以极大地提高磁盘读写的性能。

Kafka 采用`顺序写`文件的方式来提高磁盘写入性能。`顺序写`文件，基本减少了磁盘`寻道`和`旋转`的次数。磁头再也不用在磁道上乱舞了，而是一路向前飞速前行。

Kafka 中每个分区是一个有序的，不可变的消息序列，新的消息不断追加到 Partition 的末尾，在 Kafka 中 Partition 只是一个逻辑概念，Kafka 将 Partition 划分为多个 Segment，每个 Segment 对应一个物理文件，Kafka 对 segment 文件追加写，这就是顺序写文件。

#### 零拷贝优化

如图，如果采用传统的 IO 流程，先读取网络 IO，再写入磁盘 IO，实际需要将数据 Copy 四次。

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDicA87fyeP8F4ib4CGibiaAQzf0B7p4r9Aj5EgZNQDvaZd6HGbsKMDLWmIQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1)

1. 第一次：读取磁盘文件到操作系统内核缓冲区；
2. 第二次：将内核缓冲区的数据，copy 到应用程序的 buffer；
3. 第三步：将应用程序 buffer 中的数据，copy 到 socket 网络发送缓冲区；
4. 第四次：将 socket buffer 的数据，copy 到网卡，由网卡进行网络传输。

操作系统的设计就是每个应用程序都有自己的用户内存，用户内存和内核内存隔离，这是为了程序和系统安全考虑，否则的话每个应用程序内存满天飞，随意读写那还得了。

Kafka 使用到了 `mmap` 和 `sendfile` 的方式来实现`零拷贝`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDAQ0AhpNRSS1pk7iauicWHs9KXa2FbOHbYq4GfEPAYsUA0PjPAjROKLibw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1)

在此模型下，上下文切换的数量减少到一个。具体而言，`transferTo()`方法指示块设备通过 DMA 引擎将数据读取到读取缓冲区中。然后，将该缓冲区复制到另一个内核缓冲区以暂存到套接字。最后，套接字缓冲区通过 DMA 复制到 NIC 缓冲区。

减少了Copy次数以及上下文切换的次数。

#### PageCache

producer 生产消息到 Broker 时，Broker 会使用 pwrite() 系统调用按偏移量写入数据，此时数据都会先写入`page cache`。

consumer 消费消息时，Broker 使用 sendfile() 系统调用，零拷贝地将数据从 page cache 传输到 broker 的 Socket buffer，再通过网络传输。

leader 与 follower 之间的同步，与上面 consumer 消费数据的过程是同理的。

`page cache`中的数据会随着内核中 flusher 线程的调度以及对 sync()/fsync() 的调用写回到磁盘，就算进程崩溃，也不用担心数据丢失。另外，如果 consumer 要消费的消息不在`page cache`里，才会去磁盘读取，并且会顺便预读出一些相邻的块放入 page cache，以方便下一次读取。

因此如果 Kafka producer 的生产速率与 consumer 的消费速率相差不大，那么就能几乎只靠对 broker page cache 的读写完成整个生产 - 消费过程，磁盘访问非常少。

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDkRlAcohe4lGQwN9gibOrNypy0MveliasYRkvNOCm85KJRz7PhCMuRgQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 网络模型

Kafka 自己实现了网络模型做 RPC。底层基于 Java NIO，采用和 Netty 一样的 Reactor 线程模型。

在传统阻塞 IO 模型中，每个连接都需要独立线程处理，当并发数大时，创建线程数多，占用资源；采用阻塞 IO 模型，连接建立后，若当前线程没有数据可读，线程会阻塞在读操作上，造成资源浪费

针对传统阻塞 IO 模型的两个问题，Reactor 模型基于池化思想，避免为每个连接创建线程，连接完成后将业务处理交给线程池处理；基于 IO 复用模型，多个连接共用同一个阻塞对象，不用等待所有的连接。遍历到有新数据可以处理时，操作系统会通知程序，线程跳出阻塞状态，进行业务逻辑处理

其中包含了一个`Acceptor`线程，用于处理新的连接，`Acceptor` 有 N 个 `Processor` 线程 select 和 read socket 请求，N 个 `Handler` 线程处理请求并相应，即处理业务逻辑。



I/O 多路复用可以通过把多个 I/O 的阻塞复用到同一个 select 的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。它的最大优势是系统开销小，并且不需要创建新的进程或者线程，降低了系统的资源开销。

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDJXWciasSAhBfZ7QPV5u9j9VE1HUiblRs3fQwxz5WiaH2yFp9jxn1DBZqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 批量与压缩

Kafka Producer 向 Broker 发送消息不是一条消息一条消息的发送。使用过 Kafka 的同学应该知道，Producer 有两个重要的参数：`batch.size`和`linger.ms`。这两个参数就和 Producer 的批量发送有关。

Kafka Producer 的执行流程如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDuFK9fXH4ZnDLT6Ac4Gk1qUyJX9fhuG4wSYvVRvo93g8jc3CAZWOLSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

发送消息依次经过以下处理器：

- Serialize：键和值都根据传递的序列化器进行序列化。优秀的序列化方式可以提高网络传输的效率。
- Partition：决定将消息写入主题的哪个分区，默认情况下遵循 murmur2 算法。自定义分区程序也可以传递给生产者，以控制应将消息写入哪个分区。
- Compress：默认情况下，在 Kafka 生产者中不启用压缩.Compression 不仅可以更快地从生产者传输到代理，还可以在复制过程中进行更快的传输。压缩有助于提高吞吐量，降低延迟并提高磁盘利用率。
- Accumulate：`Accumulate`顾名思义，就是一个消息累计器。其内部为每个 Partition 维护一个`Deque`双端队列，队列保存将要发送的批次数据，`Accumulate`将数据累计到一定数量，或者在一定过期时间内，便将数据以批次的方式发送出去。记录被累积在主题每个分区的缓冲区中。根据生产者批次大小属性将记录分组。主题中的每个分区都有一个单独的累加器 / 缓冲区。
- Group Send：记录累积器中分区的批次按将它们发送到的代理分组。批处理中的记录基于 batch.size 和 linger.ms 属性发送到代理。记录由生产者根据两个条件发送。当达到定义的批次大小或达到定义的延迟时间时。

Producer、Broker 和 Consumer 使用相同的压缩算法，在 producer 向 Broker 写入数据，Consumer 向 Broker 读取数据时甚至可以不用解压缩，最终在 Consumer Poll 到消息时才解压，这样节省了大量的网络和磁盘开销。

#### 分区并发

Kafka 的 Topic 可以分成多个 Partition，每个 Paritition 类似于一个队列，保证数据有序。同一个 Group 下的不同 Consumer 并发消费 Paritition，分区实际上是调优 Kafka 并行度的最小单元，因此，可以说，每增加一个 Paritition 就增加了一个消费并发。

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDAuiblUIgWKDlg5q2h8nnxChibIL2eibI9jA4InPZtPnRCuj4wglGrKSNQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Kafka 具有优秀的分区分配算法——StickyAssignor，可以保证分区的分配尽量地均衡，且每一次重分配的结果尽量与上一次分配结果保持一致。这样，整个集群的分区尽量地均衡，各个 Broker 和 Consumer 的处理不至于出现太大的倾斜。

是不是分区越多越好？

- 越多的分区需要打开更多的文件句柄

  在 kafka 的 broker 中，每个分区都会对照着文件系统的一个目录。在 kafka 的数据日志文件目录中，每个日志数据段都会分配两个文件，一个索引文件和一个数据文件。因此，随着 partition 的增多，需要的文件句柄数急剧增加，必要时需要调整操作系统允许打开的文件句柄数。

- 客户端 / 服务器端需要使用的内存就越多

  客户端 producer 有个参数 batch.size，默认是 16KB。它会为每个分区缓存消息，一旦满了就打包将消息批量发出。看上去这是个能够提升性能的设计。不过很显然，因为这个参数是分区级别的，如果分区数越多，这部分缓存所需的内存占用也会更多。

- 降低高可用性

  分区越多，每个 Broker 上分配的分区也就越多，当一个发生 Broker 宕机，那么恢复时间将很长。

#### 文件结构

Kafka 消息是以 Topic 为单位进行归类，各个 Topic 之间是彼此独立的，互不影响。每个 Topic 又可以分为一个或多个分区。每个分区各自存在一个记录消息数据的日志文件。

Kafka 每个分区日志在物理上实际按大小被分成多个 Segment。

![图片](https://mmbiz.qpic.cn/mmbiz_png/FbXJ7UCc6O1hVYCbbib3UUk18ibs9EL1NDeyWiaHgQFOljZAAhfvnXOlSB3PHQvEjhUX1uHUjRhLKIFSVIBKGk8ww/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- segment file 组成：由 2 大部分组成，分别为 index file 和 data file，此 2 个文件一一对应，成对出现，后缀”.index”和“.log”分别表示为 segment 索引文件、数据文件。
- segment 文件命名规则：partion 全局的第一个 segment 从 0 开始，后续每个 segment 文件名为上一个 segment 文件最后一条消息的 offset 值。数值最大为 64 位 long 大小，19 位数字字符长度，没有数字用 0 填充。

index 采用稀疏索引，这样每个 index 文件大小有限，Kafka 采用`mmap`的方式，直接将 index 文件映射到内存，这样对 index 的操作就不需要操作磁盘 IO。`mmap`的 Java 实现对应 `MappedByteBuffer` 。

Kafka 充分利用二分法来查找对应 offset 的消息位置：

1. 按照二分法找到小于 offset 的 segment 的.log 和.index
2. 用目标 offset 减去文件名中的 offset 得到消息在这个 segment 中的偏移量。
3. 再次用二分法在 index 文件中找到对应的索引。
4. 到 log 文件中，顺序查找，直到找到 offset 对应的消息。

详细：[再过半小时，你就能明白kafka的工作原理了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/68052232)











[《浅入浅出》-Kafka (qq.com)](https://mp.weixin.qq.com/s/-IPfWPS1WQMEgcIu0Ak2VQ)

[Kafka 架构设计 (qq.com)](https://mp.weixin.qq.com/s/8wfZEsNDpeLr-_uu2CawFw)

[Kafka性能篇：为何Kafka这么"快"？ (qq.com)](https://mp.weixin.qq.com/s/kMIhPW2uLdy-mgS9sF6agw)