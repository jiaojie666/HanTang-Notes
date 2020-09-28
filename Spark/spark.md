<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

1. **Spark与Hadoop对比：**
Hadoop是有一些固有的缺陷的，例如表达能力有限，磁盘IO开销大，延迟很高，主要是任务之间衔接的时候要经过磁盘存储和读取，迭代式运算太过费时费力。相比较Hadoop,Spark很好的解决了这些问题，首先来说Spark的计算模式也属于mapReduce的范畴，但是并不局限于map和reduce两个操作，它提供多种数据操作类型，因此编程模型更加灵活。更重要的是Spark提供内存计算，将中间结果放到内存中，对于迭代运算效率极高。做到这一点的核心是，Spark提供了基于DAG图的任务调度执行机制。所以说Spark的计算性能能提升到hadoop的一百倍以上（例如逻辑回归时间）。
2. **RDD**
MapReduce中需要将中间结果写入磁盘或者hdfs会造成大量的IO开销。RDD提供一个抽象的数据架构，我们不必担心底层数据的分布式特性，只需要将具体的应用逻辑表达为一系列的转化处理，不同RDD之间的转化操作形成依赖关系，就可以实现管道化，避免中间数据的存储。RDD概念：一个RDD就是一个分布式对象集合，本质上是一个只读的分区记录集合，每个RDD包含多个分区，每个分区就是一个数据集片段，并且不同的分区可以保存到集群的不同结点上，从而可以在集群中的不同节点上进行并行计算。要注意的是RDD只能由读取数据源创建或者由其他RDD转换而来。RDD执行流程：RDD读取外部数据源创建，经过一系列transformation操作，每次产生一个转换过的RDD，最后一个RDD被action算子计算并输出到外部数据源。RDD的执行有一系列特点，例如惰性调用，管道化，避免同步等待，不需要保存中间结果，每次操作都很简单。RDD的特性：RDD具有很高效的容错性，他不需要进行数据复制或者记录日志，它只需要保存血缘关系和记录粗粒度的操作进行重新计算即可。
3. **Spark的shuffle操作：**
主要分为Shuffle Writer和Shuffle Read两部分。Shuffle Read端是经过很多技术的演进的，具体来说
-	hash shuffle v1:它假定大多数shuffle的数据不需要排序，例如wordcount，因此shuffle执行合并操作的时候会使用hashmap而不是sort&merge。具体来说在shuffle write的时候按照hash的方式重组patition数据，买个map task会为每一个reduce task生成一个文件，这可能会导致大量的磁盘io操作和大量的内存开销。
-	hash shuffle v2:它对上述情况作了优化，它将所有map task给同一个reducer生成的文件合并成一个，这样的话每个Executor就最多只会生成reducer个分区文件。
-	Sort Shuffle v1:在shuffle write的时候不会为后续的reducer单独的生成一个文件，而是将所有的结果写入到一个文件中。该文件中的记录首先按照partition id排序，patition内部再按照key排序。Shuffle write过程会顺序的写每个partition的数据，同时生成一个索引文件来记录partition的大小和偏移量。在shuffle read阶段，不再使用hashmap而是使用ExternalAppendOnlyMap,这种数据结构在做combine的时候如果内存不足，会刷写磁盘，很大程度上保证了鲁棒性。
-  Unsafe Shuffle: 它的做法是将数据记录用二进制的方式存储，直接在序列化的二进制数据上 Sort 而不是在 Java 对象上，这样一方面可以减少内存的使用和 GC 的开销，另一方面避免 Shuffle 过程中频繁的序列化以及反序列化。在排序过程中，它提供 cache-efficient sorter，使用一个 8 bytes 的指针，把排序转化成了一个指针数组的排序，极大的优化了排序性能
4. **Shuffle Read:**
当parent stage所有shuffle map tasks结束后再fetch。由于spark不要求Shuffler后的数据全局有序，因此它可以边fetch边处理。刚fetch来的数据会放在softbuffer缓冲区，经过处理后的数据会放在内存+磁盘上。不过这个地方可以比较灵活的设置，例如只用内存还是内存+磁盘。如果只用内存就使用appendOnlyMap，类似于hashmap。如果空间不足就使用ExternalAppendOnlyMap他可以将Records排序后在spill到磁盘上，等到需要的时候在进行归并。ExternalAppendOnlyMap 持有一个 AppendOnlyMap，shuffle 来的一个个 (K, V) record 先 insert 到 AppendOnlyMap 中，insert 过程与原始的 AppendOnlyMap 一模一样。如果 AppendOnlyMap 快被装满时检查一下内存剩余空间是否可以够扩展，够就直接在内存中扩展，不够就 sort 一下 AppendOnlyMap，将其内部所有 records 都 spill 到磁盘上。
5. **如何对RDD进行cache:**
那些经常被使用的RDD需要被cache，通过显式的调用rdd.cache来cache RDD,因此那些Spark自己生成的RDD是不能被用户cache到的，例如shuffledRDD。具体来说，当要计算RDD中某个partition的时候，会先去CacheManager里面领取一个blockID,表明是哪个RDD的哪个partition。然后去blockManager里面看该partition是否已经被checkPoint过，如果被chenkPoint了就只需要直接读取，然后把所有Records放到一个叫做elements的ArrayBuffer里面。如果没有被checkPoint首先要把partition计算出来。
6. **如何读取cache RDD:**
如果在下次计算中，一般是同一个application里面的下一个job计算时如果需要用到cache RDD,task会直接去blockManager的memoryStore中读取。具体来说，当要计算RDD某个partition的时候，会先去blockManager里面查找该partition是否已经被cache了，如果partition被cache在本地，就直接使用blockManager.getLocal()去本地memoryStore中读取，如果在其他节点上被cache了就通过调用blockManager.getRemote()方法去其他节点上读取。
7. **如何对RDD进行checkPoint：**
一般来说，那些运算时间很长或运算量很大才能得到的RDD需要被checkPoint。不同于cache是计算出一个要cache的partition就直接cache到内存，checkPoint是等到job结束后去另外启动专门的job来完成CheckPoint工作。也就是说需要checkPoint的RDD会被计算两次，因此最好先rdd.cache，这样避免重复计算。具体来说，RDD需要经过initialized->marked for checkpointeing->checkpointing in progress和checkPointed这几个阶段才能被checkPoint。
8. **Cache和checkPoint的区别**
他们最大的区别是cache将RDD存储到内存中，而checkPoint是将RDD存储到disk中，一般是HDFS.cache的RDD以及相对应的computing chain会被记住，所以如果出错的话可以通过重新计算来获得这部分数据。然而checkPoint不会存储Computing Chain，它通过HDFS的复制方法来实现fault tolerance。
RDD.Persist和checkPoint也是有区别的：前者虽然也是把RDD持久化到磁盘，但是对应partition会被blockManager管理。一旦driver Program结束，blockManager也会执行结束，也就是说对应persist的RDD也会被删除掉。但是CheckPoint则不会。
9. **Spark的部署模式及特点**
- 本地模式，spark应用会以多线程的方式直接运行在本地，一般都是为了方便调试，本地模式有三种种类，local代表只启动一个Executor，local[k]，代表启动k个executor，local[*]代表启动跟CPU数目相同的executor。
- standalone模式：分布式部署集群，自带完整的服务，资源管理和任务监控都是spark自己做，是基础模式
- Spark on Yarn：分布式部署集群，资源和任务监控交给yarn管理，但是目前仅支持粗粒度资源分配方式，包含cluster和client运行模式，其中cluster适合生产，driver运行在集群子节点上，具备容错功能，而client模式适合调试，driver运行在客户端
- Spark on Mesos:有两种运行模式，即粗粒度模式和细粒度模式，粗粒度模式由一个driver和若干个executor组成，其中每个Executor都要占用若干资源，内部可运行多个task。应用程序在正式运行之前，需要将运行环境中的资源全部申请好，并且运行期间一直占用资源，直至程序运行结束才会释放资源。细粒度模式则比较智能，能够按需分配。

