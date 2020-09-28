<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

## Zookeeper的watcher机制 ##
Zookeeper允许客户端向服务器端的某个znode注册一个watcher监听，当服务器端的一些指定事件触发了这个watcher，服务器端会向指定的客户端发送一个事件通知来实现分布式的通知功能，之后客户端根据watcher通知状态和事件类型做出业务上的改变。主要工作机制是，客户端注册watcher，服务器处理watcher，客户端回调watcher。Watcher的特性：一次性（一个watcher一旦被触发就会被删除掉），客户端串行执行（客户端回调watcher的过程是一个串行同步的过程），轻量级（watcher只会通知客户端发生了事件，但不会说明具体内容），只能实现最终一致性（由于ordering guarantee）.
## 客户端注册watcher##
- 调用getData/getChildern/exist三个api传入watcher对象
- 封装wather到watherRegistration
- 封装成packet对象，服务端发送request
- 将wather放入ZKWatherManager中进行管理。
### 服务器端处理wather ###
- 服务器端收到watcher并存储到watherManager的watch table和wathe2path中去
- watcher触发
- 将通知状态，事件类型，以及节点路径封装成为一个watcherEvent对象。
## ACL权限控制机制##
UGO是文件系统权限控制模式，ACL是访问控制列表主要包括三个方面，即权限模式，授权对象以及具体的权限。权限模式可以是ip,digest,world,super，授权对象包括制定的用户或实体，权限包括create,delete,read.wite和admin。
## FastLeaderElection原理##
有一些重要的概念(myid代表服务器的编号。zxid是64位的数，它代表一次更新操作的proposal id，高32位是epoch,低32位是递增的proposal id。服务器的状态主要有looking,following与leading)。选票的数据结构(logicClock表示这是该服务器发起的第多少轮投票，state表示当前服务器状态，self_id代表myid，self_zxid代表当前服务器上保存的最大zxid，vote_id被推举的服务器的myid,vote_zxid代表被推举的服务器上的最大zxid)。
### FastLeaderElection流程###
+ 自增选举轮次：zookeeper规定所有有效的投票必须属于同一轮次，每个服务器开始新一轮投票的时候，会先自增logicClock。
+ 初始化选票：每个服务器清空自己的投票箱，票箱中只会记录每一个投票者的最后一票。
+ 发送初始化选票：每个服务器都会通过广播将票投给自己。
+ 接收外部投票：服务器会尝试从其他服务器中获取投票，并将这些投票存入自己的票箱。
+ 判断选举轮次：如果外部投票的logicClock大于自己的，要清空票箱，并更新自己的logicClock,再将自己的投票广播出去，如果外部投票小于自己的logicClock则直接忽略，如果logicClock相等则进行选票PK.
+ 选票PK：主要是基于zxid和myid,如果外部投票的zxid比较大，则更新自己的vote_zxid和vote_id并广播出去，再将自己更新的票和收到的投票放入票箱。
+ 统计选票，以及更新服务器的state。
### 集群启动时候的领导选举###
（1）	初始投票给自己：集群刚启动的时候，logicClock为1，zxid都为0，各服务器初始化后都投票给自己，并将自己的一票放入票箱。
（2）	更新选票：服务器收到外部投票后，进行选票PK，响应的更新自己的选票并广播出去，并将票箱更新。
（3）	确定角色：根据选票确定角色。
### Follower重启选举###
（1）	Follower重启，或者发生网络分区后找不到leader,会进入looking状态并发起新的一轮选举。
（2）	判断集群中是否存在leader:如果当前follower在票箱中知道存在某个服务器为leader(有半数以上同意)，那么更改自身状态。
### Leader重启选举###
（1）	Follower发起新投票：Follower发现leader不工作后，自身进入looking状态并发起新一轮的投票，都投票给自己。
（2）	广播更新选票：Follower根据外部投票决定是否更新自身的选票，主要还是根据zxid和myid，如果zxid大小不同，则由zxid确地选票的**更新，如果zxid相同，则由myid确定选票的更新。**
（3）	选出新leader:确定新leader之后则更新自身状态。
（4）	旧leader恢复后发起选举：发起新一轮投票，但自身状态会变成follower。
## zookeeper分布式锁##
它基于znode的有序性和临时节点特性实现的。具体来说，使用zk的临时节点和有序节点，每个线程获取锁就是在ZK创建一个临时有序的节点，例如在某个目录下创建临时节点。创建节点成功后，获取当前目录下所有临时节点，判断当前线程创建的节点编号是不是最小的，如果当前创建的节点是最小的那么就认为获取锁成功，如果不是最小的，则对该节点的前一个节点添加一个时间监听。
## Zookeeper数据同步 ##
  * 数据同步的流程（observer和follower向leader进行注册，数据同步，同步确认）
  * 数据同步的种类：直接差异化同步，选回滚再差异化同步，仅回滚永不，全量同步。几个概念比较重要：peerLastZxid代表各个learner最后处理的zxid，minCommitedLog(leader proposal队列中最小zxid)，maxCommitedLog(队列中最大zxid)
  * 直接差异化同步：peerLastZxid处于min和max之间。先回滚再差异化同步：leader服务器发现某learner上包含自己没有的事务记录，就会让learner回滚。仅回滚同步：peerLastZxid>max的情况。全量同步，peerLastZxid<min
## Zookeeper的顺序一致性 ##
   + 顺序性：zookeeper中设计了zxid，它是递增的，所以zxid越大，数据越新。写入操作是leader负责，为了保证提案会按照zxid的顺序生效，zookeeper使用了一个concurrentHashMap来记录所有未被提交的提案，并被命名为outStandingProposal。Key是zxid，value是提案信息。必须确保前一个key不存在了，当前值才可以提交。
   + 一致性：使用两阶段提交方法来保证最终一致性。第一阶段将写入事件广播给所有的follower结点，可以写入的结点返回确认信息ack，如果ack数量超过一半就广播commit。
## Zookeeper的典型应用场景##
+	数据发布订阅：目的是动态的获取数据(配置信息)，实现数据的集中式管理和数据的动态更新。主要包含push和pull两个设计模式。数据量通常比较小，数据内容在运行时会动态更新，集群共享配置一致。如何实现？将数据存入一个znode上，并在该点注册一个数据变更watcher。一旦数据变更，客户端重新读取即可。
+	负载均衡：zookeeper的命名服务，利用zk创建一个全局路径。分布式通知和协调（改变结点的状态来实现）。对于执行任务情况汇报（每个工作进程都在某个目录下创建一个临时节点。并携带进度数据，这样可以监控子节点变化来监控情况）。
+	Zookeeper的命名服务，配置管理，zookeeper分布式锁
+	Zookeeper的集群管理：主要是是否有机器退出和加入以及选择master。
+	Zookeeper队列管理：同步队列（在约定目录下创建临时目录节点，达到某一数量才可以进行服务）。FIFO，通过有序性。
