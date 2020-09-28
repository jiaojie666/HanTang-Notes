<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

1. **Kafka的架构：**
一个典型的kafka体系架构包括若干producer，若干broker,若干consumer(group),以及一个zookeeper集群。Kafka使用zookeeper管理集群配置，选举leader以及在consumer group变化的时候进行rebalance，producer发布信息到borker，consumer从broker中pull并消费信息。其中的broker我理解为就是服务器，他主要跟topic和partition两个概念有关。
(1)	Topic&partition:一个topic可以被理解为一类信息，每个topic被分成多个partition，每个partition在存储层面上是一个append log文件。任何发布到本partition的消息都会被追加到log文件的尾部，每条消息在文件中的位置被称为offset，offset是一个long类型的数字，唯一标记一条消息。Producer push来的消息是追加到partition中的，这是一种顺序写磁盘的机制，效率很高。
(2)	Kafka为什么要将topic分区：topic是逻辑概念，是面向producer和consumer的。而partition是物理层次的概念，如果partition不分区，那么所有topic的信息都位于同一个broker，那么关于这个topic的所有请求都会由同一个broker处理，吞吐量变小。
2. **Kafka高可靠性实现：**
Kafka文件存储机制:kafka中的消息按照topic进行分类，每个topic的信息又会被分为多个partition，但是partition并不是最小的存储粒度，partition可以进一步被细分为segment，一个partition一般是一个目录名。Segment是文件，有.log和.index两种。
(1)	为什么不以partition为最小存储单位？如果以partition为最小存储单位，随着kafka producer持续的发信息，必然会引起partition数量的无限扩张，将对消息文件的维护和已消费信息的清理带来巨大的麻烦，因此以segment为单位将partition进一步细分。这相当于每个partition巨型文件被平均分配到多个大小相等的segment段中。
(2)	Segment的工作原理：segment文件包含两部分，即.index文件和.log文件，分别表示segment索引文件和数据文件。Partition全局的第一个segment从0开始计数，后续每个segment文件名为上一个segment文件最后一条消息的offset数值。
(3)	如何从partition中通过offset查找message:首先根据二分法定义具体的文件，之后从起始位置读入Size大小的数据。
3.	**复制原理和同步方式**
LEO：LogEndOffset,每个partition最后一条message的位置，HW：highWatermark，高水位
为了提高消息的可靠性，kafka中每个topic的partition有N个副本，其中N>=1,是topic的个数，kafka通过多副本机制实现故障的自动转移，当kafka集群中出现broker失效的时候，副本机制保证服务可用。对于任何一个partition，它的N个副本中，有一个leader其他都为follower，leader负责读写请求，follower则被动的区复制leader上的数据。如果leader失效则从follower中优选一个成leader。
(1)	如何保证从follower中推举出的leadaer是优选呢？由于partition的多个副本通常分散于不同的broker上，由于带宽，读写性能，网络延迟等因素，同一时刻，这些副本的状态通常是不一致的。为了保证选出follower中一个最优的，即和leader的数据同步的，leader会维护和跟踪一个ISR（同步副本队列），这个队列中的副本和leader中保持同步，状态一直。ISR中的leader是优选。
(2)	同步副本ISR：ISR中包括（leader+与leader保持同步的followers），HW=min(max{LEO, all servers in LSR})，ISR的管理是由zookeeper来做的。
(3)	数据的可靠性和持久性保证：分别分request.required.acks=1,0,-1。分别代表producer发送数据到leader，leader写本地日志成功就返回成功，producer可以不停发消息给leader，leader要等待ISR中所有同步完成后再返回。
HW意义重大，当leader，或者follower宕机后，重新加载数据必须先将数据截断到HW，在重新同步数据。为了保证数据的一致性。
4.	**Kafka中zookeeper的作用**
在kafka中，zookeeper主要负责broker注册（为broker的注册信息构建一个临时节点，当broker会话失效时，zookeeper会删除该节点），topic注册(由于topic的partition需要备份，为了保证数据一致性，引入zookeeper的只leader写，follower跟随机制可以保证数据一致性)
Producer负载均衡：对于同一个topic的不同partition，kafka会尽力将这些partition分布到不同的broker服务器上，这种均衡策略是基于zookeeper实现的，在一个broker启动的时候，会首先进行注册，producers启动后也需要到zookeeper进行注册，并创建一个节点来监听brokers服务列表的变化。由于broker是临时节点，因此当brokers发生变化的时候，producers可以得到通知改变自己的broker list。主要是依靠zookeeper的watch机制。在生产中必须指定topic，但是partition有两种指定方式，即明确parition，或者设置partitioner。
5. **Kafka全程解析：**
(1)	Producer发布信息：producer采用push的方式将消息发布到broker，每条消息都被Append到partition中，属于顺序写。信息存入partition的时候会根据分区算法选择存储到哪个partition。如果指定了partition就直接使用，如果指定了key，就根据key hash一个partition，或者采用轮询的方式。具体的，producer先从zookeeper的/brokers/..state中找到该partition的leader，producer将消息发送给leader，leader将消息写入本地log,followers从leader pull消息，写入本地后向leader发送ACK信号，leader收到所有ISR中的ACK信息后，增加HW，并向producer发送ACK。
(2)	Consumer消费信息：consumer API提供了consumer group的语义，一个消息只能被group中的一个consumer所消费，且consumer本身在消费信息的时候并不关注offset，最后一个offset由zookeeper保存。
Kafka实现高吞吐量的原理：顺序读写（kafka的消息是不断追加到文件中的，这个特性充分利用了磁盘顺序读写的性能），零拷贝（在linux kernel2.2后出现的技术，它可以跳过用户缓冲区的拷贝，建立磁盘空间与内空间的直接映射），分区（大大增加了并行操作的能力），批量发送（kafka允许批量发送消息，producer在发送消息的时候可以先将消息缓存在本地，一段时间发送一次），数据压缩（kafka可以将消息集合进行压缩，减少数据传输量）
6. **Kafka如何实现消息不重复：** 
引入producer ID和Sequence Number实现produce的幂等语义，即，每个producer在初始化的时候会被分配一个唯一的pid，sequence numebr，对于每个PID，该producer发送数据的每个<topic,partition>都对应一个从0开始的sequence number, broker端也为每个<pid,topic,partition>维护一个序号，每次Commit一条消息都会将对应的序号+1，对于接收的每一条消息来说，如果其Sequence number比当前大1则接收，否则直接丢弃。从consumer端来说，直接建立一个消息的去重表就可以了。
7. **Kafka如何实现消息不丢失：**
Producer端：(1)使用带会ID小的send方法，设置acks为-1(all)，设置retries为一个较大的数值，尽量避免消息丢失
Broker端：(1)设置unclear.leader.election.enable=false（只允许ISR中的consumer成为leader），设置replication>=3，设置min.insync.replicas>1，设置消息至少被写入到多少个副本中才算是已提交。设置成大于1可以提升消息持久性。
Consumer端：enable.auto.commit设置为False,人为设置offset更新。
8. **Kafka过期数据的清理：主要有删除和压缩两种策略。**
	删除：设置log.cleanup.policy=delete启用删除策略，除此之外可以设置超过指定时间清理和超过指定大小后删除旧消息。
	压缩：将数据压缩，只保留每个key最后一个版本的数据。设置log.cleaner.enable=true，并设置log.cleanup.policy=compact

