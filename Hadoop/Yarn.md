<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

YARN是一种新的hadoop资源管理器，他是一个通用的的资源管理系统，他可以为上层应用提供统一的资源管理和调度，它的引入为集群在利用率，资源统一管理和数据共享方面带来了巨大的好处。Yarn是hadoop2.x中的资源管理系统，他是一个通用的资源管理模块，可以为各类应用程序提供资源管理和调度功能。
1. **Yarn的设计思路**：
Yarn是为了解决hadoop1.x中一些缺陷引入进来的，这些缺陷包括：单点故障，JobTracker可能任务过重，上限是4000个节点。容易出现内存溢出问题，例如分配资源的时候只考虑mapreduce任务数量而不考虑cpu和内存，资源分配不合理。
2. **Yarn的组件架构：**
(1)ResouceManager:处理客户端请求，启动和监控Application Master，监控Node Manager，资源调度和分配。
(2)Application Master:为应用程序申请资源并分配给内部任务，任务调度，监控与容错。
(3)NodeManager：单个结点上的资源管理，处理来自ResourceManager的命令，处理来自ApplicationMaster的命令。
ResouceManager:是一个全局的资源管理器，它负责整个系统的资源管理和分配，主要包含两个组件（调度器和应用程序管理器）。调度器接收来自Application Master的应用程序自愿请求，把集群中的资源以容器的形式分配给应用程序，容器的选择通常会考虑应用程序所要处理的数据的位置，进行就近选择，实现计算向数据靠拢。
3. **容器**：
容器是用于执行特定应用程序的进程，每个容器都有资源限制（内存，CPU等）。容器可以是Unix进程也可以是linux cgroup(是Linux内核提供的一种可以限制单个进程或者多个进程所使用资源的机制，可以对cpu，内存等资源实现精细化的控制)。
应用程序管理器：负责系统中所有应用程序的管理工作，例如应用程序的提交与调度器协商资源以启动Application Master，监控master运行状态以及在失败时重新启动等。是yarn资源分配和调度的基本单位，目前yarn仅仅封装了内存和、cpu资源。Container由application向ResourceManger申请，由resourceManger负责分配。Container的运行由applicationMaster向资源所在的NodeManager发起，container运行时需要任务命令。
4. **Application Master:** 
ResouceManager接收用户提交的作业，按照作业的上下文信息和从NodeManager收集来的容器状态信息，启动调度过程为用户的作业启动一个Application Master。 
Application Master的主要功能是:
+ 当用户作业提交时，master会和resource manager协商获取资源，Resource Manager则会以容器的形式为master分配资源。
+ 把获得的资源进一步分配给内部的map和reduce任务，实现资源的二次分配。与Node manager保持交互通信进行应用程序的启动，运行，监控和停止。监控资源的使用情况，并在任务失败的时候执行失败恢复。定期向Resource Manager发送心跳消息报告资源使用情况和应用进度。当作业完成则向resource manager注销容器。
5. **Node Manager:** 
容器生命周期管理，监控容器资源使用情况，跟踪结点健康状况，用心跳的方式和Resource manager保持通信等。特殊注意的是Node manager只负责管理抽象的容器，只处理与容器相关的事情而不具体负责每个人物自身状态的管理，这些管理工作由application master负责。
6. **Yarn的工作流程：**
用户编写客户端应用程序，向YARN提交应用程序，提交的内容包括Application Master程序，启动的命令和用户程序等。Resource Manager负责接收和处理来自客户端的请求并为程序分配容器，在容器中启动application master。Application master被创建后会首先向resource manager进行注册。Application master会使用轮询的方法想Resource manager申请资源。Resource manager则以容器的形式将资源分配给application master。在容器中整整的启动任务。各个任务向application master汇报自己的状态和进度。应用程序完成后，application master会向Resource manager的应用程序管理器注销并关闭自己。
7. **Yarn对比mapreduce1.0:** 
大大减少了resouce manger的资源消耗。Application Master来完成需要大量资源消耗的任务调度和监控。多个作业对应多个application master,实现监控分布化。Mapreduce同时是计算框架又是调度框架，他只能支持mapreduce计算。但是yarn是一个纯粹的资源调度管理框架，它上面可以运行包括mapreduce在内的不同类型的计算框架，只要编程实现对应的application master。Yarn更高效，并且以容器为单位而不是slot。
Yarn的HA如何实现：通过引入冗余的Resource manager来解决单点故障问题。
8. **YARN中的调度：**
YARN中包含三种调度器，分别是FIFO调度器，容量调度器和公平调度器。
**FIFO调度器：**将应用放置在一个队列中，然后按照应用提交的顺序（先进先出）运行应用，依次执行。它的优点是，简单易懂，不需要任何配置但是不适合共享集群，因为大的应用可能会占用集群中所有资源，所以每个应用必须等待直到轮到自己才可以运行。在一个共享集群中，更适合的是容量调度器和公平调度器。这两种调度器都允许长时间运行的作业能及时完成，并且还能保证较小的应用能在短时间返回结果。
**容量调度器：**使用容量调度器时需要一个独立的专门的队列来保证小作业一提交就会被启动，由于队列容量是为那个队列中的作业所保留的，因此这种策略会牺牲一部分集群的利用率。这也就意味大作业执行的时间要长。
**公平调度器**：使用公平调度器不需要预留一定量的资源，因为调度器会在所有运行的资源之间动态的平衡资源。第一个作业启动时，它是唯一运行的作业，因此它可以获得全部的资源。当第二个作业启动时，它会被分配集群中的一半资源。当小作业执行完并释放出资源后，大作业再次获得集群的全部资源，最终既得到了较高的集群利用率，又保证小作业能及时的完成。
容量调度器的配置：容量调度器允许多个组织共享一个hadoop集群，每个组织都会分配到全部集群资源的一部分。每个组织都会被配置一个专门的队列，每个队列被配置为可以使用一定的集群资源。队列可以进一步按照层次划分，这样每个组织内不同的用户能够共享该组织队列所分配的资源，在一个队列内，使用FIFO策略来对应用进行调度。单个作业使用的资源不会超过其队列容量，当然如果队列资源不够用了，容量调度器会可能会将空余的资源分配给队列中的作业，哪怕这会超出队列容量。这就是弹性队列。正常操作的时候，容量调度器不会通过强行终止来抢占容器。因此如果一个队列一开始的资源够用，随着需求增长资源不够用了，那么这个队列只能等待其他队列释放资源。为了缓解这种情况，我们可以为队列设置一个最大容量，这样的话，队列就不会抢占其他队列过多资源。

