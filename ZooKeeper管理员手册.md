# ZooKeeper管理员手册

**部署和管理指南**

- [部署](#部署)
  - [系统需求](###系统需求)
    - [支持的平台](####支持的平台)
    - [软件需求](####软件需求)
  - [集群（多服务器）设置](###集群（多服务器）设置)
  - [单一服务器和开发者设置](###单一服务器和开发者设置)
- [管理](##管理)
  - [设计ZooKeeper部署方案](###设计ZooKeeper部署方案)
    - [跨机器需求](####跨机器需求)
    - [单机需求](####单机需求)
  - [配置](###配置)
  - [部署之前：ZooKeeper的优点和局限性](###部署之前：ZooKeeper的优点和局限性)
  - [管理](###管理)
  - [维护](###维护)
    - [持续清理data目录](####持续清理data目录)
    - [调试日志清理（log back）](####调试日志清理（logback）)
  - [监控](###监控)
  - [检测](###检测)
  - [日志打印](###日志打印)
  - [故障排除](###故障排除)
  - [配置参数](###配置参数)
    - [最小配置](####最小配置)
    - [高级配置](####高级配置)
    - [集群选项](####集群选项)
    - [加密、身份验证、授权选项](####加密、身份验证、授权选项)
    - [实验选项/功能](####实验选项/功能)
    - [不安全选项](####不安全选项)
    - [禁用数据目录自动创建](####禁用数据目录自动创建)
    - [启用db存在验证](####启用db存在验证)
    - [性能调整选项](####性能调整选项)
    - [AdminServer配置](####AdminServer配置)
    - [指标提供](####指标提供)
  - [使用Netty框架进行通信](###使用Netty框架进行通信)
    - [Quorum TLS](###Quorum TLS)
    - [非TLS集群不停机升级](###非TLS集群不停机升级)
  - [ZooKeeper命令](###ZooKeeper命令)
    - [Four Letter Words](####Four Letter Words)
    - [AdminServer](####AdminServer)
  - [数据文件管理](###数据文件管理)
    - [数据目录](####数据目录)
    - [日志目录](####日志目录])
    - [恢复 - TxnLogToolkit](####恢复 - TxnLogToolkit)
  - [注意事项](###注意事项)
  - [最佳实践](###最佳实践)

## 部署

本节包含Zookeeper部署的相关信息，并且涵盖以下主题：

- [系统需求](###系统需求)
- [集群（多服务器）设置](###集群（多服务器）设置)
- [单一服务器和开发者设置](###单一服务器和开发者设置)

前两小节针对生产环境（比如数据中心）中的ZooKeeper部署。第三小节涵盖了在有限的基础（非生产环境）上设置ZooKeeper的情况——用于评估、测试或开发。

### 系统需求

#### 支持的平台

ZooKeeper由多个组件组成。有些组件被广泛支持，而其他组件仅在少数平台上被支持。

- client是Java client library，应用程序使用连接到ZooKeeper集群。
- server是运行在ZooKeeper集群中的Java server。
- native client是由C语言实现的client，与Java client类似。
- Contrib是指多个可选的附加组件。

下表展示了不同的操作系统平台对各组件的支持情况。

| Operating System | Client                  |         Server          |      Native Client      |         Contrib         |
| :--------------: | ----------------------- | :---------------------: | :---------------------: | :---------------------: |
|    GNU/Linux     | Development、Production | Development、Production | Development、Production | Development、Production |
|     Solaris      | Development、Production | Development、Production |      Not Supported      |      Not Supported      |
|     FreeBSD      | Development、Production | Development、Production |      Not Supported      |      Not Supported      |
|     Windows      | Development、Production | Development、Production |      Not Supported      |      Not Supported      |
|     Mac OS X     | Development Only        |    Development Only     |      Not Supported      |      Not Supported      |

对于上表中没有明确提到支持的操作系统，ZooKeeper组件不能确定能够正常运行。ZooKeeper社区会修复其他平台报告的明显bug，但不会提供完整支持。

#### 软件需求

ZooKeeper在Java中运行，版本为1.8或更高（JDK 8 LTS, JDK 11 LTS, JDK 12 - 不支持Java 9和10）。它以ZooKeeper服务器集群的形式运行。三台ZooKeeper服务器是推荐的最小规模，我们也建议它们在不同的物理机上运行。

### 集群（多服务器）设置

为了搭建可靠的ZooKeeper服务，ZooKeeper应该以集群的方式部署。只要集群中的多数机器是可用状态，服务就是可用的。由于ZooKeeper的投票机制需要多数同意，所以最好使用奇数数量的ZooKeeper节点。例如，四台机器的ZooKeeper集群只能应对一台机器的故障；如果两台机器发生故障，剩下的两台机器就不构成多数。然而，五台机器的ZooKeeper集群可以应对两台机器的故障。

Note

> 正如[ZooKeeper Getting Started Guide](https://zookeeper.apache.org/doc/r3.8.0/zookeeperStarted.html)中提到的，容错集群的搭建至少需要三台服务器，强烈建议使用奇数数量的服务器。
>
> 通常来说，三台服务器对于生产安装来说是绰绰有余的，但为了在维护期间获得最大的可靠性，建议至少安装五台服务器。对于三台服务器，如果你在其中一台服务器上进行维护，那么在维护过程中，你很容易受到另外两台服务器之一的故障影响。如果你有五台服务器在运行，你可以关闭其中一台进行维护，并且如果其他四台服务器中的一台突然发生故障，仍然不会发生问题。
>
> 冗余考虑应该包括环境的所有方面。如果有三台ZooKeeper服务器，但它们的网线都插在同一个网络交换机上，那么这个交换机的故障将导致整个系统的瘫痪。

以下是集群节点安装步骤。

1. 安装Java JDK。可以使用系统安装包或者从以下网站下载：http://java.sun.com/javase/downloads/index.jsp

2. 设置Java堆大小。这对于避免swap非常重要，因为swap会严重降低ZooKeeper的性能。保守一点——对于4GB的机器，使用最大堆大小为3GB。

3. 安装ZooKeeper安装包。可以从以下网站下载： http://zookeeper.apache.org/releases.html

4. 创建一个配置文件。该文件可以任意命名。使用以下配置作为起始配置：

   >```
   >tickTime=2000
   >dataDir=/var/lib/zookeeper/
   >clientPort=2181
   >initLimit=5
   >syncLimit=2
   >server.1=zoo1:2888:3888
   >server.2=zoo2:2888:3888
   >server.3=zoo3:2888:3888
   >```

   你可以在[配置参数](###配置参数)一节中找到上述参数以及其他参数的具体含义。有几个问题需要在这里提一下：集群中的每台机器都需要知道集群中其他机器的存在。配置文件中通过一系列 **server.id=host:port:port**形式的配置来实现这一目标（host和port参数很简单，对于每台机器，需要指定一个首要端口和备用端口用于leader的选举）。从3.6.0起，每个ZooKeeper实例可以[指定多个地址](#madresses)（当集群中可以并行使用多个物理网络接口时，该特性可以提高可用性）。每台机器都可可以创建一个名为myid的文件来设置该服务器的id，该文件位于该服务器的data目录中，由配置文件参数**dataDir**指定。

5. myid由单独的一行组成，仅包含本机的id。所以服务器1的myid文件将仅包含“1”。集群中的id值必须唯一并且在1到255之间。**重要提示**：由于内部限制，如果要启用类似TTL Nodes（见下文）等的扩展特性，id值需要限制在1到254之间。

6. 在myid文件的同级目录下创建一个初始化标记文件*initialize*。该文件表明服务器需要一个空的data目录。如果该文件被创建，ZooKeeper会创建空的数据库并将该文件删除，否则，一个空的数据目录将意味着这个节点没有投票权，它将不会填充数据目录直到与一个活跃的leader进行通信。仅仅在新创建集群时需要创建该文件。

7. 如果配置文件已就绪，可以通过以下命令启动ZooKeeper：

   ~~~shell
    java -cp zookeeper.jar:lib/*:conf org.apache.zookeeper.server.quorum.QuorumPeerMain zoo.conf
   ~~~

   QuorumPeerMain启动一个ZooKeeper服务器，JMX管理bean也被注册，允许通过JMX管理控制台进行管理。[ZooKeeper JMX文档]([ZooKeeper: Because Coordinating Distributed Systems is a Zoo (apache.org)](https://zookeeper.apache.org/doc/r3.8.0/zookeeperJMX.html))包含了用JMX管理ZooKeeper的细节。详情请查看脚本bin/zkServer.sh，它包含在发行版中，是一个启动服务器实例的例子。

8. 通过连接到主机来进行部署测试。在Java中，你可以运行以下命令来执行简单的操作：

   ~~~shell
    $ bin/zkCli.sh -server 127.0.0.1:2181
   ~~~

### 单一服务器和开发者设置

如果为了开发目的设置ZooKeeper，用户可能希望设置一个ZooKeeper的单一服务器实例，然后在开发机器上安装Java或C客户端库以及依赖。

除了配置文件更简单外，设置单服务器实例的步骤与上述类似。完整的安装说明可以在[ZooKeeper Getting Started Guide](https://zookeeper.apache.org/doc/r3.8.0/zookeeperStarted.html)的[Installing、Running ZooKeeper in Single Server Mode](https://zookeeper.apache.org/doc/r3.8.0/zookeeperStarted.html#sc_InstallingSingleMode)一节中找到。

关于安装客户端库的信息，请参考[ZooKeeper Programmer's Guide](https://zookeeper.apache.org/doc/r3.8.0/zookeeperProgrammers.html)的[Bindings](https://zookeeper.apache.org/doc/r3.8.0/zookeeperProgrammers.html#ch_bindings)一节。

## 管理

本节包含ZooKeeper运行维护相关信息并包含以下主题：

- [设计ZooKeeper部署方案](###设计ZooKeeper部署方案)
- [配置](###配置)
- [部署之前：ZooKeeper的优点和局限性](###部署之前：ZooKeeper的优点和局限性)
- [管理](###管理)
- [维护](###维护)

### 设计ZooKeeper部署方案

ZooKeeper集群的可靠性依赖于两个基本假设：

1. 集群中的节点只有少部分失效。*失效*在此处的含义是某台机器宕机，或者某些网络故障导致某一结点处于离线状态。
2. 部署的机器正确运行。正确运行意味着正确地执行代码，拥有正常工作的时钟，以及拥有性能稳定的存储和网络组件。

下面的章节包含了ZooKeeper管理员的注意事项，以最大限度地满足这些假设。其中有些是跨机器的考虑，有些则是在部署中的每台机器都应该考虑的问题。

#### 跨机器需求

为了使ZooKeeper服务处于可用状态，可以互联的机器必须占到集群中的大多数。对于有N个服务器的ZooKeeper合集，如果N是奇数，集群能够容忍多达N/2个服务器故障而不丢失任何znode数据；如果N是偶数，合集能够容忍多达N/2-1个服务器故障。

例如，如果我们有一个有3个服务器的ZooKeeper集群，该集群能够容忍1（3/2）台服务器故障。如果我们有一个有5台服务器的ZooKeeper集群，该集群能够容忍多达2（5/2）台服务器故障。如果ZooKeeper集群有6台服务器，集群也能容忍最多2（6/2-1）台服务器故障而不丢失数据，同时防止 "脑裂 "问题。

ZooKeeper合集通常有奇数数量的服务器。这是因为在服务器数量为偶数的情况下，容错能力与少了一台服务器的集群相同（5节点集群和6节点集群均为2各节点故障），但集群必须为多了一台服务器保持额外的连接和数据传输。

为了提高故障容忍能力，各节点应该保持独立。例如，如果大多数节点共享同一个交换机，该交换机的故障可能会导致相关的故障，并使服务瘫痪。共享电源电路、冷却系统等也是如此。

#### 单机需求

如果ZooKeeper不得不与其他应用程序公用存储介质、CPU、网络或内存等资源的访问权的话，那么其性能将受到明显的影响。ZooKeeper有很强的持久性保证，这意味着它使用存储介质来记录变化，然后才允许导致变化的操作完成。应该关注这种依赖性并且采取谨慎措施以避免ZooKeeper的操作因为介质而被耽搁。有一些措施可以尽量减少这种性能恶化的情况。

- ZooKeeper的事务日志必须在一个专用设备上（一个专用的分区是不够的）。ZooKeeper会按顺序写入日志，而不会去搜索。 与其他进程共享日志设备会导致搜索和争用，这反过来又会导致多秒的延迟。
- 避免可能导致ZooKeeper swap的情况发生。为了使ZooKeeper高效运行，不能让swap发生。因此，请确保给ZooKeeper的最大堆大小不超过ZooKeeper可用的实际内存量。关于这一点的更多信息，请参见下面的[注意事项](###注意事项)。

### 配置

### 部署之前：ZooKeeper的优点和局限性

### 管理

### 维护

ZooKeeper几乎不需要长期维护，但是以下几点是需要注意的：

#### 持续清理data目录

ZooKeeper数据目录包含的文件是由特定服务群组存储的znode的持久性拷贝。 这些文件是快照和事务日志文件。当znode发生变化时，这些变化被追加到事务日志中。当日志变大时，包含所有znode的当前状态的快照将被写入文件系统，并为未来的事务创建一个新的事务日志文件。在快照期间，ZooKeeper可能会继续将传入的事务追加到旧的日志文件中。因此，一些比快照更新的事务可能会在快照之前的最后一个事务日志中发现。

ZooKeeper服务器在使用默认配置时不会删除旧的快照和日志文件（见下文的自动更新），这是由运维人员负责的。每个服务环境都是不同的，因此管理这些文件的要求可能因安装不同而不同（例如备份）。

PurgeTxnLog工具实现了一个简单的可以由管理人员使用的保留策略。 [API docs](https://zookeeper.apache.org/doc/r3.8.0/index.html)包含了关于调用规范（参数等）的细节。

在下面的例子中，最近的n个快照及其相关日志被保留而其他的被删除。n的值通常应大于3（虽然不是必须的，但这提供了3个备份，万一最近的日志被破坏了呢）。这可以作为ZooKeeper服务器机器上的一个crontab任务运行，每天进行日志清理。

~~~shell
java -cp zookeeper.jar:lib/slf4j-api-1.7.30.jar:lib/logback-classic-1.2.10.jar:lib/logback-core-1.2.10.jar:conf org.apache.zookeeper.server.PurgeTxnLog <dataDir> <snapDir> -n <count>
~~~

在3.4.0版本中，ZooKeeper引入了自动清理快照及其关联事务日志的自动清理功能，可以通过**autopurge.snapRetainCount**和**autopurge.purgeInterval**这两个参数进行配置。更多详细信息请参考下面的[高级配置]()。

#### 调试日志清理（logback）

请参考本文档中的[日志打印](###日志打印)一节。 预计可以使用内置的logback功能设置一个滚动文件appender。在发布的tar的conf/logback.xml中的样本配置文件提供了这方面的例子。

### 监控

每一个ZooKeeper进程（JVM）都需要有一个监控进程进行监控。ZK服务器被设计为 "快速故障"，这意味着如果发生无法恢复的错误，它将关闭（进程退出）。由于ZooKeeper服务集群是高度可靠的，这意味着当服务器可能关闭时，集群作为一个整体仍然是活跃的，并对外提供服务。此外，由于集群是 "自愈 "的，失败的服务器一旦重新启动就会自动重新加入集群，不需要任何手动操作。

通过[daemontools](http://cr.yp.to/daemontools.html)或者[SMF](http://en.wikipedia.org/wiki/Service_Management_Facility)等（仅仅作为例子）监控工具进行来确保ZooKeeper进程异常退出之后能够快速重启并且重新加入集群。

同时也建议将ZooKeeper服务器配置为发生OutOfMemoryError\*\*错误自动退出并打印堆栈。这是通过在Linux和Windows上分别使用以下参数启动 JVM 来实现的。*zkServer.sh*以及*zkServer.cmd*设置了该参数。

~~~shell
-XX:+HeapDumpOnOutOfMemoryError -XX:OnOutOfMemoryError='kill -9 %p'

"-XX:+HeapDumpOnOutOfMemoryError" "-XX:OnOutOfMemoryError=cmd /c taskkill /pid %%%%p /t /f"
~~~

### 监测

ZooKeeper进程可以通过三种主要方式监测：

- 向命令端口发送[4 letter words](####Four Letter Words)
- 通过[JMX](https://zookeeper.apache.org/doc/r3.8.0/zookeeperJMX.html)
- 通过[zkServer.sh status](https://zookeeper.apache.org/doc/r3.8.0/zookeeperTools.html#zkServer)命令

### 日志打印

ZooKeeper使用**[SLF4J](http://www.slf4j.org/)**1.7作为日志打印基础组件。ZooKeeper默认使用 **[LOGBack](http://logback.qos.ch/)**作为日志后端，但是使用者也可以选择其他支持的日志框架。

默认的*logback.xml*文件在*conf*目录中。 Logback要求*logback.xml*在工作目录（运行ZooKeeper的目录）中，或者可以从classpath访问。

关于SLF4J的更多信息，请参考[SLF4J手册](http://www.slf4j.org/manual.html)。

关于Logback的更多信息，请参考[Logback手册](http://logback.qos.ch/)。

### 故障排除

- *由于文件损坏导致的启动失败*：由于ZooKeeper服务器的事务日志中的一些文件损坏，服务器可能无法读取其数据库而无法启动。在加载ZooKeeper数据库时会看到一些IOException。在这种情况下，请确保集群中的所有其他服务器都已启动并工作。在命令端口上使用 "stat "命令来查看它们是否处于良好状态。在确认所有其他服务器都已启动后，可以清理损坏的服务器的数据库。删除datadir/version-2和datalogdir/version-2/中的所有文件。 重新启动服务器。

### 配置参数

ZooKeeper的行为是由配置文件控制的。在假定存储布局相同的前提下，这个文件的设计是为了让组成ZooKeeper服务器的所有服务器都能使用完全相同的文件。如果服务器使用不同的配置文件，必须注意确保所有不同的配置文件中的服务器列表相匹配。

> note
>
> 在3.5.0及之后的版本中，某些参数需要配置到动态配置文件中。如果这些参数被配置在了静态配置文件中，ZooKeeper会自动将其移入动态配置文件。更多信息请参考[Dynamic Reconfiguration](https://zookeeper.apache.org/doc/r3.8.0/zookeeperReconfig.html)。

#### 最小配置

以下是必须在配置文件中定义的最小关键配置：

- *clientPort*：监听客户端连接的端口；也就是客户端尝试连接的端口。

- *secureClientPort* ：监听客户端SSL安全连接；**clientPort**指定明文端口，**secureClientPort**指定SSL端口。指定两者可以启用混合模式，而省略其中一个将禁用该模式。注意，当用户启用zookeeper.serverCnxnFactory、zookeeper.clientCnxnSocket时，SSL功能将被启用（Netty）。

- *observerMasterPort* ：监听observer连接的端口；也就是观察者尝试连接端口。如果设置了该属性，则除了在leader模式下外，服务器还将在follower模式下承载observer连接，并在observer模式下相应地尝试连接到任何投票peer。

- *dataDir* ：ZooKeeper存放内存数据库快照的目录，除非指定其他目录，否则事务日志也存放在该目录下。

  > note
  >
  > 谨慎选择事务日志的存储位置。专用的事务日志设备是ZooKeeper持续高效运行的关键。日志存储到繁忙的设备会严重影响ZooKeeper性能。

- *<span id="tickTime">tickTime</span>*：毫秒计时的单一tick时长，ZooKeeper的基本时间单位。它被用来计量心跳和超时时间。例如，最小会话超时时间是两倍的tick。

#### 高级配置

本小节中的配置是可选的。可以通过这些配置微调ZooKeeper服务器的行为。有些也可以用Java系统属性来设置，一般是*zookeeper.keyword*的形式。如果有确切的系统属性的话，会在后面注明。

- *dataLogDir*：(No Java system property)该配置会将事务日志写入**dataLogDir**而不是**dataDir**。该选项允许专用的日志设备保存事务日志，避免事务日志与快照日志冲突。

  > note
  >
  > 专用的日志设备对吞吐量和稳定的延迟有很大影响。强烈建议使用专用的日志设备，并设置**dataLogDir**指向该设备上的一个目录，然后确保将**dataDir**指向一个不在该设备上的目录。

- *globalOutstandingLimit*：(Java system property: **zookeeper.globalOutstandingLimit**) 客户端提交请求的速度可以高于ZooKeeper的处理速度，特别是在客户端很多的情况下。为了避免由于排队的请求过多而耗尽内存，ZooKeeper会对客户端进行限流以确保系统中未决请求不会超过globalOutstandingLimit。默认的限制是1000。

- *preAllocSize*：(Java system property: **zookeeper.preAllocSize**)用于配置ZooKeeper事务日志文件预分配的磁盘空间大小（单位KB）。默认值为65536（64MB）。如果快照进行的比较频繁，可以减小该值。（参见**snapCount**与**snapSizeLimitInKb**。

- *snapCount*：(Java system property: **zookeeper.snapCount**)ZooKeeper使用快照和事务日志（可以认为是预写的日志）来记录它的事务。在快照（和滚动事务日志）之前，事务日志中记录的事务数量由snapCount决定。为了防止投票中的所有机器同时进行快照，每台ZooKeeper服务器将在事务日志中的事务数量达到[snapCount/2+1, snapCount]范围内运行时生成的随机值时进行快照。默认snapCount是100,000。

- *txnLogSizeLimitInKb*：(Java system property: **zookeeper.txnLogSizeLimitInKb**)Zookeeper事务日志文件也可以通过txnLogSizeLimitInKb更直接地控制。当使用事务日志进行同步时，较大的txn日志可能会导致follower的同步变慢。这是因为leader必须在磁盘上扫描相应的日志文件以找到要开始同步的事务。这个功能默认是关闭的，此时snapCount和snapSizeLimitInKb是唯一限制事务日志大小的选项。当启用时，Zookeeper会超过任何一个限制值的阈值时滚动日志。请注意，因为序列化事务的大小，实际的日志大小可能超过这个值。另一方面，如果这个值设置得太接近（或小于）preAllocSize，会导致Zookeeper为每个事务滚动日志。虽然这不是一个正确性问题，但这可能会导致严重的性能下降。为了避免这种情况，并充分利用这一功能，建议将该值设置为N***preAllocSize**，其中N >= 2。

- *maxCnxns*：(Java system property: **zookeeper.maxCnxns**)限制可以同时向ZooKeeper发起的客户端连接的数量（每台服务器的每个客户端口）。这是用来防备某些类型的拒绝服务攻击。默认值为0，设置为0意味着完全移除并发连接数量的限制。考虑到serverCnxnFactory和secureServerCnxnFactory的连接数的核算是单独进行的，因此允许一个peer最多托管2*maxCnxns，只要它们是适当的类型。

- *maxClientCnxns*：(No Java system property)在socket层面限制由IP地址区分的单个客户端可以对当个ZooKeeper集群发起的并发连接的数量。这是用来防止某些类别的DoS攻击，例如文件描述符耗尽。默认值为60。设置为0意味着完全不限制。

- *clientPortAddress*：**3.3.0版本新特性**：监听客户端连接的地址（ipv4，ipv6或者主机名）；也就是客户端尝试连接的地址。该选项是可选的，默认情况下以以下方式进行绑定：来自任何地址、接口、网卡的访问**clientPort**的连接都会被接受。

- *minSessionTimeout*： (No Java system property)：**3.3.0版本新特性**：服务端允许客户端进行协商的最小会话超时时间（毫秒）。默认值是2倍的**tickTime**。

- *maxSessionTimeout*： (No Java system property)：**3.3.0版本新特性**：服务端允许客户端进行协商的最大会话超时时间（毫秒）。默认值是20倍的**tickTime**。

- *fsync.warningthresholdms*： (Java system property: **zookeeper.fsync.warningthresholdms**)：**3.4.0版本新特性**：当事务日志（WAl）中的fsync耗时超过该值时，一条告警信息会被追加到日志中。默认值是1000毫秒。该选项仅可以通过系统属性设置。

- *maxResponseCacheSize*：(Java system property: **zookeeper.maxResponseCacheSize**)正值决定了缓存中以序列化形式存储的最近的读记录的数量。有助于减少常用znode使用的序列化成本。指标**response_packet_cache_hits**和**response_packet_cache_misses**可以用来根据给定的工作负载调整这个值。该选项的默认值是400。0或者负值关闭该选项。

- *maxGetChildrenResponseCacheSize*：(Java system property: **zookeeper.maxGetChildrenResponseCacheSize**)**3.6.0版本新特性**：类似于**maxResponseCacheSize**，但是适用于获取用户请求。指标**response_packet_get_children_cache_hits**和**response_packet_get_children_cache_misses**可以用来根据给定的工作负载调整这个值。该选项的默认值是400。0或者负值关闭该选项。

- *autopurge.snapRetainCount*： (No Java system property)**3.4.0版本新特性**：启用后，ZooKeeper自动清理功能会在**dataDir**和**dataLogDir**中分别保留autopurge.snapRetainCount个最近的快照和相应的事务日志，并删除其余的日志。默认值为3。最小值是3。

- *autopurge.purgeInterval*：(No Java system property)**3.4.0版本新特性**：清理功能调度的时间间隔（小时）。设置为正值（大于等于1）以启动自动清理。默认值是0。

- *syncEnabled*：(Java system property: **zookeeper.observer.syncEnabled**)**3.4.6,3.5.0版本新特性**：observer现在像参与者一样默认记录事务并将快照写到磁盘。该选项减少了observer重启时的恢复时间。设置为”false“关闭该选项。默认值为”true“。

- *extendedTypesEnabled*：(Java system property only: **zookeeper.extendedTypesEnabled**)**3.5.4,3.6.0版本新特性**：设置为true以启用扩展特性，例如[TTL Nodes](https://zookeeper.apache.org/doc/r3.8.0/zookeeperProgrammers.html#TTL+Nodes)。这些特性默认为不启用。重要：由于内部限制，该选项启用时服务器ID必须小于255。

- *emulate353TTLNodes*：(Java system property only:**zookeeper.emulate353TTLNodes**)**3.5.4,3.6.0版本新特性**：由于[ZOOKEEPER-2901](https://issues.apache.org/jira/browse/ZOOKEEPER-2901)的原因，3.5.3中添加的TTL Nodes在3.5.4/3.6.0中不被支持。然而，通过zookeeper.emulate353TTLNodes这一系统属性提供了变通。如果使用了ZooKeeper3.5.3中的TTL Nodes的同时还想保持兼容性，那么除了启用**zookeeper.extendedTypesEnabled**以外，还要将 **zookeeper.emulate353TTLNodes**设置为true。注意：由于这个bug，服务器ID必须小于127。此外最大的受支持的TTL值是1099511627775，比3.5.3中的允许值（1152921504606846975）要小。

- *watchManagerName*：(Java system property only: **zookeeper.watchManagerName**)**3.6.0版本新特性**： [ZOOKEEPER-1179](https://issues.apache.org/jira/browse/ZOOKEEPER-1179)中添加。增加了新的watcher管理器WatchManagerOptimized以优化重度watch使用情况下的内存开销。该配置项用来决定使用哪种watcher管理器。目前，ZooKeeper仅支持WatchManager和WatchManagerOptimized。

- *watcherCleanThreadsNum*： (Java system property only: **zookeeper.watcherCleanThreadsNum**) **3.6.0版本新特性**：[ZOOKEEPER-1179](https://issues.apache.org/jira/browse/ZOOKEEPER-1179)中添加。新的watcher管理器WatchManagerOptimized将懒惰地清理无效的watcher，这个配置用于决定在WatcherCleaner中使用多少个线程。更多的线程通常意味着更大的清理吞吐量。默认值是2，即使在大量连续的会话关闭/重新创建的情况下，这也足够了。

- *watcherCleanIntervalInSeconds*：(Java system property only:**zookeeper.watcherCleanIntervalInSeconds**)**3.6.0版本新特性**：[ZOOKEEPER-1179](https://issues.apache.org/jira/browse/ZOOKEEPER-1179)中添加。这是用来控制在WatcherCleaner中待处理watcher的阈值，当达到阈值时，将减缓向WatcherCleaner添加失效的watcher，这又将减缓添加和关闭watcher，这样就可以避免OOM问题。默认是无限制，可以将改值设置为watcherCleanThreshold * 1000。

- *bitHashCacheSize*：(Java system property only: **zookeeper.bitHashCacheSize**)**3.6.0版本新特性**：[ZOOKEEPER-1179](https://issues.apache.org/jira/browse/ZOOKEEPER-1179)中添加。这是用于决定BitHashSet实现中HashSet缓存大小的设置。如果没有HashSet，我们需要用O(N)时间来获取元素，N是elementBits中的位数。但是需要缓存保持较小的大小，以确保它不会花费太多的内存，在内存和时间复杂度之间有一个权衡。默认值是10，这似乎是一个相对合理的缓冲区大小。

- *fastleader.minNotificationInterval*：(Java system property: **zookeeper.fastleader.minNotificationInterval**)：leader选举的两次连续通知检查之间的时间长度下限。这个时间间隔决定了peer等待检查选举投票集的时间，并影响选举的完成速度。该时间间隔遵循从配置的最小值（本配置值）到配置的最大值（fastleader.maxNotificationInterval）的退避策略，用于长时间的选举。

- *fastleader.maxNotificationInterval*：(Java system property: **zookeeper.fastleader.maxNotificationInterval**)：leader选举的两次连续通知检查之间的时间长度下限。这个时间间隔决定了peer等待检查选举投票集的时间，并影响选举的完成速度。该时间间隔遵循从配置的最小值（fastleader.minNotificationInterval）到配置的最大值（本配置值）的退避策略，用于长时间的选举。

- *connectionMaxTokens*：(Java system property: **zookeeper.connection_throttle_tokens**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了令牌桶中最大的令牌数。当设置为0时，限流功能被禁用。默认值是0。

- *connectionTokenFillTime*： (Java system property: **zookeeper.connection_throttle_fill_time**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了以*connectionTokenFillCount*数量的令牌对令牌桶进行重新装填的时间间隔（毫秒）。默认值是1。

- *connectionTokenFillCount* ： (Java system property: **zookeeper.connection_throttle_fill_count**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了每经过*connectionTokenFillTime*毫秒对令牌桶进行装填时装入令牌的数量。默认值是1。

- *connectionFreezeTime* ： (Java system property: **zookeeper.connection_throttle_freeze_time**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了丢弃概率调整的时间间隔（毫秒）。该值设置为-1意味着不启用概率性丢弃。默认值是-1。

- *connectionDropIncrease* ：(Java system property: **zookeeper.connection_throttle_drop_increase**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了丢弃概率的增长量。限流器每connectionFreezeTime毫秒检查一次，如果令牌桶是空的，丢弃概率就会增长connectionDropIncrease。默认值是0.02。

- *connectionDropDecrease*： (Java system property: **zookeeper.connection_throttle_drop_decrease**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了丢弃概率的减小量。限流器每connectionFreezeTime毫秒检查一次，如果令牌桶中令牌的数量超过阈值，丢弃概率就会降低connectionDropDecrease。阈值是*connectionMaxTokens* * *connectionDecreaseRatio*。默认值是0.002。

- *connectionDecreaseRatio* ：(Java system property: **zookeeper.connection_throttle_decrease_ratio**)**3.6.0版本新特性**：这是调整服务器端连接限流器的参数之一，采用了基于令牌的、具有选择性概率丢弃机制的速率限制机制。该参数定义了降低丢弃概率的阈值。默认值是0。

- *zookeeper.connection_throttle_weight_enabled* ： (Java system property only)**3.6.0版本新特性**：限流时是否考虑连接权重。只有连接限流启用（connectionMaxTokens大于0）时才有效。默认值是false。

- *zookeeper.connection_throttle_global_session_weight*：(Java system property only) **3.6.0版本新特性**： 全局会话的权重。它是一个全局会话请求通过连接限流器所需的令牌数量。它必须是一个正整数，不小于本地会话的权重。默认值是3。

- *zookeeper.connection_throttle_local_session_weight*：(Java system property only) **3.6.0版本新特性**：本地会话的权重。它是一个本地会话请求通过连接限流器所需的令牌数量。它必须是一个正整数，不大于全局会话或更新会话的权重。默认值是1。

- *zookeeper.connection_throttle_renew_session_weight* ： (Java system property only) **3.6.0版本新特性**：更新会话的权重。是一个重新连接请求通过限流器所需的令牌数量。必须是一个正整数，不能小于本地会话的权重。默认是2。

- *clientPortListenBacklog* ： (Java system property only)**3.4.14, 3.5.5, 3.6.0版本新特性**：ZooKeeper服务器积压的套接字的数量。该选项控制了服务器端队列中待处理的请求数量。超过该数量的连接请求会收到网络超时（30s）并导致ZooKeeper会话超时问题。该选项默认不设置（-1），在Linux上的队列长度是50。该选项必须是一个正数。

- *serverCnxnFactory*：(Java system property: **zookeeper.serverCnxnFactory**)指定ServerCnxnFactory的具体实现。基于TLS的通信需要指定NettyServerCnxnFactory。默认值是NIOServerCnxnFactory。

- *flushDelay*：(Java system property: **zookeeper.flushDelay**)延迟提交日志的flush时间（毫秒）。不会影响*maxBatchSize*的限制。默认不启用（0）。具有高写入率的集群可能会看到吞吐量在10-20毫秒的数值下得到改善。

- *maxWriteQueuePollTime*：(Java system property: **zookeeper.maxWriteQueuePollTime**)如果flushDelay被启用，该选项决定了当没有新的请求排队时，在flush前要等待的时间（毫秒）。默认值为*flushDelay*/3（默认禁用）。

- *maxBatchSize*：(Java system property: **zookeeper.maxBatchSize**)在已提交日志flush之前，服务器中允许的事务数量。不会影响*flushDelay*的限制。默认值是1000。

- *enforceQuota*：(Java system property: **zookeeper.enforceQuota**)**3.7.0版本新特性**：强制执行配额检查。启用后，如果客户端超过了znode下的总字节数或者子节点数的硬配额，服务器将拒绝该请求，并强制向客户端返回QuotaExceededException。默认值为false。详细信息请参考[quota feature](http://zookeeper.apache.org/doc/current/zookeeperQuotas.html)。

- *requestThrottleLimit* ：(Java system property: **zookeeper.request_throttle_max_requests**)**3.6.0版本新特性**：请求限流器开始暂停之前允许的未完成的请求总数。0意味着限流不启用。默认值是0。

- *requestThrottleStallTime* ： (Java system property: **zookeeper.request_throttle_stall_time**)**3.6.0版本新特性**：一个线程等待被通知可以继续处理一个请求的最大时间（毫秒）。

- *requestThrottleDropStale*：(Java system property: **request_throttle_drop_stale**)**3.6.0版本新特性**：如果启用，限流器会将过期请求丢弃而不是放入请求管道。过期请求是由一个已经关闭的连接发出的请求，和/或一个请求的延迟将高于sessionTimeout的请求。默认值是true。

- *requestStaleLatencyCheck*：(Java system property: **zookeeper.request_stale_latency_check**)**3.6.0版本新特性**：如果启用，当一个请求的请求延迟超过其会话的超时时间时，该请求会被看作过期请求。默认不启用。

- *requestStaleConnectionCheck* ：(Java system property: **zookeeper.request_stale_connection_check**)**3.6.0版本新特性**：如果启用，当一个请求的连接被关闭时，该请求会被看作过期请求。默认启用。

- *zookeeper.request_throttler.shutdownTimeou*：(Java system property only)**3.6.0版本新特性**：在请求限流器关闭期间等待请求队列耗尽的时间（毫秒），在此之后请求限流器会被强制关闭。默认值是10000。

- *advancedFlowControlEnabled* ：(Java system property: **zookeeper.netty.advancedFlowControl.enabled**)在netty中使用基于ZooKeeper管道状态的精确流量控制，以避免直接缓冲区OOM。该选项会禁用Netty中的AUTO_READ。

- *enableEagerACLCheck*：(Java system property only: **zookeeper.enableEagerACLCheck**)如果设置为true，在请求发送到quorum之前，对每台本地服务器上的写请求启用快速ACL检查。默认值为false。

- *maxConcurrentSnapSyncs*：(Java system property: **zookeeper.leader.maxConcurrentSnapSyncs**)一个leader或者follower可以同时同步的最大快照数量。默认值是10。

- *maxConcurrentDiffSyncs*：(Java system property: **zookeeper.leader.maxConcurrentDiffSyncs**)一个leader或者follower可以同时进行的最大差异同步数量。默认值是100。

- *digest.enabled*：(Java system property only: **zookeeper.digest.enabled**)**3.6.0版本新特性**：添加摘要功能是为了从磁盘加载数据库以及与leader同步时，从ZooKeeper内部检测数据的不一致性，其基于adHash论文中提到的DataTree做增量哈希检查

  ```shell
  https://cseweb.ucsd.edu/~daniele/papers/IncHash.pdf
  ```

  该方法很简单，DataTree的哈希值将根据数据集的变化而逐步更新。leader在准备txn时会通过公式根据发生的变化预先计算树的哈希值。

  ```shell
  current_hash = current_hash + hash(new node data) - hash(old node data)
  ```

  如果正在创建新节点，hash(old node data)是0，如果是一个删除操作，hash(new node data)是0。

  这个哈希值将与每个txn相关联，代表将txn应用于数据树后的预期哈希值，它将与原始提案一起被发送给follower。learner将比较实际的哈希值和应用txn到数据树后txn中的哈希值，如果不一样就报告不匹配。

  这些摘要值也将与磁盘上的每个txn和快照一起持续保存，因此当服务器重新启动并从磁盘加载数据时，它将比较并检查是否存在哈希值不匹配，这将有助于检测磁盘上的数据丢失问题。

  对于实际的哈希函数，我们在内部使用了CRC，它不是一个无碰撞的哈希函数，但与无碰撞哈希相比，它的效率更高，而且碰撞的可能性真的非常少，已经可以满足需求。

  该特性是后向和前向兼容的，所以可以安全地滚动升级、降级、启用以及后续禁用，没有任何兼容问题。以下是已经涵盖和测试的场景：

  1. leader运行新代码follower运行旧代码，摘要会被追加到txn之后，follower仅读取txn头部和数据，摘要数据会被忽略。摘要不会影响follower读取并处理下一个txn。
  2. leader运行旧代码follower运行新代码，摘要不会通过txn发送，当follower尝试读取摘要时会抛出EOF异常，通过将摘要值设为null该异常被捕获并且妥善处理。
  3. 使用新代码加载旧快照，当试图读取不存在的摘要值时，将抛出IOException，该异常将被捕获，摘要将被设置为null，这意味着在加载这个快照时不会比较摘要，这在滚动升级中预计会发生。
  4. 当用旧代码加载新快照时，在反序列化数据树后加载将成功完成，快照文件末尾的摘要值将被忽略。
  5. 在标志变化的情况下，滚动重启的情况与上面讨论的第1和第2种情况类似，如果leader启用但follower没有，摘要值将被忽略，follower在运行期间不会比较摘要；如果leader禁用但follower启用，follower将获得EOF异常并妥善处理。

  注意： 由于/zookeeper/quota统计节点的潜在不一致性，目前的摘要计算排除了/zookeeper下的节点，在这个问题被解决后可以将其包括在内。

  该特性默认启用，设置为false将其禁用。

- *snapshot.compression.method*：(Java system property: **zookeeper.snapshot.compression.method**)**3.6.0版本新特性**：该特性决定了ZooKeeper在将快照日志写入硬盘时是否进行压缩（参考[ZOOKEEPER-3179](https://issues.apache.org/jira/browse/ZOOKEEPER-3179)）。可选值如下：

  - ""：不启用（不对快照进行压缩）。这是默认行为。
  - "gz"：参考[gzip compression](https://en.wikipedia.org/wiki/Gzip)。
  - "snappy"：参考[Snappy compression](https://en.wikipedia.org/wiki/Snappy_(compression))。

- *snapshot.trust.empty*：(Java system property: **zookeeper.snapshot.trust.empty**)**3.5.6版本新特性**：该特性决定了ZooKeeper是否将快照丢失作为不可恢复的灾难性状态。设置为true允许ZooKeeper服务器在没有快照文件的情况下恢复。这只应在从旧版本的ZooKeeper（3.4.x，3.5.3之前）升级时设置，因为ZooKeeper可能只有事务日志文件，而没有快照文件。如果该值是在升级过程中设置的，我们建议在升级后将该值设回false，并重新启动ZooKeeper进程，这样ZooKeeper就可以在恢复过程中继续进行正常的数据一致性检查。默认值为false。

- *audit.enable*：(Java system property: **zookeeper.audit.enable**)**3.6.0版本新特性**：默认不启用审计日志。设置为true以启用审计日志。默认值是false。请参考[ZooKeeper audit logs](https://zookeeper.apache.org/doc/r3.8.0/zookeeperAuditLogs.html)获取更多信息。

- *audit.impl.class*：(Java system property: **zookeeper.audit.impl.class**)**3.6.0版本新特性**：审计日志的具体实现。默认使用org.apache.zookeeper.audit .Slf4jAuditLogger。请参考[ZooKeeper audit logs](https://zookeeper.apache.org/doc/r3.8.0/zookeeperAuditLogs.html)获取更多信息。

- *largeRequestMaxBytes*：(Java system property: **zookeeper.largeRequestMaxBytes**)**3.6.0版本新特性**：所有处理中的大型请求的最大字节数。如果一个新到达的大型请求导致该阈值被超过，连接会被关闭。默认值是100\*1024\*1024（100MB）。

- *largeRequestThreshold*：(Java system property: **zookeeper.largeRequestThreshold**)**3.6.0版本新特性**：一个请求被认为是大型请求的大小阈值。如果设置为-1，所有请求都被认为是小型的，这关闭了大型请求限流。默认值是-1。

- *outstandingHandshake.limit*：(Java system property only: **zookeeper.netty.server.outstandingHandshake.limit**)ZooKeeper可以处于处理中的TLS握手的阈值，超过该阈值的连接会在握手开始前被拒绝。这个设置并不限制TLS的最大并发量，但有助于避免由于TLS握手超时而产生的羊群效应。当有太多的TLS握手时，设置为250就足以避免羊群效应。

- *netty.server.earlyDropSecureConnectionHandshakes*：(Java system property: **zookeeper.netty.server.earlyDropSecureConnectionHandshakes**)如果ZooKeeper没有完全启动，在开始TLS握手之前丢弃TCP连接。该选项对于避免服务器重启之后大量并发TLS握手的导致的洪泛是非常有帮助的。请注意如果启用该标志，在服务器完全启动之前其不会对‘ruok’命令做出回应。丢弃连接的行为已经在3.7.0版本中引入并且不能禁用。从3.7.1及3.8.0开始该特性默认被禁用。

- *throttledOpWaitTime*：(Java system property: **zookeeper.throttled_op_wait_time**)请求限流器中的时间，超过这个时间的请求将被标记为已限流。一个被限流的请求将不会被处理，只是被送入它所属的服务器的管道，以保持所有请求的顺序。对于这些被限流的请求，FinalProcessor会发出一个错误应答（新错误码：ZTHROTTLEDOP）。潜在含义是告诉客户端不要立即重试。如果设置为0，请求不会被标记为已限流。默认值是0。

- *learner.closeSocketAsync*：(Java system property: **zookeeper.learner.closeSocketAsync**) (Java system property: **learner.closeSocketAsync**)(为了后向兼容而增加)**3.7.0版本新特性**：如果启用，learner会异步关闭quorum socket。这对TLS连接非常有用，在这种情况下，关闭一个套接字可能需要很长的时间，阻塞shutdown进程，可能会延迟新的leader选举，并使quorum不可用。尽管套接字关闭时间很长而且在套接字关闭的同时可以开始新的leader选举，异步关闭套接字可以避免阻塞shutdown进程。默认是false。

- *leader.closeSocketAsync*：(Java system property: **zookeeper.leader.closeSocketAsync**) (Java system property: **leader.closeSocketAsync**)(为了后向兼容而增加)**3.7.0版本新特性**：如果启用，leader会异步关闭quorum socket。这对TLS连接非常有用，在这种情况下，关闭一个套接字可能需要很长的时间。如果因为SyncLimitCheck而在ping()中开始断开与follower的连接，那么套接字的长时间关闭将阻止向其他follower发送ping。其他follower收不到ping将不会向leader发送会话信息，这将导致会话超时。设置该标志为true以确保ping能正常发送。默认值是false。

- *learner.asyncSending*：(Java system property: **zookeeper.learner.asyncSending**) (Java system property: **learner.asyncSending**)(为了后向兼容而增加)**3.7.0版本新特性**：在learner中，发送和接受数据包是在一个关键的部分中同步进行的。突发的网络问题会导致follower hang住（参见[ZOOKEEPER-3575](https://issues.apache.org/jira/browse/ZOOKEEPER-3575)以及[ZOOKEEPER-4074](https://issues.apache.org/jira/browse/ZOOKEEPER-4074)）。在学习者中，新的设计将发送数据包的工作放到了单独的线程中并且异步发送。通过该参数(learner.asyncSending)启用新设计。默认值是false。

- *forward_learner_requests_to_commit_processor_disabled*：(Java system property: **zookeeper.forward_learner_requests_to_commit_processor_disabled**)如果该特性启用，来自learner的请求不会被放到提交处理器队列中，这有助于减少leader的资源消耗和GC时间。默认值是false。

#### 集群选项

本小节中的选项是为了集群使用而设计的——也就是说，以集群的方式部署服务器。

- *electionAlg*：(No Java system property)要使用的选举的具体实现。“1”代表基于UDP的、非认证版本的快速leader选举，“2”代表基于UDP的、认证版本的快速leader选举，“3”代表基于TCP版本的快速leader选举。3.2.0版本默认使用算法3，之前的版本（3.0.0和3.0.1）同时使用算法1和算法2。

  > note
  >
  > leader选举1和2的实现在3.4.0中被废弃。从3.6.0开始只有FastLeaderElection是可用的，如果要升级，必须将所有服务器重启并且将electionAlg设置为3（或者从配置文件中将该配置移除）。

- *maxTimeToWaitForEpoch*：(Java system property: **zookeeper.leader.maxTimeToWaitForEpoch**)**3.6.0版本新特性**：激活leader时，等待其他投票者的epoch的最大等待时间。如果leader从其投票者收到LOOKING通知并且在maxTimeToWaitForEpoch时间内没有从大多数服务器收到epoch包，那么该leader会进入LOOKING状态并且重新进行leader选举。调整该值可以减少quorum人数或者服务器不可用时间，可以将该值设置为一个比initLimit\*tickTime小得多的时间。在跨数据中心环境中，可以将该值设置为2s。

- *initLimit*：(No Java system property)follower连接并且与leader进行同步的总的tick数量（参考[tickTime](#tickTime)）。如果ZooKeeper管理的数据量非常大，酌情增加该值。

- *connectToLearnerMasterLimit*：(Java system property: zookeeper.**connectToLearnerMasterLimit**)在选举leader之后，允许follower连接到leader的总的tick数量（参考[tickTime](#tickTime)）。在initLimit较高时使用该配置，这样连接到learner master就不会导致更高的超时。

- *leaderServes*：(Java system property: zookeeper.**leaderServes**)leader接受客户端连接。默认值是“yes”。leader协调更新。为了获得更高的更新吞吐量，在略微牺牲读取性能的情况下，leader可以被配置为不接受客户端连接而专注于协调。这个选项的默认值是“yes”，这意味着leader可以接受客户端连接。

  > **Note**
  >
  > 如果集群中的ZooKeeper服务器数量超过三台，那么打开leader selection是非常建议的。

- *server.x=[hostname]:nnnnn[:nnnnn] 等*：(No Java system property)Zookeeper集群是由服务器组成的。当启动时，服务器通过查找data目录下的*myid*文件来决定它是那一台。该文件包含ASCII编码的服务器编号，它应该与本设置左侧的**server.x**中的**x**匹配。客户端使用的构成ZooKeeper服务器的列表必须与每个ZooKeeper服务器所拥有的ZooKeeper服务器列表相匹配。配置中有两个端口号**nnnnn**。follower通过第一个端口连接leader，第二个端口用于leader选举。如果要在单台机器上进行多服务器测试，那么每个服务可以指定不同端口。从ZooKeeper3.6.0开始，对于每台ZooKeeper服务器可以指定**<span id="madresses">多地址</span>**（参考[ZOOKEEPER-3188](https://issues.apache.org/jira/projects/ZOOKEEPER/issues/ZOOKEEPER-3188)）。启用该特性必须将配置属性*multiAddress.enabled*设置为*true*。该特性增加了ZooKeeper的可用性和网络层面的恢复能力。当服务器使用了多个物理网络接口时，ZooKeeper能够绑定到所有接口上并且在发生网络故障的情况下运行时切换到一个工作的接口。在配置文件中可以使用管道符（'|'）指定不同的地址。如下是一个有效的多地址配置：

  ~~~shell
  server.1=zoo1-net1:2888:3888|zoo1-net2:2889:3889
  server.2=zoo2-net1:2888:3888|zoo2-net2:2889:3889
  server.3=zoo3-net1:2888:3888|zoo3-net2:2889:3889
  ~~~

  > **Note**
  >
  > 通过启用该特性，Quorum协议（ZooKeeper的Server-Server协议）会改变。这一点对用户是透明的，当用新的配置启动ZooKeeper集群时，一切都会正常工作。然而，如果旧的ZooKeeper集群不支持多地址功能（以及新的Quorum协议），就不可能在滚动升级中启用这个功能并指定多个地址。如果需要这个功能，但又需要从3.6.0以上的ZooKeeper集群进行滚动升级，那么首先需要在不启用多地址功能的情况下进行滚动升级，然后用新的配置进行单独的滚动重启，其中**multiAddress.enabled**设置为**true**并提供多个地址。

- *syncLimit*：(No Java system property)允许follower与leader进行同步的最大tick数量（参考[tickTime](#tickTime)）。如果follower落后leader太远，该follower会被抛弃。

- *group.x=nnnnn[:nnnnn]*：(No Java system property)启用分层的quorum构建。"x "是一个组的标识符，"="号右面的数字对应于服务器标识符。式子的右边是一个用冒号分隔的服务器标识符列表。请注意，组必须是不相交的，所有组的联合必须是ZooKeeper的集合。[这里](https://zookeeper.apache.org/doc/r3.8.0/zookeeperHierarchicalQuorums.html)是一个例子。

- *weight.x=nnnnn*：(No Java system property)与“group”同时使用，当构建quorum时，该配置为每台服务器指定一个权重。配置的数值对应于投票时服务器的权重。ZooKeeper有几个部分需要投票，如leader选举和原子广播协议。服务器的默认权重是1。如果配置中定义了组但是没定义权重，那么1会被分配给每台服务器。[这里](https://zookeeper.apache.org/doc/r3.8.0/zookeeperHierarchicalQuorums.html)是一个例子。

- *cnxTimeout*：(Java system property: zookeeper.**cnxTimeout**)leader选举通知的连接建立超时时间。仅当electionAlg设置为3时有效。

  > **Note**
  >
  > 默认值是5s。

- *quorumCnxnTimeoutMs*：(Java system property: zookeeper.**quorumCnxnTimeoutMs**)leader选举通知的连接读超时时间。仅当electionAlg设置为3时有效。

  > **Note**
  >
  > 默认值是-1，此时使用syncLimit * tickTime作为超时时间。

- *standaloneEnabled*：(No Java system property)**3.5.0版本新特性**：当设置为true时，单个服务器可以在复制模式下启动，一个单独的参与者可以与observer一起运行，集群可以重新配置到一个节点，并从一个节点启动。为了向后兼容，默认为true。它可以通过QuorumPeerConfig的setStandaloneEnabled方法或者在服务器的配置文件中加入 ”standaloneEnabled=false“或 ”standaloneEnabled=true“来设置。

- *reconfigEnabled*：(No Java system property)**3.5.3版本新特性**：该配置控制是否启用[Dynamic Reconfiguration](https://zookeeper.apache.org/doc/r3.8.0/zookeeperReconfig.html)特性。启用该功能后，在被授权进行重新配置的前提下，用户可以通过ZooKeeper客户端API或通过ZooKeeper命令行工具进行重新配置操作。禁用该功能后包括超级用户在内的所有用户都不能进行重配置。任何重配置的尝试都会返回一个错误。“**reconfigEnabled**”选项在配置文件中可以被设置为“**reconfigEnabled=false**”或者“**reconfigEnabled=true**”，或者通过QuorumPeerConfig的setReconfigEnabled方法进行设置。默认值是false。如果配置的话，集群中的每台服务器应该保持相同配置。在某些服务器上设置为true，而在其他服务器上设置为false，将导致不一致的行为，这取决于哪个服务器被选为leader。如果leader设置了“**reconfigEnabled=true**”，那么整个集群的重配置特性会被启用。如果leader设置了“**reconfigEnabled=false**”，那么整个集群的重配置特性会被禁用。因此建议在集群中的每台服务器上保持“**reconfigEnabled**”配置一致。

- *4lw.commands.whitelist*：(Java system property: **zookeeper.4lw.commands.whitelist**)**3.5.3版本新特性**：用户可以使用的逗号分隔的[Four Letter Words](https://zookeeper.apache.org/doc/r3.8.0/zookeeperAdmin.html#sc_4lw)。 有效的Four Letter Words命令必须放在这个列表中，否则ZooKeeper服务器不会启用该命令。默认情况下，白名单只包含zkServer.sh使用的”srvr“命令。其余的Four Letter Words命令默认是禁用的：试图使用这些命令会得到一个回应“....，因为它不在白名单中，所以没有执行”。下面是一个配置的例子，它启用了stat、ruok、conf和isro命令，同时禁用了其余的Four Letter Words：

  ~~~shell
  4lw.commands.whitelist=stat, ruok, conf, isro
  ~~~

  如果需要默认启用所有Four Letter Words命令，可以使用星号选项，这样就不必在列表中逐一包括每个命令。下面的例子将启用所有Four Letter Words的命令：

  ~~~shell
  4lw.commands.whitelist=*
  ~~~

- *tcpKeepAlive*：(Java system property: **zookeeper.tcpKeepAlive**)**3.5.4版本新特性**：设为true在quorum成员的用来进行选举的socket上启用TCP keepAlive标志。这是quorum成员之间的连接在可能被网络基础设施中断的情况下保持正常。长期或者空闲的连接可能会被某些NAT设备和防火墙终止或者丢失。启用该选项需要依靠操作系统级别的设置才能正常工作，请检查操作系统关于TCP keepalive的选项以了解更多信息。默认值是false。

- *clientTcpKeepAlive*：(Java system property: **zookeeper.clientTcpKeepAlive**)**3.6.1版本新特性**：设置为true将在客户端socket上设置TCP keepAlive标志。一些损坏的网络基础设施可能会丢失从正在关闭的客户端发送的FIN数据包。这些从不关闭的客户端socket会导致OS资源泄露。启用该选项通过空闲检查终止僵尸socket。启用该选项需要依靠操作系统级别的设置才能正常工作，请检查操作系统关于TCP keepalive的选项以了解更多信息。默认值是false。请注意该选项与**tcpKeepAlive**的差别。本选项作用于客户端socket而**tcpKeepAlive**作用于quorum成员socket。当前该选项仅在使用默认NIOServerCnxnFactory时是可用的。

- *electionPortBindRetry*：(Java system property only: **zookeeper.electionPortBindRetry**)ZooKeeper绑定leader选举端口失败时的最大重试次数。此类错误可能是临时的和可恢复的，例如[ZOOKEEPER-3320](https://issues.apache.org/jira/projects/ZOOKEEPER/issues/ZOOKEEPER-3320)中描述的DNS问题，也可能是不可重试的，例如端口已在使用中。在出现暂时错误的情况下，此属性可以提高Zookeeper服务器的可用性，并帮助它进行自我恢复。默认值是3。在容器环境中，特别是在Kubernetes中，应该增加此值或将其设置为0（无限重试）以克服与DNS名称解析相关的问题。

- *observer.election.DelayMs*(Java system property: **zookeeper.observer.election.DelayMs**)断开连接后，延迟observer参与leader选举，以防止在选举过程中给投票peer带来意外的额外负担。默认为200毫秒。

- *localSessionsEnabled*和*localSessionsUpgradingEnabled*：**3.5版本新特性**：可选值是ture或false。默认值是false。通过设置*localSessionsEnabled=true*开启本地会话特性。在*localSessionsEnabled*启用时，开启*localSessionsUpgradingEnabled*可以在需要时（例如创建临时节点）自动将本地会话升级为全局会话。

#### 加密、身份验证、授权选项

本小节中的选项允许控制服务执行的加密/身份验证/授权。

除此之外，还可以在[Programmers Guide](https://zookeeper.apache.org/doc/r3.8.0/zookeeperProgrammers.html#sc_java_client_configuration)中找到有关客户端配置的有用信息。ZooKeeper Wiki还提供了一些有用的页面，介绍[ZooKeeper SSL support](https://cwiki.apache.org/confluence/display/ZOOKEEPER/ZooKeeper+SSL+User+Guide)和[SASL authentication for ZooKeeper](https://cwiki.apache.org/confluence/display/ZOOKEEPER/ZooKeeper+and+SASL)。

- *DigestAuthenticationProvider.enabled*：(Java system property: **zookeeper.DigestAuthenticationProvider.enabled**)**3.7版本新特性**：确定是否启用摘要身份验证提供程序。为了后向兼容所以默认值是true，但是如果没有使用该提供程序最好将其禁用，因为它可能会导致审计日志中出现误导性条目（请参考[ZOOKEEPER-3979](https://issues.apache.org/jira/browse/ZOOKEEPER-3979)）。

- *DigestAuthenticationProvider.superDigest*：(Java system property: **zookeeper.DigestAuthenticationProvider.superDigest**)**3.2版本新特性**：该特性默认不启用。允许ZooKeeper集群管理员以“超级”用户身份访问znode层次结构。特别是，对于作为超级用户进行身份验证的用户，不会进行ACL检查。org.apache.zookeeper.server.auth。DigestAuthenticationProvider可用于生成superDigest，使用一个参数“super:”调用它。在启动集合的每个服务器时，提供生成的“super:”作为系统属性值。当向ZooKeeper服务器进行身份验证时（从ZooKeoper客户端），传递“摘要”方案和“super:”身份验证数据。请注意，摘要身份验证以明文形式将身份验证数据传递给服务器，谨慎的做法是仅在本地主机上（而不是通过网络）或通过加密连接使用此身份验证方法。

- *DigestAuthenticationProvider.digestAlg*：(Java system property: **zookeeper.DigestAuthenticationProvider.digestAlg**)**3.7版本新特性**：设置ACL摘要算法。默认值是：SHA1，该算法即将因安全问题而被废弃。该属性在所有服务器中要保持相同的值。

  - 如何支持其他算法？

    - 修改 `$JAVA_HOME/jre/lib/security/java.security`下的配置文件`java.security`，指定： `security.provider.<n>=<provider class name>`。

      ~~~shell
      For example:
      set zookeeper.DigestAuthenticationProvider.digestAlg=RipeMD160
      security.provider.3=org.bouncycastle.jce.provider.BouncyCastleProvider
      ~~~

    - 拷贝jar文件到`$JAVA_HOME/jre/lib/ext/`。

      ~~~shell
      For example:
      copy bcprov-jdk15on-1.60.jar to $JAVA_HOME/jre/lib/ext/
      ~~~

  - 如何从一种算法迁移到另一种算法？

    - 1.当迁移新算法时重新生成 `superDigest`。
    - 1.对旧算法进行摘要身份验证的znode的调用SetAcl。

- *X509AuthenticationProvider.superUser*： (Java system property: **zookeeper.X509AuthenticationProvider.superUser**)使ZooKeeper集群管理员能够作为“超级”用户访问znode层次结构的支持SSL的方式。当此参数设置为X500主体名称时，只有具有该主体的经过身份验证的客户端才能绕过ACL检查，并对所有znode具有完全权限。

- *zookeeper.superUser*：(Java system property: **zookeeper.superUser**)类似于**zookeeper.X509AuthenticationProvider.superUser**，但对于基于SASL的登陆是通用的。它存储可以作为“超级”用户访问znode层次结构的用户的名称。可以使用 **zookeeper.superUser.[suffix]**指定多个SASL超级用户，例如：`zookeeper.superUser.1=...`。

- *ssl.authProvider*：(Java system property: **zookeeper.ssl.authProvider**)指定**org.apache.zookeeper.auth**的子类用于安全客户端身份验证的X509AuthenticationProvider。这在不使用JKS的证书密钥基础结构中很有用。可能需要扩展**javax.net.ssl.X509KeyManager**和**javax.net.ssl.X509TrustManager**从SSL协议栈获取所需的行为。要将ZooKeeper服务器配置为使用自定义提供程序进行身份验证，请为自定义AuthenticationProvider选择scheme名称，并设置属性**zookeeper.authProvider.[scheme]**为自定义实现的完全限定类名。这将把提供程序加载到ProviderRegistry中。然后设置属性**zookeeper.ssl.authProvider=[scheme]**，该提供程序将用于安全身份验证。

- *zookeeper.ensembleAuthName*：(Java system property only: **zookeeper.ensembleAuthName**)**3.6.0版本新特性**：指定逗号分隔的有效名称/别名列表。客户端可以提供其尝试连接的集群名称作为scheme“集群”的凭据。EnsembleAuthenticationProvider将根据接收连接请求的集合的名称/别名列表检查凭据。如果凭据不在列表中，则连接请求将被拒绝。这可以防止客户端意外连接到错误的集群。

- *sessionRequireClientSASLAuth*：(Java system property: **zookeeper.sessionRequireClientSASLAuth**)**3.6.0版本新特性**：如果设置为true，ZooKeeper服务器将仅接受通过SASL身份验证的客户端的连接和请求。未配置SASL身份验证或配置了SASL但身份验证失败的客户端（例如凭证失效）不能与服务器建立会话。在这种情况下将发送特定的错误代码（-124），此后，Java和C客户端都将关闭与服务器的会话，而不再尝试重新连接。

  该配置同时设置了**enforce.auth.enabled=true** 和**enforce.auth.scheme=sasl**。默认地，该特性是不启用的。用户可以通过将**sessionRequireClientSASLAuth**设置为**true**以启用该特性。

  该特性使得zookeeper.allowSaslFailedClients选项失效，所以如果该特性启用的话，即使服务器被配置为允许SASL认证失败的客户端登录，客户端也无法与服务端建立会话。

- *enforce.auth.enabled*：(Java system property：**zookeeper.enforce.auth.enabled**)**3.7.0版本新特性**：如果设置为true，ZooKeeper服务器将仅接受通过配置的身份验证方案与服务器进行身份验证的客户端的连接和请求。身份认证方案可以通过属性enforce.auth.schemes进行配置。未配置任何身份认证方案或者配置了但未通过认证的客户端（例如凭证失效）不能与服务器建立会话。在这种情况下将发送特定的错误代码（-124），此后，Java和C客户端都将关闭与服务器的会话，而不再尝试重新连接。默认地，该特性是不启用的。用户可以通过将**enforce.auth.enabled**设置为**true**以启用该特性。

  如果设置了**enforce.auth.enabled=true**和**enforce.auth.schemes=sasl**，那么zookeeper.allowSaslFailedClients配置就会失效。所以即使服务器被配置为允许SASL认证失败的客户端登录，客户端也无法与服务端建立会话。

- *enforce.auth.schemes*：(Java system property：**zookeeper.enforce.auth.schemes**)**3.7.0版本新特性**：逗号分隔的认证方式列表。在进行任何ZooKeeper操作之前，客户端必须通过至少一种认证方式进行认证。仅当**enforce.auth.enabled**设置为**true**时该属性才使用。

- *sslQuorum*：(Java system property: **zookeeper.sslQuorum**)**3.5.5版本新特性**：启用加密quorum通信。默认值是false。如果启用该特性，请考虑同时启用*leader.closeSocketAsync*和*learner.closeSocketAsync*以避免关闭SSL连接时socket关闭时间较长导致的相关问题。

- *ssl.keyStore.location、ssl.keyStore.password*、*ssl.quorum.keyStore.location* 、*ssl.quorum.keyStore.password*：(Java system properties: **zookeeper.ssl.keyStore.location**、**zookeeper.ssl.keyStore.password**、**zookeeper.ssl.quorum.keyStore.location**、**zookeeper.ssl.quorum.keyStore.password**)**3.5.5版本新特性**：指定Java密钥库的文件路径，其中包含用于客户端和quorum TLS连接的本地凭据，以及用于解锁文件的密码。

- *ssl.keyStore.passwordPath*、*ssl.quorum.keyStore.passwordPath*：(Java system properties: **zookeeper.ssl.keyStore.passwordPath**、**zookeeper.ssl.quorum.keyStore.passwordPath**)**3.8.0版本新特性**：指定包含密钥库密码的文件路径。从文件中读取密码优先于显式密码属性。

- *ssl.keyStore.type*、*ssl.quorum.keyStore.type*：(Java system properties: **zookeeper.ssl.keyStore.type**、**zookeeper.ssl.quorum.keyStore.type**)**3.5.5版本新特性**：指定客户端密钥库和quorum密钥库的文件格式。可选值：JKS，PEM，PKCS12或者null（通过文件名检测）。默认值：null。**3.6.3，3.7.0版本新特性**：新增BCFKS格式。

- *ssl.trustStore.location*、*ssl.trustStore.password*、*ssl.quorum.trustStore.location*、*ssl.quorum.trustStore.password*：(Java system properties: **zookeeper.ssl.trustStore.location**、**zookeeper.ssl.trustStore.password**、**zookeeper.ssl.quorum.trustStore.location**、**zookeeper.ssl.quorum.trustStore.password**)**3.5.5版本新特性**：指定Java信任库的文件路径，该信任库包含用于客户端和quorum TLS连接的远程凭据，以及用于解锁文件的密码。

- *ssl.trustStore.passwordPath*、*ssl.quorum.trustStore.passwordPath*：(Java system properties: **zookeeper.ssl.trustStore.passwordPath**、**zookeeper.ssl.quorum.trustStore.passwordPath**)**3.8.0版本新特性**：指定包含信任库密码的文件路径。从文件中读取密码优先于显式密码属性指定的密码。

- *ssl.trustStore.type*、*ssl.quorum.trustStore.type*：(Java system properties: **zookeeper.ssl.trustStore.type**、**zookeeper.ssl.quorum.trustStore.type**)**3.5.5版本新特性**：指定客户端密钥库和quorum信任库的文件格式。可选值：JKS，PEM，PKCS12或者null（通过文件名检测）。默认值：null。**3.6.3，3.7.0版本新特性**：新增BCFKS格式。

- *ssl.protocol*、*ssl.quorum.protocol*:(Java system properties: **zookeeper.ssl.protocol**、**zookeeper.ssl.quorum.protocol**)**3.5.5版本新特性**：指定客户端以及quorum TLS协商时使用的协议版本。默认值：TLSv1.2。

- *ssl.enabledProtocols*、*ssl.quorum.enabledProtocols*：(Java system properties: **zookeeper.ssl.enabledProtocols**、**zookeeper.ssl.quorum.enabledProtocols**)**3.5.5版本新特性**：指定客户端以及quorum TLS协商时启用的协议版本。默认值：protocol属性值。

- *ssl.ciphersuites*、*ssl.quorum.ciphersuites*：(Java system properties: **zookeeper.ssl.ciphersuites**、**zookeeper.ssl.quorum.ciphersuites**) **3.5.5版本新特性**：指定客户端以及quorum TLS协商时使用的密码套件。默认值：基于所使用的Java运行时版本。

- *ssl.context.supplier.class*、*ssl.quorum.context.supplier.class*：(Java system properties: **zookeeper.ssl.context.supplier.class**、**zookeeper.ssl.quorum.context.supplier.class**) **3.5.5版本新特性**：指定客户端以及quorum SSL通信过程中用来创建SSL上下文的类。这允许使用自定义SSL上下文并实现以下场景：

  1. 使用硬件密码库，通过PKCS11或其他类似方式加载。
  2. 无权访问软件密钥库，但可以从其容器中检索已构建的SSL Context。默认值：null。

- *ssl.hostnameVerification*、*ssl.quorum.hostnameVerification*：(Java system properties: **zookeeper.ssl.hostnameVerification**、**zookeeper.ssl.quorum.hostnameVerification**) **3.5.5版本新特性**：指定是否在客户端以及quorum TLS协商过程中启用主机名验证。建议仅出于测试目的时禁用该选项。默认值：true。

- *ssl.crl*、*ssl.quorum.crl*：(Java system properties: **zookeeper.ssl.crl**、**zookeeper.ssl.quorum.crl**) **3.5.5版本新特性**：指定是否在客户端以及quorum TLS协议中启用证书吊销列表。默认值：false。

- *ssl.ocsp*、*ssl.quorum.ocsp*：(Java system properties: **zookeeper.ssl.ocsp**、**zookeeper.ssl.quorum.ocsp**)**3.5.5版本新特性**：指定是否在客户端以及quorum TLS协议中启用联机证书状态协议。默认值：false。

- *ssl.clientAuth*、*ssl.quorum.clientAuth*：(Java system properties: **zookeeper.ssl.clientAuth**、**zookeeper.ssl.quorum.clientAuth**)**3.5.5中增加,但是直到3.5.7都是失效的**：指定对来自客户端的ssl连接进行身份验证的选项。有效值包括：

  - "none"：服务端不会请求客户端进行认证
  - "want"：服务端请求客户端进行认证
  - "need"：服务端要求客户端进行认证

- *ssl.handshakeDetectionTimeoutMillis*、*ssl.quorum.handshakeDetectionTimeoutMillis*：(Java system properties: **zookeeper.ssl.handshakeDetectionTimeoutMillis**、**zookeeper.ssl.quorum.handshakeDetectionTimeoutMillis**)**3.5.5版本新特性**：待定

- *client.portUnification*: (Java system property: **zookeeper.client.portUnification**)指定客户端端口应该接受SSL连接（使用相同的配置作为安全客户端端口）。默认值：false。

- *authProvider*：(Java system property: **zookeeper.authProvider**)为ZooKeeper指定多个身份验证提供程序类。通常使用此参数指定SASL身份验证提供程序，如`authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider`。

- *kerberos.removeHostFromPrincipal*：(Java system property: **zookeeper.kerberos.removeHostFromPrincipal**)指示ZooKeeper在身份验证期间从客户端主体名称中删除域（例如：zk/myhost@EXAMPLE.COM客户端主体将在ZooKeeper中作为zk/myhost进行身份验证）。默认值：false。

- *kerberos.canonicalizeHostNames*(Java system property: **zookeeper.kerberos.canonicalizeHostNames**)**3.7.0版本新特性**：指示ZooKeeper规范化从server.x行提取的服务器主机名。这允许使用类似于CNAME记录的配置引用配置文件中的服务器，同时仍在quorum成员之间启用SASL Kerberos身份验证。这本质上相当于quorum对于客户端的属性*zookeeper.sasl.client.canonicalize.hostname*。为了后向兼容，默认值是false。

- *multiAddress.enabled*：(Java system property: **zookeeper.multiAddress.enabled**)**3.6.0版本新特性**：从ZooKeeper3.6.0开始对于每个ZooKeeper服务器实例可以指定[多地址](#madresses)（当集群中可以并行使用多个物理网络接口时，这可以提高可用性）。将该特性设置为**true**可以启用该特性。请注意，在滚动升级的过程中，如果老版本的ZooKeeper版本低于3.6.0，那么可以禁用该特性。默认值是false。

- *multiAddress.reachabilityCheckTimeoutMs*：(Java system property: **zookeeper.multiAddress.reachabilityCheckTimeoutMs**)**3.6.0版本新特性**：从ZooKeeper3.6.0开始对于每个ZooKeeper服务器实例可以指定[多地址](#madresses)（当集群中可以并行使用多个物理网络接口时，这可以提高可用性）。ZooKeeper会向目的主机发送ICMP ECHO或者在端口7（Echo）上建立TCP连接以查找可达地址。这仅在配置中提供了多地址时才会发生。该属性可以设置毫秒计数的可达性探测超时时间。可达性探测在多个地址之间并行进行，因此这里设置的超时时间是检查所有地址的可达性所需的最大时间。默认值是**1000**。

  除非通过设置*multiAddress.enabled=true*启用多地址特性，否则该参数是不生效的。

#### 实验选项/功能

当前被认为是实验性的新特性

+ *Read Only Mode Server*：(Java system property: **readonlymode.enabled**)**3.4.0版本新特性**：设置为true启用只读模式服务器支持（默认不启用）。ROM模式允许请求ROM支持的客户端会话即使在服务器有可能从quorum离开的情况下也能连接到服务器。在ROM模式下客户端可以从ZK服务进行读取，但是不能写入和观察到其他客户端的变化。参考ZOOKEEPER-784以获取详细信息。
+ *zookeeper.follower.skipLearnerRequestToNextProcessor*：(Java system property: **zookeeper.follower.skipLearnerRequestToNextProcessor**)当集群中存在连接到ObserverMaster的observer时，打开该选项有助于降低Observer Master的内存压力。如果集群中没有任何observer，或者observer没有连接到ObserverMaster，或者Observer的写操作很少，那么该选项是没有帮助的。目前，该标志导致的变化有助于我们获得更多关于内存增长的把握。从长期来看，我们可能希望删除此标志并将其行为设置为默认代码路径。

#### 不安全选项

下列选项是非常有用的，但是使用的时候需要谨慎。参数风险与参数作用同时做了说明。

- *forceSync*：(Java system property: **zookeeper.forceSync**)在完成更新处理之前，需要将更新同步到事务日志的存储介质。如果此选项设置为no，ZooKeeper将不需要将更新同步到存储。

- *jute.maxbuffer*：(Java system property:**jute.maxbuffer**)。

  - 该选项仅可以设置为Java系统属性。选项之前没有zookeeper前缀。该选项指定了单个znode中可以存储的最大数据量。单位：byte。默认值是oxfffff(1048575)，也就是1M以下。
  - 如果更改此选项，则必须在所有服务器和客户端上设置系统属性，否则会出现问题。
  - 如果客户端的*jute.maxbuffer*大于服务端，当客户端写入的数据超过了服务端的*jute.maxbuffer*时，服务端会抛出**java.io.IOException: Len error**异常！
  - 如果客户端的*jute.maxbuffer*小于服务端，当客户端读取的数据超过了客户端的*jute.maxbuffer*时，客户端会抛出**java.io.IOException: Unreasonable length**或者**Packet len is out of range**！
  - 这是一个必要的检验。ZooKeeper设计用于存储大小约为千字节的数据。在生产环境中，不建议将此属性增加到超过默认值，原因如下：
    - 大数据量的znode会导致不合理的延迟峰值，使吞吐量恶化
    - 较大的znode使leader和follower之间的同步时间不可预测且不收敛（有时超时），导致quorum不稳定

- *jute.maxbuffer.extrasize*：(Java system property: **zookeeper.jute.maxbuffer.extrasize**) **3.5.7版本新特性**：处理客户端请求时，ZooKeeper在将其持久化到事务之前在其中添加了一些额外信息。此前，该信息大小被固定为1024字节。对于许多场景，特别是jute.maxbuffer超过1MB并且请求类型为multi的场景下，这个大小是不足的。为了应对所有场景，额外信息从1024提升到了与*jute.maxbuffer*相同的大小，并且也通过jute.maxbuffer.extrasize进行设置。通常来说该属性不需要设置，默认值就是最优值。

- *skipACL*：(Java system property: **zookeeper.skipACL**)跳过ACL检查。该选项可以大大提升吞吐量，但是会向所有人开放对数据树的完全访问权限。

- *quorumListenOnAllIPs*：如果设置为true，除了在配置文件中的服务器列表外，ZooKeeper会在所有可用的IP地址上监听来自peer的连接。该配值影响ZAB协议和快速Leader选举协议的所用的连接。默认值时false。

- *multiAddress.reachabilityCheckEnabled*：(Java system property: **zookeeper.multiAddress.reachabilityCheckEnabled**)**3.6.0版本新特性**：从ZooKeeper3.6.0开始对于每个ZooKeeper服务器实例可以指定[多地址](#madresses)（当集群中可以并行使用多个物理网络接口时，这可以提高可用性）。ZooKeeper会向目的主机发送ICMP ECHO或者在端口7（Echo）上建立TCP连接以查找可达地址。这仅在配置中提供了多地址时才会发生。当尝试在一台机器上启动大型（例如11+）集群进行测试时，如果遇到某些ICMP速率限制（如在macOS上），则可达性探测可能会失败。

  默认值是**true**。通过设置为‘false’可以禁用可达性探测。请注意，禁用可达性探测会导致集群在网络故障期间无法正确重新自配置，因此仅建议在测试期间禁用。

  除非通过设置*multiAddress.enabled=true*启用多地址特性，否则该参数是不生效的。

#### 禁用数据目录自动创建

**3.5.0版本新特性**：ZooKeeper的默认行为是如果启动时数据目录（由配置文件指定）不存在则自动创建。在某些情况下，这可能会带来不便甚至危险。以对正在运行的服务器进行配置更改的情况为例，其中**dataDir**参数意外更改。当ZooKeeper服务器重新启动时，它将创建这个不存在的目录，并开始使用空的znode名字空间提供服务。这种情况可能导致实际的“脑裂”情况（即新的无效目录和原始有效数据存储中的数据）。因此，最好能选择关闭这种自动创建行为。一般来说，对于生产环境，应该这样做，但不幸的是，此时无法更改默认的遗留行为，因此必须根据具体情况进行更改。这留给用户和ZooKeeper发行版的打包人员。

当运行**zkServer.sh**，可以通过将环境变量**ZOO_DATADIR_AUTOCREATE_DISABLE**设置为1以禁用自动创建。如果直接通过类文件运行ZooKeeper服务器，可以通过在java命令行中将设置**zookeeper.datadir.autocreate=false**来实现这一点，例如**`-Dzookeeper.datadir.autocreate=false`**

如果禁用此功能，当ZooKeeper服务器确定所需的目录不存在时，它将生成错误并拒绝启动。

ZooKeeper提供了新的脚本**zkServer-initialize.sh**以支持这一新特性。如果禁用了autocreate，则用户需要首先安装ZooKeeper，然后创建数据目录（可能还有txnlog目录），然后启动服务器。否则如前一段所述，服务器将无法启动。运行**zkServer-initialize.sh**将创建所需的目录，并可以选择设置myid文件（可选命令行参数）。即使不使用自动创建功能本身，也可以使用此脚本，并且用户可能会用到它，因为这（设置，包括创建myid文件）在过去一直是用户的问题。请注意，此脚本确保数据目录仅存在，它不会创建配置文件，而是需要一个可用的配置文件才能执行。

#### 启用db存在验证

**3.6.0版本新特性**：ZooKeeper的默认行为是如果在启动时没有发现数据树，就会将zxid设置为0并且以投票成员的身份加入quorum。如果某个事件（如恶意'rm-rf'）在服务器关闭时删除了数据目录，该行可能导致危险，因为此服务器可能有助于选择缺少事务的leader。启用db存在验证会改变没有发现数据树时的启动行为：服务器作为无投票权的参与者加入集群直到其能够与leader进行同步并且获取最新版本的集群数据。为了表预期的是空的数据树（集群创建），用户应将“initialize”文件放在与‘’myid‘相同的目录中。服务器启动时会检测并删除该文件。

直接通过类文件运行ZooKeeper时，可以通过在java命令行中设置**zookeeper.db.autocreate=false**以启用初始化校验，例如**-Dzookeeper.db.autocreate=false**。运行**zkServer-initialize.sh**会创建需要的初始化文件。

#### 性能调整选项

**3.5.0版本新特性**：数个子系统已经改造以提升读吞吐量。改造工作包括多线程的NIO通信子系统和请求处理管道（Commit Processor）。NIO是默认的客户端/服务器通信子系统。NIO的线程模型包括1个acceptor线程，1-N个selector线程以及0-M个socket I/O woker线程。在请求处理管道中，可以将系统配置为一次处理多个读取请求，同时保持相同的一致性保证（同一会话写后读）。Commit Processor包的线程模型包含1个main线程和0-N个worker线程。

默认值旨在最大化专用的ZooKeeper机器上的读取吞吐量。这两个子系统都需要有足够数量的线程来实现峰值读取吞吐量。

- *zookeeper.nio.numSelectorThreads*：(Java system property only: **zookeeper.nio.numSelectorThreads**) **3.5.0版本新特新**：NIO selector线程数量。至少需要一个selector线程。对于大量的客户端连接，建议使用一个以上的selecor线程。默认值是cup核心数一半的平方根。
- *zookeeper.nio.numWorkerThreads*：(Java system property only: **zookeeper.nio.numWorkerThreads**) **3.5.0版本新特新**：NIO worker线程数量。如果配置0个worker线程，selector会直接进行socket I/O操作。默认值是2倍的cpu核心数。
- *zookeeper.commitProcessor.numWorkerThreads*：(Java system property only: **zookeeper.commitProcessor.numWorkerThreads**) **3.5.0版本新特新**：Commit Processor worker线程数量。如果配置0个worker线程，main线程会直接处理请求。默认值是cpu核心数。
- *zookeeper.commitProcessor.maxReadBatchSize*：(Java system property only: **zookeeper.commitProcessor.maxReadBatchSize**)切换到提交处理之前，可以从请求队列中读取的请求的最大数量。如果该值小于0（默认值），那么只要有本地写入和挂起的提交就会切换。较高的读取批处理大小将延迟提交处理，导致提供陈旧数据。如果已知读请求会按照固定的批处理大小到达，那么将该批处理大小与此属性的值匹配可以平滑队列性能。由于读操作是并行处理的，所以建议将该值与*zookeeper.commitProcessor.numWorkerThread*（默认值是cpu数量）匹配或者设的更低。
- *zookeeper.commitProcessor.maxCommitBatchSize*：(Java system property only: **zookeeper.commitProcessor.maxCommitBatchSize**)处理读取之前要处理的最大提交数。ZooKeeper将尝试处理尽可能多的远程/本地提交，直到达到这个数量。较高的提交批处理大小将在处理更多提交时延迟读取。较低的提交批处理大小将有利于读取。建议仅当集群以高提交率服务于工作负载时才设置此属性。如果已知写会按照设定的数量到达，则将该批处理大小与此属性的值匹配可以平滑队列性能。一般方法是将此值设置为等于集群大小，以便在处理每个批处理时，当前服务器将概率性地处理与其直连的客户端之一相关的写入。默认值为“1”。不支持负值和零值。
- *znode.container.checkIntervalMs*：(Java system property only) **3.6.0版本新特新**：毫秒计数的检查候选容器和ttl节点的时间间隔。默认值是“60000”。
- *znode.container.maxPerMinute*：(Java system property only) **3.6.0版本新特新**：每分钟可以删除的容器和ttl节点的最大数量。这可以防止容器删除期间出现羊群现象。默认值为“10000”。
- *znode.container.maxNeverUsedIntervalMs*：(Java system property only) **3.6.0版本新特新**：毫秒计数的没有任何子节点的容器的最大保留时间。该时间需要足够长以便客户端创建容器，完成任何需要的工作，然后创建子对象。默认值为“0”表示从未有过任何子对象的容器不会被删除。

Debug可观察性配置

**3.6.0版本特性**：引入下列选项使得zookeeper更易于debug。

+ *zookeeper.messageTracker.BufferSize*：(Java system property only)控制存储在**MessageTracker**中的消息数量的最大值。该值应该是一个正值。默认值是10。**3.6.0**引入**MessageTracker**是为了在一台服务器（follower或者observer）与leader断开连接时记录它们之间最后的消息集合。这些消息集随后将被转储到zookeeper的日志文件中，并将有助于在断开连接时重建服务器的状态，对调试非常有用。
+ *zookeeper.messageTracker.Enabled*：(Java system property only)设置为”true“会启用**MessageTracker**以追踪和记录消息。默认值是”false“。

#### AdminServer配置

**3.7.1版本特性**：下列选项用来配置[AdminServer](####AdminServer)。

+ *admin.forceHttps*：(Java system property: **zookeeper.admin.forceHttps**)强制AdminServer使用SSL，因此仅仅允许HTTPS流量。默认不启用。覆盖**admin.portUnification**设置。

**3.6.0版本特性**：下列选项用来配置[AdminServer](####AdminServer)。

+ *admin.portUnification*：(Java system property: **zookeeper.admin.portUnification**)：admin端口同时接受HTTP和HTTPS请求。默认不启用。

**3.5.0版本特性**：下列选项用来配置[AdminServer](####AdminServer)。

+ *admin.enableServer*：(Java system property: **zookeeper.admin.enableServer**)设置为”false“禁用AdminServer。默认地AdminServer是启用的。
+ *admin.serverAddress*：(Java system property: **zookeeper.admin.serverAddress**)内置的Jetty服务器的监听地址。默认是0.0.0.0。
+ *admin.serverPort*：(Java system property: **zookeeper.admin.serverPort**)内置的Jetty服务器的监听端口。默认是8080。
+ *admin.idleTimeout*：(Java system property: **zookeeper.admin.idleTimeout**)一个连接发送或接收数据时的最大空闲时间（毫秒）。默认值是30000ms。
+ *admin.commandURL*：(Java system property: **zookeeper.admin.commandURL**)监听并处理命令的URL（相对于根URL）。默认值是”/commands“。

#### 指标提供

**3.6.0版本特性**：下列选项用来配置指标。

默认情况下，ZooKeeper通过[AdminServer](https://zookeeper.apache.org/doc/r3.8.0/zookeeperAdmin.html#sc_adminserver)和[Four Letter Words](https://zookeeper.apache.org/doc/r3.8.0/zookeeperAdmin.html#sc_4lw)接口提供了有用的运行指标。

从3.6.0开始可以配置不同的指标提供程序，将运行指标暴露给用户希望的系统。

从3.6.0开始ZooKeeper二进制包捆绑了与[Prometheus.io](https://prometheus.io/)的集成。

+ *metricsProvider.className*：设置为”org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider“以启用Prometheus.io exporter。
+ *metricsProvider.httpHost*：**3.8.0版本特性**：Prometheus.io exporter会在该地址启动Jetty服务器，默认值是”0.0.0.0“。
+ *metricsProvider.httpPort*：Prometheus.io exporter会在该端口启动Jetty服务器，默认值是7000。Prometheus访问点将是http://hostname:httPort/metrics。
+ *metricsProvider.exportJvmInfo*：设置为**true** Prometheus.io会导出有用的Jvm运行指标。默认值是true。
+ *metricsProvider.numWorkerThreads*：**3.7.1版本特性**：报告Prometheus摘要指标的worker线程的数量。默认值是1。如果小于1则使用main线程。
+ *metricsProvider.maxQueueSize*：**3.7.1版本特性**：Prometheus摘要指标报告任务的最大队列大小。默认值是1000000。
+ *metricsProvider.workerShutdownTimeoutMs*：**3.7.1版本特性**：Prometheus worker线程的超时时间（毫秒）。默认值是1000ms。

### 使用Netty框架进行通信

[Netty](http://netty.io/)是基于NIO的客户端/服务器通信框架，它简化了（直接使用NIO时）java应用程序网络级通信的许多复杂性。此外，Netty框架内置了对加密（SSL）和身份验证（证书）的支持。这些是可选功能，可以单独打开或关闭。

在3.5+版本中，ZooKeeper可以通过设置环境变量**zookeeper.serverCnxnFactory**设置为**org.apache.zookeeper.server.NettyServerCnxnFactory**以使用Netty而不是NIO（默认选项）；对于客户端，将**zookeeper.clientCnxnSocket**设置为**org.apache.zookeeper.ClientCnxnSocketNetty**。

#### Quorum TLS

*3.5.5新特性*

基于Netty框架，ZooKeeper集群可以设置为在其通信信道中使用TLS加密。本节介绍如何在quorum通信上设置加密。

请注意，Quorum TLS封装了leader选举和quorum通信协议。

1. 创建SSL密钥库JKS以存储本地凭据

每个ZK实例创建一个密钥库。

本例生成一个自签名证书，并将其与私钥一起存储在*keystore.jks*中。这适合于测试目的，生产环境中可能需要一个官方证书来签名密钥。

请注意，别名（-alias）和标识名称（-dname）必须与关联的计算机的主机名匹配，否则主机名验证将无法工作。

~~~shell
keytool -genkeypair -alias $(hostname -f) -keyalg RSA -keysize 2048 -dname "cn=$(hostname -f)" -keypass password -keystore keystore.jks -storepass password
~~~

1. 从密钥库提取签名的公钥（证书）

*该步骤可能仅对自签名证书时必要的。*

~~~shell
keytool -exportcert -alias $(hostname -f) -keystore keystore.jks -file $(hostname -f).cer -rfc
~~~

1. 创建包含所有ZooKeeper实例证书的SSL信任库JKS

应该在集群的所有成员上共享相同的信任库（存储所有接受的证书）。需要使用不同的别名在同一信任库中存储多个证书。别名的名称无关紧要。

~~~shell
keytool -importcert -alias [host1..3] -file [host1..3].cer -keystore truststore.jks -storepass password
~~~

1. 需要使用NettyServerCnxnFactory作为serverCnxnFactory，这是因为NIO不支持SSL。向zoo.cfg添加以下配置：

~~~shell
sslQuorum=true
serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
ssl.quorum.keyStore.location=/path/to/keystore.jks
ssl.quorum.keyStore.password=password
ssl.quorum.trustStore.location=/path/to/truststore.jks
ssl.quorum.trustStore.password=password
~~~

1. 检查运行于TLS方式的集群的日志：

~~~shell
INFO  [main:QuorumPeer@1789] - Using TLS encrypted quorum communication
INFO  [main:QuorumPeer@1797] - Port unification disabled
...
INFO  [QuorumPeerListener:QuorumCnxManager$Listener@877] - Creating TLS-only quorum server socket
~~~

#### 非TLS集群不停机升级

*3.5.5新特性*

下列步骤使用端口联合功能在不停机的状况下对ZooKeeper集群进行TLS升级。

1. 按照之前小节的描述对所有ZK成员创建密钥库和信任库
2. 添加下列配置并且重启第一个节点

~~~shell
sslQuorum=false
portUnification=true
serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
ssl.quorum.keyStore.location=/path/to/keystore.jks
ssl.quorum.keyStore.password=password
ssl.quorum.trustStore.location=/path/to/truststore.jks
ssl.quorum.trustStore.password=password
~~~

注意TLS尚未启用，但是端口联合功能已启用。

1. 对剩余的节点重复步骤#2。验证是否在日志中看到下列条目：

~~~shell
INFO  [main:QuorumPeer@1791] - Using insecure (non-TLS) quorum communication
INFO  [main:QuorumPeer@1797] - Port unification enabled
...
INFO  [QuorumPeerListener:QuorumCnxManager$Listener@874] - Creating TLS-enabled quorum server socket
~~~

在每个节点重新启动后再次检查quorum是否正常。

1. 在每个节点上启用Quorum TLS并且滚动重启：

~~~shell
sslQuorum=true
portUnification=true
~~~

1. 如果验证了整个集群都是在TLS的方式上运行，那么可以禁用端口联合功能并且再做一次滚动重启

~~~shell
sslQuorum=true
portUnification=false
~~~

### ZooKeeper命令

#### Four Letter Words

ZooKeeper会响应一组精简的命令。每条命令由四个字母组成。可以通过telnet或nc在客户端端口向ZooKeeper发出命令。

其中三个更有趣的命令：“stat”提供了有关服务器和连接的客户端的一般信息，而“srvr”和“cons”分别提供了服务器和连接方面的详细信息。

**3.5.3版本特性**：Four Letter Words在使用前需要明确列出。请[参考集群配置一节]()中描述的**4lw.commands.whitelist**以获取详细信息。在后续计划中，Four Letter Words将被废弃，请使用 [AdminServer]()。

- *conf*：**3.3.0版本新特性**：输出服务器的详细配置。

- *cons*：**3.3.0版本新特性**：列出连接到此服务器的所有客户端的完整连接/会话的详细信息。包括接收/发送的数据包数量、会话id、操作延迟、上次执行的操作等信息。。。

- *crst*：**3.3.0版本新特性**：重置所有连接/会话的统计数据。

- *dump*：列出未完成的会话和临时节点。

- *envi*：打印服务器环境的详细信息。

- *ruok*：当白名单启用ruok时，如果服务器正在运行，它将使用*imok*进行响应，否则它将完全不响应。当ruok未启用时，服务器会回应：“ruok is not executed because it is not in the whitelist.”。响应“imok”不一定表示服务器已加入quorum，只表示服务器进程处于活动状态并绑定到指定的客户端端口。有关状态wrt quorum和客户端连接信息的详细信息，请使用“stat”。

- *srst*：重置服务器统计数据。

- *srvr*：**3.3.0版本新特性**：列出服务器的完整详细信息。

- *stat*：列出服务器和连接的客户端的简要详细信息。

- *wchs*：**3.3.0版本新特性**：列出服务器上的watch的简要详细信息。

- *wchc*：**3.3.0版本新特性**：按会话列出服务器watch的详细信息。该领命输出一个会话（连接）及其关联watch（路径）的列表。注意，基于watch的数量不同，该命令可能是代价非常大的（即非常消耗服务器性能），请谨慎使用。

- *dirs*：**3.5.1版本新特性**：输出快照和日志文件的总字节数。

- *wchp*：**3.3.0版本新特性**：按路径列出服务器watch的详细信息。该领命输出一个路径（znode）及其关联会话的列表。注意，基于watch的数量不同，该命令可能是代价非常大的（即非常消耗服务器性能），请谨慎使用。

- *mntr*：**3.4.0版本新特性**：输出可用于监视群集运行状况的变量列表。

  ~~~shell
  $ echo mntr | nc localhost 2185 zk_version 3.4.0 zk_avg_latency 0.7561 - be account to four decimal places zk_max_latency 0 zk_min_latency 0 zk_packets_received 70 zk_packets_sent 69 zk_outstanding_requests 0 zk_server_state leader zk_znode_count 4 zk_watch_count 0 zk_ephemerals_count 0 zk_approximate_data_size 27 zk_followers 4 - only exposed by the Leader zk_synced_followers 4 - only exposed by the Leader zk_pending_syncs 0 - only exposed by the Leader zk_open_file_descriptor_count 23 - only available on Unix platforms zk_max_file_descriptor_count 1024 - only available on Unix platforms
  ~~~

  输出与java属性格式兼容，内容可能会随时间变化（添加了新key）。脚本应该会发生更改。注意：有些key是特定于平台的，有些key仅由leader导出。输出包含以下格式的多行：

  ~~~shell
  key \t value
  ~~~

- *isro*：**3.4.0版本新特性**:检测服务器是否以只读模式运行。如果是只读模式的话服务器会回应“ro”，否则回应“rw”。

- *hash*：**3.6.0版本新特性**：返回与zxid关联的树摘要的最新历史记录。

- *gtmk*：以十进制64位有符号长值的格式获取当前的跟踪掩码。可能的取值请参考[stmk]()。

- *stmk*：设置当前的跟踪掩码。跟踪掩码为64位，其中每个位启用或禁用服务器上特定类别的跟踪日志记录。为了查看跟踪日志消息，logback必须先配置以启用**TRACE**级别。跟踪掩码的位对应于以下跟踪日志记录类别。

  | 跟踪掩码比特值 |                                                              |
  | -------------- | ------------------------------------------------------------ |
  | 0b0000000000   | 未使用，保留供将来使用。                                     |
  | 0b0000000010   | 记录除了ping请求以外的客户端请求。                           |
  | 0b0000000100   | 未使用，保留供将来使用。                                     |
  | 0b0000001000   | 记录客户端ping请求。                                         |
  | 0b0000010000   | 记录从作为当前leader的quorum peer接收到的除了ping以外的请求。 |
  | 0b0000100000   | 记录客户端会话的添加、删除和验证。                           |
  | 0b0001000000   | 记录watch事件向客户端会话的传递。                            |
  | 0b0010000000   | 记录从作为当前leader的quorum peer接收到的ping请求。          |
  | 0b0100000000   | 未使用，保留供将来使用。                                     |
  | 0b1000000000   | 未使用，保留供将来使用。                                     |

  64比特值中的所有剩余位均未使用，并保留以供将来使用。通过计算记录值的逐位OR来指定多个跟踪日志记录类别。默认的跟踪掩码是0b0100110010。因此，因此，默认情况下，跟踪日志记录包括客户端请求、从leader接收的数据包和会话。要设置不同的跟踪掩码，请发送一个请求，其中包含**stmk** four-letter命令，后跟以64位有符号长值表示的跟踪掩码。此示例使用Perl-pack函数构造一个跟踪掩码，该掩码启用上述所有跟踪日志记录类别，并将其转换为具有大端字节顺序的64位有符号长值。结果被附加到stmk并使用netcat发送到服务器。服务器用十进制格式的新跟踪掩码进行响应。

  ~~~shell
  $ perl -e "print 'stmk', pack('q>', 0b0011111010)" | nc localhost 2181 250
  ~~~

  下面是一个**ruok**命令的例子。

  ~~~shell
  $ echo ruok | nc 127.0.0.1 5111
      imok
  ~~~

#### AdminServer

**3.5.0版本新特性**：AdminServer是一个内置的Jetty服务器，提供了HTTP接口访问four-letter命令。默认地，该服务器在8080端口监听，通过URL “/commands/[command name]“发送命令，例如，http://localhost:8080/commands/stat。命令响应以JSON格式返回。不像原始的协议，命令并不局限于four-letter名字，可以有多个名字；例如，”stmk“也可以通过”set_trace_mask“引用。通过浏览器访问URL/commands（例如，http://localhost:8080/commands）以获取完整的命令列表。关于如何改变端口和URLs请参考[AdminServer configuration options](https://zookeeper.apache.org/doc/r3.8.0/zookeeperAdmin.html#sc_adminserver_config)。

AdminServer默认是启用的，但是可以通过以下任意方式禁用：

+ 将zookeeper.admin.enableServer系统属性设置为false。
+ 从类路径中移除Jetty。（如果希望覆盖ZooKeeper的Jetty依赖，此选项非常有用。）

注意如果AdminServer被禁用，TCP four-letter命令接口仍然是可用的。

可用的命令包括：

+ *connection_stat_reset/crst*：重置所有客户端连接统计数据。没有新的域返回。
+ *configuration/conf/config*：打印服务端配置的基本信息，例如客户端端口，data目录的绝对路径。
+ *connections/cons*：服务端的客户连接信息。注意，基于客户端连接的数量不同该操作可能是代价非常大的（即非常消耗服务器性能）。返回”connections“，一个连接信息对象列表。
+ *hash*：历史摘要列表中的Txn摘要。每128个事务日志记录一次。返回”digests“，一个事务摘要对象列表。
+ *dirs*：关于日志文件目录和快照目录大小的信息。返回”datadir_size“和”logdir_size“。
+ *dump*：关于会话过期和临时节点的信息。注意，基于全局会话和临时节点的数量不同该操作可能是代价非常大的（即非常消耗服务器性能）。以字典的形式返回”expiry_time_to_session_ids“和”session_id_to_ephemeral_paths“。
+ *environment/env/envi*：所有定义的环境变量。将每一个环境变量作为自己的字段返回。
+ *get_trace_mask/gtmk*：当前的跟踪掩码。*set_trace_mask*的只读版本。更多信息请参考four-letter命令*stmk*。返回”tracemask“。
+ *initial_configuration/icfg*：打印用于启动peer的配置文件的文本。返回”initial_configuration“。
+ *is_read_only/isro*：根据服务器是否是只读模式返回true/false。返回”read_only“。
+ *last_snapshot/lsnp*：zookeeper服务器已成功保存到磁盘的最后一个快照的信息。如果在服务器启动和服务器完成保存其第一个快照之间的初始时间段内调用，该命令将返回启动服务器时读取的快照信息。返回“zxid”和“timestamp”，后者使用秒作为时间单位。
+ *monitor/mntr*：发出各种有用的信息用于监控。包括性能统计信息、有关内部队列的信息和数据树的摘要（以及其他内容）。将每一个变量作为自己的域返回。
+ *observer_connection_stat_reset/orst*：重置所有observer连接统计数据。适用于observer的命令。没有新的域返回。
+ *ruok*：非操作命令，检查服务器是否处于运行状态。应答并不一定意味着服务器已经加入quorum，仅仅表示服务器是活动的并且已经绑定了特定的端口。没有新的域返回。
+ *set_trace_mask/stmk*：设置跟踪掩码（因此，它需要一个参数）。*get_trace_mask*的可写版本。更多信息请参考four-letter命令*stmk*。返回“tracemask”。
+ *server_stats/srvr*：服务器信息。返回多个字段，简要描述服务器状态。
+ *stats/stat*：与*server_stats*类似但是同时返回“connections”域（参考*connections*）。注意，基于客户端连接的数量不同该操作可能是代价非常大的（即非常消耗服务器性能）。
+ *stat_reset/srst*：重置服务器的统计数据。这是*server_stats*和*stats*返回的信息的子集。没有新的域返回。
+ *observers/obsr*：连接到服务器的observer连接的信息。在leader上始终是可用的，如果follower充当学习者的master，那么该follower也是可用的。返回“synced_observers”（整数）和“observers”（每个observer的属性列表）。
+ *system_properties/sysp*：所有定义的系统属性。将每一个属性作为自己的字段返回。
+ *voting_view*：提供集群中的当前投票成员。以字典的形式返回“current_config”。
+ *watches/wchc*：按照会话分类的watch信息。注意，基于watch的数量不同，该命令可能是代价非常大的（即非常消耗服务器性能）。以字典的形式返回“session_id_to_watched_paths”。
+ *watches_by_path/wchp*：按照路径分类的watch信息。注意，基于watch的数量不同，该命令可能是代价非常大的（即非常消耗服务器性能）。以字典的形式返回“path_to_session_ids”。
+ *watch_summary/wchs*：汇总的watch信息。返回“num_total_watches”、“num_paths”和“num_connections”。
+ *zabstate*：peer正在运行的Zab协议的当前阶段以及它是否在投票。peer可以处于以下阶段：ELECTION，DISCOVERY，SYNCHRONIZATION，BROADCAST。返回“voting”和“zabstate”。

### 数据文件管理

ZooKeeper分别在数据目录和事务日志目录中存储数据和事务日志。默认地这两个目录是一致的。服务器可以（并且应该）将事务日志存储在与数据文件不同的目录中。当事务日志存储在专用日志设备上可以提高吞吐量并降低延迟。

#### 数据目录

有两个或三个文件存在于该目录中：

+ *myid* - 包含人类可读 ASCII 文本中形式的单个整数，表示服务器 ID。
+ *initialize* - 该文件存在表示缺少数据树。创建数据树后该文件被清理。
+ *snapshot* - 保存数据树的模糊快照。

每台ZooKeeper服务器都有唯一的id。该id在两个位置使用：*myid*文件和配置文件。*myid*文件标识与给定数据目录关联应的服务器。配置文件列出了由服务器ID标识的每个服务器的联系信息。当一个实例启动时，ZooKeeper从*myid*文件获取它的id，然后读取配置文件，使用该id查找它应该监听的端口。

数据目录中存储的快照文件是模糊快照，从某种意义上来说当ZooKeeper采集快照的时候，数据树正在发生更新。快照文件名的后缀是快照开始时最后一个已提交事务的 zxid（ZooKeeper 事务 ID）。因此，快照包括快照处理过程中发生的数据树更新的子集。此时，快照可能与实际存在的任何数据树都不对应，所以我们将其称为模糊快照。尽管如此ZooKeeper仍然可以使用此快照进行恢复，因为它利用了其更新的幂等性质。通过针对模糊快照重放事务日志，ZooKeeper会在日志结束获取系统的状态。

#### 日志目录

日志目录包括ZooKeeper的事务日志。在任何更新发生之前，ZooKeeper会确保代表更新的事务写入非易失去性存储。写入当前日志文件的事务数量达到（变量）阈值后会创建新的日志文件。阈值是使用影响快照频率的相同参数计算的（请参考上文的snapCount和snapSizeLimitInKb）。日志文件的前缀是第一个写入该日志的zxid。

文件管理

快照和日志文件在单机模式的ZooKeeper服务器和不同配置的集群模式的ZooKeeper之间是相同的。因此，可以将这些文件从运行中的集群ZooKeeper服务器复制到部署有单机模式的ZooKeeper服务器的开发机器上进行故障排除。

使用旧的日志和快照文件，可以查看ZooKeeper服务器以前的状态，甚至可以恢复那个状态。

ZooKeeper服务器创建快照和日志文件但是从来不删除他们。数据和日志文件保留策略是在ZooKeeper服务器之外执行的。服务器本身只需要最新的完整的模糊快照，它之后的所有日志文件，以及它之前的最后一个日志文件。后一项要求对于将在此快照启动后发生但当时已进入现有日志文件的更新包括在内是必需的。由于在ZooKeeper中日志的快照和滚动更新在某种程度上是独立的，所以这种情况是可能发生的。更多关于保留策略和ZooKeeper存储维护设置的详细信息请参考本文档中的[维护](###维护)一节。

> 注意
>
> 存储在这些文件中的数据未加密。在ZooKeeper中存储敏感数据的情况下，需要采取必要措施来防止未经授权的访问。此类措施是ZooKeeper的外部措施（例如，控制对文件的访问），并且取决于部署它的个别设置。

#### 恢复 - TxnLogToolkit

更多信息可以在[这里](http://zookeeper.apache.org/doc/current/zookeeperTools.html#zkTxnLogToolkit)找到。

### 注意事项

以下是一些通过正确配置ZooKeeper可以避免的常见问题。

+ *服务器列表不一致*：客户端使用的ZooKeeper服务器列表必须与每台ZooKeeper服务器都有的ZooKeeper服务器列表相匹配。如果客户端的列表是真实列表的子集也没有问题，但是如果客户端的ZooKeeper服务器列表中的服务器属于不同的ZooKeeper集群就会发生奇怪的问题。另外，每个Zookeeper服务器配置文件中的服务器列表应该是相互一致的。
+ *事务日志没有存储到正确位置*：ZooKeeper性能最关键的部分是事务日志。ZooKeeper在返回应答前会将事务同步到介质。专用的事务日志设备是持续良好性能的关键。把日志放在一个繁忙的设备上会对性能产生不利的影响。如果只有一个存储设备，增加snapCount，使快照文件产生的频率降低；这并不能消除问题，但可以为事务日志提供更多资源。
+ *错误的Java堆大小*：应该特别注意正确设置Java最大堆大小。特别是，不应该创造一个ZooKeeper swap到磁盘的情况。swap到磁盘会导致ZooKeeper性能严重下降。一切都是有序的，所以如果处理一个请求时swap了磁盘，所有其他排队的请求可能也会这样做。**不要发生swap！**保守估计：如果有4G的内存，不要把Java的最大堆大小设置为6G甚至4G。例如，对一台4G机器最好使用3G堆，因为操作系统和缓存也需要内存。估算系统所需的堆大小，最好的也是唯一推荐的做法是运行负载测试，然后确保远远低于会导致系统swap的使用限制。
+ *公开可访问的部署*：ZooKeeper集群应该在可信的计算环境中运行。因此建议在防火前之后部署ZooKeeper。

### 最佳实践

为了获得最佳效果，请注意以下ZooKeeper良好实践：

对于多租户安装，请参考详细介绍ZooKeeper "chroot "支持的[部分](https://zookeeper.apache.org/doc/r3.8.0/zookeeperProgrammers.html#ch_zkSessions)，这在部署多应用程序/服务与单个ZooKeeper集群对接时可能非常有用。
