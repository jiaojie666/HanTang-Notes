<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

1. **Leveldb的优点，特性：**leveldb，写性能特别优秀，是典型的LSM(log structured-Merge)-tree实现。它的核心思想是放弃部分读的性能，换取最大写入能力的实现。LSM树写性能很好的原理是尽量减少随机写的次数，对于每次写入操作，并不直接将最新的数据写入磁盘，而是(1)一次日志文件的顺序写(2)一次内存中数据的插入，leveldb将数据更新在内存中，等到数据达到一定阈值的时候在将这部分数据真正的刷新到磁盘文件中个，因此可以获得很高的写性能。
2. **LevelDB整体架构：**
(1)	Memtable:leveldb的一次写入并不是直接将数据刷新到磁盘文件，而是首先写入内存中代替，memtable就是leveldb内存中进行数据组织与维护的结构，所有的数据按照用户定义的顺序方法排序存储。它使用跳跃表来存储这部分数据。绝大多数操作时间复杂度为O(log n)。一般来说当memtable大小超过4MB之后，memtable会变为immutable memtable。
(2)	immutable memtable:memtable的容量到达阈值后，就会变为一个immutable memtable,两者结构定义完全一样，但它是只读的。
(3)	log(jornal)：leveldb的写操作并不是直接写入磁盘，而是首先写入内存。假设写入到内存的数据还没来得及持久化，leveldb就发生了异常，或者server宕机，那就会造成用户写入丢失，因此leveldb在写内存之前会首先将所有的写操作写到日志文件中，便于数据恢复，由于每次写入log都是顺序写，因此写效率很高。
(4)	sstable：leveldb先写内存的方式来提高写入效率，但是内存中的数据不可能无限增长，并且日志中记录的写入操作过多也会导致异常发生的时候，恢复时间过长。因此等到内存中数据到达一定容量的时候，就需要将数据持久化到磁盘中。Sst文件只读的，除了level 0层的Sst之间可能存在交集，其它层次不同sst之间不存在任何交集。
(5)	manifest：leveldb中有版本的概念，一个版本中主要记录了每一层中所有文件的元数据，元数据主要包括(1)文件大小(2)最大最小key值。版本信息十分关键，一方面在查找数据的时候可以利用维护的每个sst文件的最大最小值来加快查找，另一方面则是manifest中维护了一些进行compaction的统计值来控制compaction的进行。具体来说，每次compaction完成后，就会创建一个新的版本信息。而manifest是用来记录versionEdit信息的，一个versionEdit数据，会被编码成一条记录写入到manifest文件中。每条versionEdit记录包括：(1)新增了哪些SST文件(2)删除了哪些SST文件(3)当前compaction的下标(4)日志文件编号(5)操作seqNumber等信息。VersionNew=versionOld+versionEdit
(6)	current：这个文件中的内容只有一个信息，就是自己在当前的manifest文件名。他会储存那个manifest才是我们最关注的
3. **leveldb写入操作：**
  leveldb写入整体流程：先写入log文件，再写入内存。其实也不是写入到log文件中后就一定不会有数据丢失，因为操作系统并不是写文件直接写入到磁盘，它也是由缓存的，如果直接系统挂掉，这部分数据还是会丢失。
  leveldb的写类型：有put和delete两种。其实都是put操作，只不过delete操作的value为空。无论是put/del还是批量操作，底层都会为这些操作创建一个batch作为数据库操作的最小执行单元。Batch中每个数据项包括:type(put/del)+keyLen+key+valueLen+value
  leveldb实际key：是internalKey,它是由userKey+sequenceNumber+type(put/del)构成的.
  Leveldb合并写：leveldb在面对并发写入的时候，做了一个处理优化。在同一时刻只允许一个写入操作将内容写入到日志文件以及内存数据库中。为了在写入进程较多的情况下，减少日志文件的小写入，增加整体写入性能，leveldb将一些小写入合并成为大写入。
第一个获取写锁的写操作，在当前写操作的数据量没达到合并上限&&有其他写入操作在等待的情况下，将其他写入操作内容合并到自身。之后再写入log文件和内存中。
4. **Leveldb读操作：**
Leveldb读取数据的接口：直接通过get接口读取数据或者创建一个snapshot，在基于这个snapshot接口读取数据。其实两者本质是相同的，因为get方法也会默认的以当前的数据库状态创建一个snapshot,并基于这个快照读取。
快照是什么：快照代表数据库某一个时刻的状态，在leveldb中，使用sequenceNumber来表示数据库的状态。所以每个序列号都可以作为一个状态快照。使用快照可以保证数据库进行并发的读写操作。在获取到快照后，leveldb会为本次查询构建一个internalKey，这种方式可以快速过滤掉所有seq中大于快照号的数据项。
具体来说，数据读取分为以下三步：
(1)	在memory db中查找指定的key，若搜索到符合条件的数据项，就结束查找。
(2)	在immutable table中查找指定key，若搜索到符合条件的数据项就结束查找。
(3)	按底层到高层的顺序在level i层的sst中查找指定key。要注意在level 0层中，一定要从编号的sst文件中开始查找，因为这些数据最新。
5. **Leveldb中的日志：**
日志的结构：
 为了增加读取效率，日志文件中按照block进行划分，每个block大小是32kb,每个block中包含若干个完整的chunk。
   Chunk=checksum+length+type(first|middle|last|full)+data其中checksum检验type+data。由于数据block不足够或者是chunk过大。
  日志的data部分：
   sequenceNumber+entryNumber(put/del数量)+[batch data]+
