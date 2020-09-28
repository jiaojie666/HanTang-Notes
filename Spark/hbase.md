<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

1. **Hbase架构：Client, HRegionServer, HRegion, Hlog, Store, memStore, StoreFile等.**
	**HMaster的作用：**不存在单点问题，只需要zookeeper保证同时只有一个服务即可，它主要负责region和table的管理工作，例如为HRegionServer分配HRegion,负责HregionServer的负载均衡，发现失效的HRegion并重新分配。
	**HRegionServer的作用：**维护HMaster分配给他的Region处理对这些Region的IO请求，负责切分正在运行过程中边的过大的HRegion。
	HRegion：Table在行的方向上分割为多个Region，是HBase中分布式存储和负载均衡的最小单元，不同的Region可以位于不同的RegionServer上。但是同一个Region是不会拆分到多个RegionServer上的，一般来说一个表一般只有一个Region。
	**Store：**每个Region由一个或多个Store组成，每个Store中包含一个ColumnsFamily信息，每个Store包含一个MemStore和若干个StoreFile,实际存储形式是HFile。
	**HLog:**是为了WAL设计的，HLog记录数据的所有变更，做灾难恢复用的。
2. **Zookeeper的作用：**
Hbase通过zookeeper来做master的高可用，regionServer的监控，元数据的入口和集群配置的维护工作等。具体来说，通过zookeeper来保证集群只有一个master在运行，如果master异常可以重新选举出新的master。监控RegionServer的状态，当Region Server异常的时候通过回调的形式通知Master信息。存储元数据的统一入口地址。
3. **Hbase读数据：分为新旧两个层次**
旧寻址方法：(1)访问zookeeper获得-root所在的RegionServer的地址(2)client请求-root所在的RS地址，获取.meta表的地址，client会将-root相关的信息Cache下来(3)client访问.meta表的RS地址，client会cache .meta信息(4)client请求访问数据所在的RS地址，读数据
旧的寻址方式其实是考虑metadata可能很多，要分多个Region去存储，但是实际上metadata并不会有想象中那么大。Bigtable中说一个metadata大小为1kb。因此完全可以省略访问root这一步
所以新的寻址方式为：
(1)client访问zk获取.meta所在的regionServer的地址(2)client请求.meta所在的RS获取数据所在的RS，这个.meta数据会被Client cache。(3)client请求数据所在的region server，获取所需要的数据。
4. **Hbase的特点：**
(1)分布式列式存储数据库，使用hdfs存储和zookeeper进行管理
(2)适合半结构化和非结构化数据
(3)数据稀疏，null不会被存储
(4)是主从架构，hmaster是主节点，hregionServer是从节点
5. **Hbase 和hive的区别：**
共同点（都是架构在hadoop上的，使用hdfs做存储）。区别：(1)hive是类sql引擎，它建立在hadoop之上来减少mapreduce编写工作的批处理系统，hbase则主要是为了弥补实时性操作(2)hive要使用mapreduce做计算来返回结果，运行时间长，但是hbase高效的多(3)Hive本身不存储和计算数据，完全依赖mapreduce和hdfs，hive的存储并不是完全依赖hdfs的(4)hbase是列式存储
Hbase中scan和get的功能以及实现的异同：get方法可以按照rowkey获取唯一的一条记录。Get方法包含两种情况，设置了closetRowBefore和没设置的rowlock，主要是用来保证行的事务性，即get使以一个row来标记的。Scan方法可以通过setCaching和setBatch方法来提高速度（空间换时间），scan可以设置setStartRow和setEndRow来限定范围，还可以设置setFilter来设置过滤器。也可以实现权标扫描。
6. **setCache和setBatch方法的使用：**
cache:默认情况下hbase会对返回的每个结果row都执行一次RPC操作，这样的话代价比较高。一种合适的想法是一次RPC返回多行数据，这样就会减少实际的通讯开销。但是cache并不是越大越好，尽管设置大了会提高性能，但是也会导致一次RPC的时间过长，甚至导致outofmemoryException异常错误。
Batch：可理解为cache主要是面向行的优化，而batch主要是面向列的优化。它可以用来设置每次调用next的时候会返回多少列。
7. **Hbase中compact的作用：**
当storeFile达到一定数量之和就要执行compaction操作(1)合并文件(2)清楚过期的，多余版本的数据(3)提高读写效率。主要包含两种compact：其一为minor compact只用来做部分文件的合并操作，以及设置minVersion等于0，并设置ttl的过期版本清理，不做任何删除数据，多版本数据的清理工作。Major compaction则是对region中Store下面所有的storeFile执行合并操作，最终只整理合并出一个文件。
