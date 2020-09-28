<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>


1. **Hive内部表和外部表**：被external修饰的是外部表，不被修饰的是内部表。具体来说，创建内部表的时候会将数据移动到数据仓库指向的路径，但是外部表却只记录数据所在的路径，不对数据的位置做任何改变。因此删除表的时候，内部表的元数据和数据会同时被删除，而外部表只删除元数据。内部表由hive管理而外部表由hdfs管理。Hive有的数据都是由HDFS管理，没有为他建立任何索引。
Hive中的四种排序方式：order by(要求全局有序，因此只能有一个reducer), sort by(是一种局部有序，每个reducer中的数据是有序的), distribute by(按照指定的字段对数据进行划分到不同的Reduce中), cluste by(具备distribute by和sort by功能，但只能倒/降序排序)。
Hive中metaStore的三种方式： 主要是被用来存储元数据，包括表名，表的列，分区以及数据所在的目录等。客户端连接metaStore，metaStore再去连接mysql来存取元数据，就可以使得多个客户端同时连接服务。一般使用关系数据库存储这些元数据信息。内嵌derby方式（在同一时间只能有一个进程连接使用数据库），local方式（本地mysql存储元数据），远程方式（远程mysql一般常用此种方式）。
Hive中join方式：Join,left outer join, right outer join, full outer join, left semi join, cross join。
2.** Hive数据倾斜：**
原因：key分布不均匀，数据本身就存在这样的问题，分区方法问题，某些SQL语句本身的问题。例如join的时候某key值过多，group by某值的数量过多。          
HiveSQL常用优化方案：
(1)	列查询与分区裁剪：在查询时只查询需要的列，分区裁剪则是只读取需要的分区。尽可能加强约束条件
(2)	谓词下推：关系数据库中也有这个概念，将SQL语句中的where谓词逻辑尽可能提前执行
(3)	Sort by代替order by
(4)	Group by代替distinct：当要统计某一列的去重数时候，count(distinct)会非常的缓慢，因为它只有很少的reducer处理。所以我们可以尝试使用group by来改写。但是group by会启动两个MR job，因此一般只有当数据量特别大的时候才考虑去使用group by。
3. **Join优化：**
(1)	Buil table前置，即小表前置，Hive在解析的时候会默认将最后一个表看做probe table，而将前面的表作为build table并试图读入内存，如果顺序写反，那么很可能产生OOM问题。
(2)	大表join大表：将空值的key加随机数，使得倾斜的数据分配到不同的Reduce上，由于null关联不上，对结果没有影响。
(3)	多表join时尽量控制key相同：这样的话就可以将多个join合并为一个MR job来执行。
(4)	利用Map Join特性：map join特别适合大小表的情况，Hive会将build table和probe table直接在map端完成join过程，消灭了reduce，效率很高。Map join的配置项是hive.auto.convert.join默认值为true。当build table<hive.mapjoin.smalltable.filesize的时候就会启用map join。
4. **Hive对应的MapReduce优化：**
Mapper个数：mapper=min{split_num,max{default_num,mapred.map.tasks}}
Reducer个数：使用mapred.reduce.tasks可以直接设定reduce_num=min{total_input_size/reduces.bytes.per.reducer, reduces.max}
合并小文件：输入阶段合并，将hive.input.format设置为org.apache.hadoop.hive.ql.io.CombineHiveInputFormat，输出阶段合并：将hive.merge.mapfiles和hive.merge.mapredfiles都设置为true
启用压缩：压缩方式一般采用snappy,设定hive.exec.compress.intermediate为True
JVM重用：mapred.job.reuse.jvm.num.tasks=n，代表MR job中顺序执行的n个Task可以重用一个JVM，对不同MRjob的task无效。
并行执行与本地模式：hive中互相没有依赖关系的job之间可以并行处理，典型的有union操作。可以设置hive.exec.parallel为true。
本地模式指的是hive可以不将任务提交到集群进行运算，而是直接在一台节点上处理。适合数据量很小并且逻辑运算简单的操作。
严格模式：不允许用户执行三种有风险的Sql语句，一旦执行会直接失败，e.g., 查询分区表时不限定分区列，join产生笛卡尔积，用order by排序但不指定limit的语句。
合适的存储方式：hiveSQL可以选择Store as ,它支持textFile,sequenceFile,Parquent等。绝大多数使用会使用textFile和parquent。像parquent必须使用TextFile来中转，即不能直接将数据写入到parquent文件中。
5. **HiveSQL如何解析成MR job:**
(1)	SQL Parser:Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为AST Tree。
(2)	Semantic Analyzer:遍历AST Tree, 抽象出查询最基本的组成单元Query Block
(3)	Logic Plan: 遍历Query Block，翻译为Operator Tree
(4)	Logical Plan Optimizer:逻辑优化器进行operator Tree变换，合并不必要的ReduceSinkOperator,减少shuffle数量
(5)	Physical Plan：遍历operator tree，翻译为mapReduce任务
(6)	Logical Plan Optimizer:物理优化器进行mapreduce任务变换，生成最终执行计划
6. **Hive分桶与分区：**
(1)	Hive的select查询一般会扫描整个表的内容，但有些时候我们只需要扫描表中关心的一部分数据，因此建表的时候引入了partition概念。例如互联网的日志，我们可能只关心某些日期内的日志。它可以根据一个表的某列或者某些列来决定。
(2)	对于Hive中的某些表和分区可以进一步组织成为桶，一般一个桶是一个文件，按照某列的hash取值进行分桶。
分桶是有很大的好处的，例如它能获得更高的查询效率。桶为表加上了额外的结构。Hive在处理的时候能利用这个结构。简而言之就是方便使用map join。大大减少join的数据量。