6. **Leveldb中SStable:**
   SStable的物理结构：每个SST文件按照固定大小进行分块，默认每个块大小为4kb，每个block中包含如下部分：
   Block=data+压缩类型+crc
   SST中block的类型，注意这些block都有压缩类型和CRC：
Data block:主要用来存储key value数据，由于sstable中所有的keyvalue都是严格按序存储的，为了节省存储空间，leveldb并不会为每一对keyvalue都存储完整的key值，而是存储与上个key非共享的部分，避免了key重复内容的冗余存储。但是每间隔若干个keyvalue对就会重新存储一个完整的key，被称为restart point。由于每个Restart point存储的都是完整的key值所以在sstable中进行数据查找的时候，可以首先利用restart point的值进行范围缩小，以便于定位目标数据所在区域。
       Filter block:为了加快sstable中数据查询的效率，在直接查询datablock中的内容之前，leveldb首先根据filter block中的过滤数据判断指定的datablock中是否有需要查询的数据，如果不存在则不需要对这个datablock进行数据查找。
	  Meta index block：用来存储filter block在整个sstable中的索引信息，是一个key value结构，其中key是filter.与过滤器名字组成的常量字符串。Value是在sstable中的偏移量和数据长度。
	  Index block：主要用来存储所有data block相关的索引信息。其中包含多条记录，每个记录代表一个data block索引信息（data block i中的最大key值，data block起始地址的偏移量，该data block的大小）。
Leveldb缓存系统：
	Leveldb中使用基于LRUCache的缓存机制，用于缓存两个方面的内容：已打开的sstable文件对象和相关元数据，datablock中的内容
Cache结构：主要包含两部分内容，hashTable（用来存储数据），LRU（用来维护数据项的新旧信息）
	Leveldb中的hash table可以实现resize的过程中不阻塞其他并发的读写请求。LRU中则是当整个cache中存储的数据容量达到上限的时候会根据LRU算法自动删除最旧的数据，使得cache的容量保持一个常量。
	Hash table:一个hash表通常是由若干个bucket组成，每一个bucket中会存储若干条被散列至此的数据项。当hash进行Resize的时候，需要将旧桶中的数据读出，并重新散列到一个新桶中。这个过程如果不是原子操作，可能会导致其他并发的读写请求结果发生异常。为了解决这样的问题，论文中提出：一个bucket的数据是可以冻结的。具体来说，当一个hash桶中的数据超过32个时候或总数据量达到某一阈值的时候，会触发hash table的扩张。
7. **Hash table的扩张的过程为：**(1)计算新hash table中桶的个数，即原来的一倍(2)创建一个新的hash table，并将旧的hash table转换为一个过渡期的hash table，这个过渡期的hash table是只读的(3)后台将过渡期的hash table中的数据分散到新的hash table中去。值得注意的是，在完成新的hash table构建的整个过程中，hash table不会拒绝服务，所有的读写操作仍旧可以进行。Hash table的扩张过程中，封锁粒度是bucket。当有新的读写请求发生的时候，如果被散列之后的hash桶还没有构建完成，那么就主动进行构建。并将构建后的hash桶填入新的hash table中。后台进程构建到该桶的时候发现已经被构建，那么就无需重复构建了。
Leveldb compaction:
8. **Compaction的作用：**
(1)数据持久化：主要指的是minor compaction，它将内存中的immutable memtable dump成level 0中的sst。
(2)提高读写效率:其实主要是优化读操作，假如没有major compaction，由于level 0层中的数据是存在overlap的，那么如果一个数据不能在内存中获得，就需要去level 0层中寻找数据，在最差的情况下可能需要遍历所有level 0层数据。有了major compaction就可以将一层文件向下放，例如1层2层，这些层的文件都是不存在overlap的，所以查找效率大大提升。
(3)平衡读写差异：由于major compaction是一个多路归并过程有大量的磁盘读写开销，因此非常耗费时间，如果写入速度特别高，那么会导致0层文件还是过多，因此leveldb设置了两个参数来控制写入速度，甚至暂停写入。
(4)整理数据：利用多版本数据，以及delete类型的数据都是在compaction过程中被操作的。
  Compaction过程：主要说major compaction
(1)	触发条件：当0层文件超过预订上限的时候（默认4），当i层文件总大小超过(10`i)MB的时候，当文件无效读取的次数过多
(2)	具体的过程：
a)	寻找合适的输入文件：对于level 0层文件合并，采用轮转的方法选择输入文件，每一次都记录上一次该层合并文件的最大key值，下一次选择此key之后的首个文件。对于错峰合并起始文件则是查询次数过多的文件。
b)	扩大输入集合：首先标注一个起始输入文件，之后在level i和level i+1层中寻找与key重叠的文件，标注这些文件。
c)	利用两层输入文件，在不扩大level i+1层输入文件的前提下，再去level i层寻找key重叠文件。之后寻找完毕