10. **Driver的功能：**
Spark运行的时候包括一个driver进程也是作业的主进程，具备main函数，并且由sparkContext实例，是程序的入口点，它负责向集群申请资源，向master注册信息，负责作业调度，作业解析，生成Stage并调度task到Executor。
Worker的作用：管理当前节点内存，cpu的使用情况，接收master传来的资源指令，管理分配新进程，做计算的服务。
Spark工作机制，运行流程：
* 构建一个Application的运行环境，driver创建一个SparkContext
* SparkContext向资源管理器(Mesos,Yarn)申请Executor资源，资源管理器分配并启动Executor.
* Executor向SparkContext申请Task
* SparkContext将应用程序发送给Executor
* SparkContext构建DAG图，之后DAGScheduler将DAG图解析为多个Stage,每个stage包含多个task，将taskset发送给TaskScheduler，由taskSchedule将Task发送给Executor运行(5)executor运行Task,结束后释放资源。
11. **RDD的弹性与缺陷：**
可以设置为内存存储，或者灵活的内存+磁盘存储；基于lineage的高效容错机制；如果Task或者Stage失败会自动进行特定次数的重试操作；可以进行checkPoint和persist持久化操作；数据分片的高度弹性。缺陷是读操作可以是粗粒度细粒度都可，但写操作都是粗粒度的。即批量写入数据。
12. **Spark数据本地性的表现：**
ProcessLocal是指读取本地节点上的数据，NodeLocal指的是读取本地节点硬盘数据，Any则是指非本地节点数据。由于processLocal和Cache相关，因此经常用到的RDD应该Cache在内存中，于此同时由于cache是lazy的必须通过Action的触发。
13. **Spark on Yarn模式的优点：**
可以与mapreduce框架同时运行，如果不使用yarn进行资源分配，mapreduce分到的内存资源会很少，效率低下。资源按需分配，进而提高集群资源利用等。相比较Spark自带的standalone模式，yarn资源分配更加细致。Yarn以container作为资源隔离和分配的单位去使用内存CPU等。Yarn通过队列的方式，实现资源弹性管理。
14. **Parquent:**
采用列式存储，这使得查询的时候不需要扫描全部的数据，因为一般来说查询是针对某一个具体列的查询，而非整个表的查询，如果只需要scan某一列显然可以大大降低IO的消耗。由于每一列的类型都是同构的，因此可以针对不同的类型采用更加高效的数据压缩方式进一步减小IO。更加适用CPUpipline的编码方式，减小CPU缓存失效。实际存储的时候为了表明嵌套关系，存储repetition level,definition level和level。优点：速度更快，压缩速率更高，因此占空间更少，极大减少磁盘IO，更重要的是它能够有效减少stage的执行消耗。
15. **Partition和Block之间的关系**
HDFS中的文件会分block存储，block是分布式存储的最小单元，等分，可设置冗余，尽管这样设计会造成磁盘空间的浪费，但是整齐的block便于快速找到和读取对应的内容。Partition是弹性分布式数据集RDD的最小单元，RDD由分布在各个节点上的partiion构成，同一份RDD里面partition大小不一，数量不定，是根据算子和最初读入的数据分块数量定的。Block谓语存储空间,partition位于计算空间。Block大小固定，partition大小不固定。
16. **spark.storage.memoryFraction：**
spark.executor.memory代表每个Executor可用的内存，而Spark.storage.memoryFraction则决定了在这部分内存中有多少可以被用于memoryStore管理RDD cache数据，剩下的内存用来保证任务运行时其他内存空间的需要。其默认值是0.6.如果说代码运行过程中频繁的发生gc，那么就要适当调低这个数值。
17. **统一内存管理：**
是spark1.6以后引入的统一内存管理机制，与静态内存管理的区别在于存储内存和执行内存共享同一块空间，可以动态占用对方的空间区域。其中最重要的优化在于动态占用机制。首先根据参数spark.storage.storageFraction参数可以设定双方各自拥有的空间范围，当双方空间都不足的时候，就存储到硬盘，若己方空间不足而对方空余时，可以借用对方空间。执行内存的空间被对方占用后可以让对方写磁盘再归还借用的空间。存储空间的内存被占用后无法让执行空间归还，主要是实现比较复杂。
18. **Spark中哪些算子会导致shuffle产生：**
去重:distinct，聚合和排序:主要是by和bykey算子，重分区：repartition。集合运算例如join等
19. **为了application在没有获得足够资源的情况下job就开始执行了：**应该是task的调度线程和Executor资源申请是异步的导致的，这可能会导致执行该job时资源一直不足。为了避免这个问题可以适当调整  spark.scheduler.maxRegisteredResourcesWaitingTime和 spark.scheduler.minRegisteredResourcesRatio两个参数。
20. **Spark调优部分：**
参数调优：
+ -spark.shuffle.file.beffer默认值32kb，该参数值白用来设置shuffle writer task的bufferedOutputStream的buffer缓冲大小，在将数据spill到磁盘之前会先写入buffer缓冲之中，因此如果作业可用内存资源比较充足，可以适当调大这个参数，从而减少spill次数。
+ spark.reducer.maxSizeInFlight默认值48mb，它决定了shuffle read task的buffer缓冲大小，这个buffer缓冲决定了每次http拉取多少数据
+ spark.shuffle.io.maxRetries默认值3，它代表shuffle read如果因为网络失败，自动重试的次数。超过次数则会失败，适当调大提升稳定性
+ spark.shuffle.io.retryWait默认值5s，这个参数表明shuffle read拉取数据失败后，距离下一次去拉取数据的时间间隔
+ spark.shuffle.memoryFraction默认值0.2，这个参数代表Executor内存中，分配给shuffle read task进行局和操作的内存比例
+ spark.shuffle.manager：Hash和Sort方式，Sort是默认，Hash在reduce数量 比较少的时候，效率会很高。
+ spark.shuffle.sort. bypassMergeThreshold：设置的是Sort方式中，启用Hash输出方式的临界值，如果你的程序数据不需要排序，而且reduce数量比较少，那推荐可以适当增大临界值。
+ spark. shuffle.cosolidateFiles：如果你使用Hash shuffle方式，推荐打开该配置，实现更少的文件输出。
21. **数据倾斜：**
+ 什么是数据倾斜会导致什么后果：并行处理的数据集中某个partition的数据显著多于其他部分，这部分就成为性能瓶颈。它最容易导致的后果就是oom。
+ 如何定位：一般来说是因为shuffle导致的数据倾斜，一般来说可以通过查看任务，查看那Stage，查看代码来定位
+ 经常出现的情况以及对应的解决办法：
   +	数据分布不均匀，spark频繁交互：提前进行数据预处理，在进行kafka数据分发的时候尽可能平均分配，它从根源上解决数据倾斜，彻底避免执行shuffle类算子，那么肯定就不会有数据倾斜问题。
   +	数据集中不同的key由于分区方式，导致数据倾斜：增加shuffle read task数量，让原本分给一个Task的多个key分配给多个task,从而让每个task处理更少的数据。或者自己从新定义partitioner方法 
   +Join操作中一个数据集的数据分布不均匀，另一个数据集比较小：将reduce join转移为map join，当join操作其中的一个RDD或者表的数据量比较小的时候适用这个方案。通过广播小RDD全量数据+map算子来实现mapjoin避免Shuffle操作。
    +聚合操作中，数据集中的数据分布不均匀：将原本相同的key+随机前缀的方式变成不同的key，之后就可以把原本一个Task要做的任务分散给多个task去做局部聚合，最后去掉随机前缀进行一次全局聚合即可。
