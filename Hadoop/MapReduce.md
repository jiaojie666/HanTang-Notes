<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

### MapReduce作业运行机制 ###
  主要涉及五个实体（提交mapreduce作业的客户端，YARN资源管理器，YARN结点管理器，Application Master，HDFS）。   
  - 作业的提交：Job类对象的submit方法内部会创建一个jobSubmiter对象并调用submitJobInternal方法，提交作业后通过waitForComplete轮询作业进度，之后向RM请求一个新的应用ID，作为MR的作业ID，将运行作业的容器环境复制到一个以作业ID命名的目录下的共享文件系统中，最后RM提交作业  
  - 作业初始化：RM中的调度器会选择一个NM并在这个NM中启动容器并开启Application Mastert进程，之后MRAppMaster初始化job，MRAppMaster接受来自共享文件系统的输入分片之后创建mapTask对象  
  - 任务分配：MRAppMaster中的所有task向RM申请容器，一个Task对应一个容器。  
  - 任务的执行：Application Master启动运行任务的容器，Application Master的重要任务之一就是监控Task进度  
  - 作业完成  
### MapReduce数据流 ###
   文件>InputFormat>InputSplit>RecordReader>Mapper>Shuffle>Reducer>OutPutFormat>。  
   输入文件可以是HDFS中任意类型的文件，例如SequenceFile, Line-based File(是否以tab分割)。InputFormat会读取文件中的内容并将文件内容划分为InputSplit，每个InputSplit对应一个Mapper。InputSplit之后对应的RecordReader，他会将InputSplit中的数据转化为kv对。之后这些kv pair会被发送给Mapper进一步处理并生成intermediate kv pair，一个重要的区别是intermediate kv pair的value可能是一个list
   + **InputFormat阶段** 描述了如何读入输入文件和如何对文件进行分片。Mapper获取数据的函数有getsplits()和createRecordReader()。为了处理HDFS中多种类型的文件，有多种类型的inputformat可以被调用：               
    - FileInputFormat会指定一个包含多个文件的输入文件夹，等mapreduce job开始执行的时候，FileInputFormat就会根据这个路径去读取文件夹下所有的文件。  
    - TextInputFormat:按行读取文件中的内容，这是默认的方法，它会将数据读为kv pair，k是行首的字节偏置量，v是具体的line content。  
    - keyValueTextInputFormat也是按行读取文件但是它每行的数据会以tab分割，tab前为key,后为value。  
    - SequenceFileInputFormat包含序列化和反序列化方法可以读取二进制的kv pair。与之相似的还有SequenceFileAsTextInputFormat，它包含tostring方法能将keyvalue映射为Text内容。还有SequenceFileAsBinaryInputFormat方法。最后还有NLineInputFormat和DBInputFormat可以从关系数据库。
   +  **InputSplit：** 是mapreduce中数据的逻辑表示形式，每个inputsplit会对应一个mapper，由于mapreduce具有data locility优化方法，因此task tracker常常将mapper就近放在所用数据周围。这个会有一个贪心的策略，就是尽可能优先处理更大的inputSplit，即保持mapper和data在同一个Datanode上,以获得更高的性能。要注意的是inputsplit并不是真的保存了数据，他只是数据的引用。通常来说InputFormat会根据HDFS分块的大小来将inputFile分割为input split。
   + **RecordReader：** 会将inputSplit中的数据convert成便于mapper处理的k,v结构。关键函数有setup(contex),context.nextKeyValue()等，常用的RecordReader有LineRecordReader和SequenceFileRecordread。
   + **Partitioner:** partitions的数量取决于Reducer的数量。哪个k,v pair进哪一个patition取决于key的hashCode。
   + **Shuffle阶段：** Shuffle是一个神奇的阶段，我认为他一部分属于Map端一部分属于Reduce端。Map端：在Map函数开始产生输出的时候并不是简单的将intermediate kv直接写入磁盘中，它会利用环形缓冲区的方式先写入到内存中进行分区和预排序。具体来说，每个Mapper都有一个属于自己的环形缓冲区，这个缓冲区默认大小为100MB，这个数值可以通过mapreduce.task.io.sort.mb设置，通常来说0.8是个阈值，如果缓冲区中的数据超过整体的0.8会发生溢写操作,之后后台会启动一个线程将这些数据spill到磁盘中。但是也并不是直接写入磁盘的，线程首先会根据Reducer的数量将数据进行分区，之后分区内排序，如果有combiner函数就要去执行，combiner会使得结果更加紧密，数据量更少。要注意的是多次溢写会导致产生多个文件，在任务完成前移除文件会被merge成一个分区的并且排好序的输出文件。当然写入磁盘之前也是可能进行数据压缩的，这也是个比较好的主意。Reduce端：map输出的文件会位于运行map task的tasktracker的本地磁盘上，map完全完成后，tasktracker需要运行Reduce任务去获取这些输出文件。这就是reduce任务的copy阶段，默认reduce有5个Copy线程，会轮询master来获取map的位置，知道map完成后通过心跳机制通知master后才可以获得位置并去copy数据。注意copy线程并会等待所有map task都完成后才去执行，而是能执行就执行。copy过来数据后，会进入merge阶段，这个阶段将合并map输出，维持其顺序排序。在最后的reduce阶段，对已排序的每个键都调用reduce函数，此阶段的输出会直接输入到HDFS。
