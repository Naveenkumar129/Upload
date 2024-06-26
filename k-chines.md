[## 3.3、指标说明]
(https://doc.knowstreaming.com/study-kafka-old/3-common-exceptions-and-resolutions#335-kafka-%E6%9F%90%E4%B8%AA%E8%8A%82%E7%82%B9-cpu-%E4%BD%BF%E7%94%A8%E7%8E%87%E5%BE%88%E9%AB%98)

当前 KnowStreaming 支持针对 kafka 集群的多维度指标的采集和展示，同时也支持多个 kafka 版本的指标进行兼容，以下是 KnowStreaming 支持的指标说明。

现在对当前 KnowStreaming 支持的指标从指标名称、指标单位、指标说明、kafka 版本、企业/开源版指标 五个维度进行说明。

### 3.3.1、Cluster 指标

| 指标名称                  | 指标单位 | 指标含义                           | kafka 版本       | 企业/开源版指标 |
| ------------------------- | -------- |--------------------------------| ---------------- | --------------- |
| HealthScore               | 分       | 集群总体的健康分                       | 全部版本         | 开源版          |
| HealthCheckPassed         | 个       | 集群总体健康检查通过数                    | 全部版本         | 开源版          |
| HealthCheckTotal          | 个       | 集群总体健康检查总数                     | 全部版本         | 开源版          |
| HealthScore_Topics        | 分       | 集群 Topics 的健康分                 | 全部版本         | 开源版          |
| HealthCheckPassed_Topics  | 个       | 集群 Topics 健康检查通过数              | 全部版本         | 开源版          |
| HealthCheckTotal_Topics   | 个       | 集群 Topics 健康检查总数               | 全部版本         | 开源版          |
| HealthScore_Brokers       | 分       | 集群 Brokers 的健康分                | 全部版本         | 开源版          |
| HealthCheckPassed_Brokers | 个       | 集群 Brokers 健康检查通过数             | 全部版本         | 开源版          |
| HealthCheckTotal_Brokers  | 个       | 集群 Brokers 健康检查总数              | 全部版本         | 开源版          |
| HealthScore_Groups        | 分       | 集群 Groups 的健康分                 | 全部版本         | 开源版          |
| HealthCheckPassed_Groups  | 个       | 集群 Groups 健康检查总数               | 全部版本         | 开源版          |
| HealthCheckTotal_Groups   | 个       | 集群 Groups 健康检查总数               | 全部版本         | 开源版          |
| HealthScore_Cluster       | 分       | 集群自身的健康分                       | 全部版本         | 开源版          |
| HealthCheckPassed_Cluster | 个       | 集群自身健康检查通过数                    | 全部版本         | 开源版          |
| HealthCheckTotal_Cluster  | 个       | 集群自身健康检查总数                     | 全部版本         | 开源版          |
| TotalRequestQueueSize     | 个       | 集群中总的请求队列数                     | 全部版本         | 开源版          |
| TotalResponseQueueSize    | 个       | 集群中总的响应队列数                     | 全部版本         | 开源版          |
| EventQueueSize            | 个       | 集群中 Controller 的 EventQueue 大小 | 2.0.0 及以上版本 | 开源版          |
| ActiveControllerCount     | 个       | 集群中存活的 Controller 数            | 全部版本         | 开源版          |
| TotalProduceRequests      | 个       | 集群中的 Produce 每秒请求数             | 全部版本         | 开源版          |
| TotalLogSize              | byte     | 集群总的已使用的磁盘大小                   | 全部版本         | 开源版          |
| ConnectionsCount          | 个       | 集群的连接(Connections)个数           | 全部版本         | 开源版          |
| Zookeepers                | 个       | 集群中存活的 zk 节点个数                 | 全部版本         | 开源版          |
| ZookeepersAvailable       | 是/否    | ZK 地址是否合法                      | 全部版本         | 开源版          |
| Brokers                   | 个       | 集群的 broker 的总数                 | 全部版本         | 开源版          |
| BrokersAlive              | 个       | 集群的 broker 的存活数                | 全部版本         | 开源版          |
| BrokersNotAlive           | 个       | 集群的 broker 的未存活数               | 全部版本         | 开源版          |
| Replicas                  | 个       | 集群中 Replica 的总数                | 全部版本         | 开源版          |
| Topics                    | 个       | 集群中 Topic 的总数                  | 全部版本         | 开源版          |
| Partitions                | 个       | 集群的 Partitions 总数              | 全部版本         | 开源版          |
| PartitionNoLeader         | 个       | 集群中的 PartitionNoLeader 总数      | 全部版本         | 开源版          |
| PartitionMinISR_S         | 个       | 集群中的小于 PartitionMinISR 总数      | 全部版本         | 开源版          |
| PartitionMinISR_E         | 个       | 集群中的等于 PartitionMinISR 总数      | 全部版本         | 开源版          |
| PartitionURP              | 个       | 集群中的未同步的 Partition 总数          | 全部版本         | 开源版          |
| MessagesIn                | 条/s     | 集群每秒消息写入条数                     | 全部版本         | 开源版          |
| Messages                  | 条       | 集群总的消息条数                       | 全部版本         | 开源版          |
| LeaderMessages            | 条       | 集群中 leader 总的消息条数              | 全部版本         | 开源版          |
| BytesIn                   | byte/s   | 集群的每秒写入字节数                     | 全部版本         | 开源版          |
| BytesIn_min_5             | byte/s   | 集群的每秒写入字节数，5 分钟均值              | 全部版本         | 开源版          |
| BytesIn_min_15            | byte/s   | 集群的每秒写入字节数，15 分钟均值             | 全部版本         | 开源版          |
| BytesOut                  | byte/s   | 集群的每秒流出字节数                     | 全部版本         | 开源版          |
| BytesOut_min_5            | byte/s   | 集群的每秒流出字节数，5 分钟均值              | 全部版本         | 开源版          |
| BytesOut_min_15           | byte/s   | 集群的每秒流出字节数，15 分钟均值             | 全部版本         | 开源版          |
| Groups                    | 个       | 集群中 Group 的总数                  | 全部版本         | 开源版          |
| GroupActives              | 个       | 集群中 ActiveGroup 的总数            | 全部版本         | 开源版          |
| GroupEmptys               | 个       | 集群中 EmptyGroup 的总数             | 全部版本         | 开源版          |
| GroupRebalances           | 个       | 集群中 RebalanceGroup 的总数         | 全部版本         | 开源版          |
| GroupDeads                | 个       | 集群中 DeadGroup 的总数              | 全部版本         | 开源版          |
| Alive                     | 是/否    | 集群是否存活，1：存活；0：没有存活             | 全部版本         | 开源版          |
| AclEnable                 | 是/否    | 集群是否开启 Acl，1：是；0：否             | 全部版本         | 开源版          |
| Acls                      | 个       | ACL 数                          | 全部版本         | 开源版          |
| AclUsers                  | 个       | ACL-KafkaUser 数                | 全部版本         | 开源版          |
| AclTopics                 | 个       | ACL-Topic 数                    | 全部版本         | 开源版          |
| AclGroups                 | 个       | ACL-Group 数                    | 全部版本         | 开源版          |
| Jobs                      | 个       | 集群任务总数                         | 全部版本         | 开源版          |
| JobsRunning               | 个       | 集群 running 任务总数                | 全部版本         | 开源版          |
| JobsWaiting               | 个       | 集群 waiting 任务总数                | 全部版本         | 开源版          |
| JobsSuccess               | 个       | 集群 success 任务总数                | 全部版本         | 开源版          |
| JobsFailed                | 个       | 集群 failed 任务总数                 | 全部版本         | 开源版          |
| LoadReBalanceEnable       | 是/否    | 是否开启均衡, 1：是；0：否                | 全部版本         | 企业版          |
| LoadReBalanceCpu          | 是/否    | CPU 是否均衡, 1：是；0：否              | 全部版本         | 企业版          |
| LoadReBalanceNwIn         | 是/否    | BytesIn 是否均衡, 1：是；0：否          | 全部版本         | 企业版          |
| LoadReBalanceNwOut        | 是/否    | BytesOut 是否均衡, 1：是；0：否         | 全部版本         | 企业版          |
| LoadReBalanceDisk         | 是/否    | Disk 是否均衡, 1：是；0：否             | 全部版本         | 企业版          |

### 3.3.2、Broker 指标

| 指标名称                | 指标单位 | 指标含义                              | kafka 版本 | 企业/开源版指标 |
| ----------------------- | -------- | ------------------------------------- | ---------- | --------------- |
| HealthScore             | 分       | Broker 健康分                         | 全部版本   | 开源版          |
| HealthCheckPassed       | 个       | Broker 健康检查通过数                 | 全部版本   | 开源版          |
| HealthCheckTotal        | 个       | Broker 健康检查总数                   | 全部版本   | 开源版          |
| TotalRequestQueueSize   | 个       | Broker 的请求队列大小                 | 全部版本   | 开源版          |
| TotalResponseQueueSize  | 个       | Broker 的应答队列大小                 | 全部版本   | 开源版          |
| ReplicationBytesIn      | byte/s   | Broker 的副本流入流量                 | 全部版本   | 开源版          |
| ReplicationBytesOut     | byte/s   | Broker 的副本流出流量                 | 全部版本   | 开源版          |
| MessagesIn              | 条/s     | Broker 的每秒消息流入条数             | 全部版本   | 开源版          |
| TotalProduceRequests    | 个/s     | Broker 上 Produce 的每秒请求数        | 全部版本   | 开源版          |
| NetworkProcessorAvgIdle | %        | Broker 的网络处理器的空闲百分比       | 全部版本   | 开源版          |
| RequestHandlerAvgIdle   | %        | Broker 上请求处理器的空闲百分比       | 全部版本   | 开源版          |
| PartitionURP            | 个       | Broker 上的未同步的副本的个数         | 全部版本   | 开源版          |
| ConnectionsCount        | 个       | Broker 上网络链接的个数               | 全部版本   | 开源版          |
| BytesIn                 | byte/s   | Broker 的每秒数据写入量               | 全部版本   | 开源版          |
| BytesIn_min_5           | byte/s   | Broker 的每秒数据写入量，5 分钟均值   | 全部版本   | 开源版          |
| BytesIn_min_15          | byte/s   | Broker 的每秒数据写入量，15 分钟均值  | 全部版本   | 开源版          |
| BytesOut                | byte/s   | Broker 的每秒数据流出量               | 全部版本   | 开源版          |
| BytesOut_min_5          | byte/s   | Broker 的每秒数据流出量，5 分钟均值   | 全部版本   | 开源版          |
| BytesOut_min_15         | byte/s   | Broker 的每秒数据流出量，15 分钟均值  | 全部版本   | 开源版          |
| ReassignmentBytesIn     | byte/s   | Broker 的每秒数据迁移写入量           | 全部版本   | 开源版          |
| ReassignmentBytesOut    | byte/s   | Broker 的每秒数据迁移流出量           | 全部版本   | 开源版          |
| Partitions              | 个       | Broker 上的 Partition 个数            | 全部版本   | 开源版          |
| PartitionsSkew          | %        | Broker 上的 Partitions 倾斜度         | 全部版本   | 开源版          |
| Leaders                 | 个       | Broker 上的 Leaders 个数              | 全部版本   | 开源版          |
| LeadersSkew             | %        | Broker 上的 Leaders 倾斜度            | 全部版本   | 开源版          |
| LogSize                 | byte     | Broker 上的消息容量大小               | 全部版本   | 开源版          |
| Alive                   | 是/否    | Broker 是否存活，1：存活；0：没有存活 | 全部版本   | 开源版          |

### 3.3.3、Topic 指标

| 指标名称              | 指标单位 | 指标含义                              | kafka 版本 | 企业/开源版指标 |
| --------------------- | -------- | ------------------------------------- | ---------- | --------------- |
| HealthScore           | 分       | 健康分                                | 全部版本   | 开源版          |
| HealthCheckPassed     | 个       | 健康项检查通过数                      | 全部版本   | 开源版          |
| HealthCheckTotal      | 个       | 健康项检查总数                        | 全部版本   | 开源版          |
| TotalProduceRequests  | 条/s     | Topic 的 TotalProduceRequests         | 全部版本   | 开源版          |
| BytesRejected         | 个/s     | Topic 的每秒写入拒绝量                | 全部版本   | 开源版          |
| FailedFetchRequests   | 个/s     | Topic 的 FailedFetchRequests          | 全部版本   | 开源版          |
| FailedProduceRequests | 个/s     | Topic 的 FailedProduceRequests        | 全部版本   | 开源版          |
| ReplicationCount      | 个       | Topic 总的副本数                      | 全部版本   | 开源版          |
| Messages              | 条       | Topic 总的消息数                      | 全部版本   | 开源版          |
| MessagesIn            | 条/s     | Topic 每秒消息条数                    | 全部版本   | 开源版          |
| BytesIn               | byte/s   | Topic 每秒消息写入字节数              | 全部版本   | 开源版          |
| BytesIn_min_5         | byte/s   | Topic 每秒消息写入字节数，5 分钟均值  | 全部版本   | 开源版          |
| BytesIn_min_15        | byte/s   | Topic 每秒消息写入字节数，15 分钟均值 | 全部版本   | 开源版          |
| BytesOut              | byte/s   | Topic 每秒消息流出字节数              | 全部版本   | 开源版          |
| BytesOut_min_5        | byte/s   | Topic 每秒消息流出字节数，5 分钟均值  | 全部版本   | 开源版          |
| BytesOut_min_15       | byte/s   | Topic 每秒消息流出字节数，15 分钟均值 | 全部版本   | 开源版          |
| LogSize               | byte     | Topic 的大小                          | 全部版本   | 开源版          |
| PartitionURP          | 个       | Topic 未同步的副本数                  | 全部版本   | 开源版          |

### 3.3.4、Partition 指标

| 指标名称       | 指标单位 | 指标含义                                  | kafka 版本 | 企业/开源版指标 |
| -------------- | -------- | ----------------------------------------- | ---------- | --------------- |
| LogEndOffset   | 条       | Partition 中 leader 副本的 LogEndOffset   | 全部版本   | 开源版          |
| LogStartOffset | 条       | Partition 中 leader 副本的 LogStartOffset | 全部版本   | 开源版          |
| Messages       | 条       | Partition 总的消息数                      | 全部版本   | 开源版          |
| BytesIn        | byte/s   | Partition 的每秒消息流入字节数            | 全部版本   | 开源版          |
| BytesOut       | byte/s   | Partition 的每秒消息流出字节数            | 全部版本   | 开源版          |
| LogSize        | byte     | Partition 的大小                          | 全部版本   | 开源版          |

### 3.3.5、Group 指标

| 指标名称          | 指标单位 | 指标含义                   | kafka 版本 | 企业/开源版指标 |
| ----------------- | -------- | -------------------------- | ---------- | --------------- |
| HealthScore       | 分       | 健康分                     | 全部版本   | 开源版          |
| HealthCheckPassed | 个       | 健康检查通过数             | 全部版本   | 开源版          |
| HealthCheckTotal  | 个       | 健康检查总数               | 全部版本   | 开源版          |
| OffsetConsumed    | 条       | Consumer 的 CommitedOffset | 全部版本   | 开源版          |
| LogEndOffset      | 条       | Consumer 的 LogEndOffset   | 全部版本   | 开源版          |
| Lag               | 条       | Group 消费者的 Lag 数      | 全部版本   | 开源版          |
| State             | 个       | Group 组的状态             | 全部版本   | 开源版          |
