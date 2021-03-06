![Kafka 示意图](https://www.confluent.io/wp-content/uploads/streaming_platform_rev.png)

# Kafka

Kafka 是由 Linkedin 公司开发的，它是一个分布式的，支持多分区、多副本，基于 Zookeeper 的分布式消息流平台，它同时也是一款开源的基于发布订阅模式的消息引擎系统。与传统的消息系统相比，Kafka 能够很好地处理活跃的流数据，使得数据在各个子系统中高性能、低延迟地不停流转。它最初由 LinkedIn 公司开发，后来成为 Apache 项目的一部分。Kafka 核心模块使用 Scala 语言开发，支持多语言(如 Java、C/C++、Python、Go、Erlang、Node.js 等)客户端，它以可水平扩展和具有高吞吐量等特性而被广泛使用。

![Kafka 管道应用示意图](https://pic.imgdb.cn/item/6077f7a18322e6675cb26c90.png)

据 Kafka 官方网站介绍，当前的 Kafka 已经定位为一个分布式流式处理平台(a distributed streaming platform)，在官方看来，作为一个流式处理平台，必须具备以下 3 个关键特性。

- 能够允许发布和订阅流数据。从这个角度来讲，平台更像一个消息队列或者企业级的消息系统。
- 存储流数据时提供相应的容错机制。
- 当流数据到达时能够被及时处理。

Kafka 能够很好满足以上 3 个特性，通过 Kafka 能够很好地建立实时流式数据通道，由该通道可靠地获取系统或应用程序的数据，也可以通过 Kafka 方便地构建实时流数据应用来转换或是对流式数据进行响应处理。特别是在 0.10 版本之后，Kafka 推出了 Kafka Streams，这让 Kafka 对流数据处理变得更加方便。

# 特性

Kafka 是一种分布式的，基于发布/订阅的消息系统。主要设计目标与特性如下：

- 高吞吐、低延迟：高吞吐量是 Kafka 设计的主要目标，即使在非常廉价的商用机器上也能做到单机支持每秒 100K 条以上消息的传输。Kafka 将数据写到磁盘，充分利用磁盘的顺序读写。同时，Kafka 在数据写入及数据同步采用了零拷贝(zero-copy)技术，采用 sendFile() 函数调用，sendFile() 函数是在两个文件描述符之间直接传递数据，完全在内核中操作，从而避免了内核缓冲区与用户缓冲区之间数据的拷贝，操作效率极高。Kafka 还支持数据压缩及批量发送，同时 Kafka 将每个主题划分为多个分区，这一系列的优化及实现方法使得 Kafka 具有很高的吞吐量。经大多数公司对 Kafka 应用的验证，Kafka 支持每秒数百万级别的消息。

- 持久性与可靠性：Kafka 能够允许数据的持久化存储，消息被持久化到磁盘，并支持数据备份防止数据丢失；以时间复杂度为 O(1)的方式提供消息持久化能力，即使对 TB 级以上数据也能保证常数时间复杂度的访问性能。Kafka 可以为每个主题指定副本数，对数据进行持久化备份，这可以一定程度上防止数据丢失，提高可用性。

- 扩展性与高伸缩性：每个主题(topic) 包含多个分区(partition)，主题中的分区可以分布在不同的主机(broker)中。Kafka 要支持对大规模数据的处理，就必须能够对集群进行扩展，分布式必须是其特性之一，这样就可以将多台廉价的 PC 服务器搭建成一个大规模的消息系统。Kafka 依赖 ZooKeeper 来对集群进行协调管理，这样使得 Kafka 更加容易进行水平扩展，生产者、消费者和代理都为分布式，可配置多个。同时在机器扩展时无需将整个集群停机，集群能够自动感知，重新进行负责均衡及数据复制。

- 轻量级：Kafka 的代理是无状态的，即代理不记录消息是否被消费，消费偏移量的管理交由消费者自己或组协调器来维护。同时集群本身几乎不需要生产者和消费者的状态信息，这就使得 Kafka 非常轻量级，同时生产者和消费者客户端实现也非常轻量级。

- 消息压缩：Kafka 支持 Gzip、Snappy、LZ4 这 3 种压缩方式，通常把多条消息放在一起组成 MessageSet，然后再把 MessageSet 放到一条消息里面去，从而提高压缩比率进而提高吞吐量。

由于 Kafka 本身是存储计算耦合的架构，使得数据不均衡的问题经常凸显，集群扩容、故障恢复也变得异常麻烦，给运维工作带来不少痛苦；同时，由于 Consumer 的 Rebalance 算法每次都是全部重新计算，使得业务的消费体验也不是很好。存储计算耦合的架构在扩容和故障转移时都需要进行数据搬迁；故障恢复时一般要经历复杂的算法先选举 Leader，且提供服务前要先保证各副本数据是一致的。

## 与 RabbitMQ 对比

与 RabbitMQ 这样的传统消息中间件相比，Kafka 并不适合做 Task Queue。Kafka 将一组消息抽象归纳为一个主题(Topic)，每个主题又被分成一个或多个分区(Partition)。每个分区由一系列有序、不可变的消息组成，是一个有序队列；Kafka 只能保证一个分区之内消息的有序性，并不能保证跨分区消息的有序性。同一个分区的一条消息只能被同一个消费组下某一个消费者消费，但不同消费组的消费者可同时消费该消息。

Kafka Broker 本身并不会记录消息的消费情况，而是交由消费者通过偏移量记录，这样使其非常方便地实现了 "at-least-once" 处理模型；但是也意味着我们并不能随机地获取某条消息。

## Kafka 应用

消息系统或是说消息队列中间件是当前处理大数据一个非常重要的组件，用来解决应用解耦、异步通信、流量控制等问题，从而构建一个高效、灵活、消息同步和异步传输处理、存储转发、可伸缩和最终一致性的稳定系统。当前比较流行的消息中间件有 Kafka、RocketMQ、RabbitMQ、ZeroMQ、ActiveMQ、MetaMQ、Redis 等，这些消息中间件在性能及功能上各有所长。如何选择一个消息中间件取决于我们的业务场景、系统运行环境、开发及运维人员对消息中件间掌握的情况等。我认为在下面这些场景中，Kafka 是一个不错的选择。

- 消息系统：Kafka 作为一款优秀的消息系统，具有高吞吐量、内置的分区、备份冗余分布式等特点，为大规模消息处理提供了一种很好的解决方案。
- 应用监控：利用 Kafka 采集应用程序和服务器健康相关的指标，如 CPU 占用率、IO、内存、连接数、TPS、QPS 等，然后将指标信息进行处理，从而构建一个具有监控仪表盘、曲线图等可视化监控系统。例如，很多公司采用 Kafka 与 ELK(ElasticSearch、Logstash 和 Kibana)整合构建应用服务监控系统。
- 网站用户行为追踪：为了更好地了解用户行为、操作习惯，改善用户体验，进而对产品升级改进，将用户操作轨迹、内容等信息发送到 Kafka 集群上，通过 Hadoop、Spark 或 Strom 等进行数据分析处理，生成相应的统计报告，为推荐系统推荐对象建模提供数据源，进而为每个用户进行个性化推荐。
- 流处理：需要将已收集的流数据提供给其他流式计算框架进行处理，用 Kafka 收集流数据是一个不错的选择，而且当前版本的 Kafka 提供了 Kafka Streams 支持对流数据的处理。
- 持久性日志：Kafka 可以为外部系统提供一种持久性日志的分布式系统。日志可以在多个节点间进行备份，Kafka 为故障节点数据恢复提供了一种重新同步的机制。同时，Kafka 很方便与 HDFS 和 Flume 进行整合，这样就方便将 Kafka 采集的数据持久化到其他外部系统

# Links

- https://mp.weixin.qq.com/s/fX26tCdYSMgwM54_2CpVrw

- https://www.cnblogs.com/huxi2b/p/6223228.html