3. **MapReduce调优**
    出现问题的原因(计算机性能上的因素：CPU，内存，磁盘健康，网络。IO操作优化：数据倾斜，map与reduce数量设置不合理，Map运行时间太长或太短，小文件太多，大量不可分块的超大文件，splill次数过多，merge次数过多)。  
* 数据输入阶段（合并小文件，因为大量的小文件会产生大量的map任务，而这些map任务执行时间很短，任务装载很耗时，可以使用CombineTextInputFormat来解决输入端大量小文件的场景）。  
* Map阶段(跳过调整mapreduce.task.io.sort.mb和mapreduce.map.sort.spill.percent参数来减少溢写次数。通过调整mapreduce.task.io.sort.factor增大merge时同时打开的文件数目，减少merge次数，从而缩短mr处理时间。在不影响业务逻辑的前提下，执行combine操作以减少IO)。  
* Reduce阶段(合理设置map和reduce数量，不能太少或太多，太少会导致task等待，延长处理时间，太多会导致map,reduce之间竞争资源。设置map和reduce共存，调整mapreduce.job.reduce.slowstart.completedmaps参数，默认0.05，表示至少map任务中，执行完成的maptask数不少于5%才启动reducetask。能不使用reduce就不使用reduce，因为shuffle过程很耗时。合理设置reduce端的buffer：默认情况下，数据达到一个阈值的时候，buffer中的数据就会写入磁盘，然后reduce会从磁盘中获得所有的数据。也就是说，buffer和reduce是没有直接关联的，中间多了一个写磁盘->读磁盘的过程，既然有这个弊端，那么就可以通过参数来配置，使得buffer中的一部分数据可以直接输送到reduce，从而减少IO开销mapreduce.reduce.input.buffer.percent，默认为0.0。当值大于0的时候，会保留指定比例的内存读buffer中的数据直接拿给reduce使用)。(4)IO传输阶段：采用数据压缩的方式，减少网络IO时间,安装snappy和lzo压缩编码器。使用sequencefile，写入速度更快。减少数据倾斜：数据倾斜主要发生在reduce阶段，可以通过抽样和范围分区、执行combine合并数据，多采用mapJoin。开启JVM重用。  
4.**MapReduce Counter:** 计数器是收集作业统计信息的有效手段之一，用于质量控制或者应用级统计。计数器还可用于辅助诊断系统故障。计数器通常可以被分为两类，系统内置的计数器和用户自定义的计数器。MapReduce任务计数器（hadoop任务计数器收集任务运行期间产生的信息，比如读写记录数）。文件系统计数器(会收集从文件系统读写字节相关的信息)。FileInputFormat计数器，由map任务通过FileInputFormat读取的字节数。MapReduce作业计数器（作业计数器由 application master 维护，因此无需通过网络传输数据，这一点包括 “用户定义的计数器” 在内的其他计数器不同。这些计数器都是作业级别的统计，其值不会随着任务运行而改变）。