22. **Spark程序调优：**
避免创建重复的RDD，尽可能复用RDD，对经常使用的RDD要进行持久化，尽量少的使用Shuffle类算子，使用mapSide预聚合的shuffle操作。
使用高性能的算子：使用reduceByKey/aggregateByKey替代groupByKey:map-size;使用mapPartitions代替map，使用foreachPartitions代替foreach；使用filter+coalesce操作，对分区进行压缩；如果要在repartition之后还要进行排序直接使用repartitionAndSortWithinPartitions
23. **Spark资源调优：**
+ num-executors：应用运行时executor的数量，推荐50-100左右比较合适
+ executor-memory：应用运行时executor的内存，推荐4-8G比较合适
+ executor-cores：应用运行时executor的CPU核数，推荐2-4个比较合适
+ driver-memory：应用运行时driver的内存量，主要考虑如果使用map side join或者一些类似于collect的操作，那么要相应调大内存量
+ spark.default.parallelism：每个stage默认的task数量，推荐参数为num-executors * executor-cores的2~3倍较为合适
+ spark.storage.memoryFraction：每一个executor中用于RDD缓存的内存比例，如果程序中有大量的数据缓存，可以考虑调大整个的比例，默认为60%
+ spark.shuffle.memoryFraction：每一个executor中用于Shuffle操作的内存比例，默认是20%，如果程序中有大量的Shuffle类算子，那么可以考虑其它的比例

