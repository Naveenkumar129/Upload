Example: check if the ServiceAccount can list secrets in its namespace (default here):
curl -s --cacert $CA_CERT \
  -H "Authorization: Bearer $TOKEN" \
  "${APISERVER}/api/v1/namespaces/default/secrets" | jq .
200 OK + JSON list → SA has permission.
403 Forbidden → RBAC issue inside Kubernetes.
401 Unauthorized → Token invalid (maybe expired, wrong mount, or mismatched API server CA).
 

---------

---
Troubleshooting Steps:
Verify ACLs:

Check the current ACLs for the affected topics and consumer groups using the kafka-acls.sh command:
bash
Copy code
kafka-acls.sh --bootstrap-server <broker-address> --list
Look for ACL entries for the User:clrg principal on the affected topics and groups.
Grant Necessary Permissions:

If permissions are missing, grant the required access. For example, to allow DESCRIBE access for a topic:
bash
Copy code
kafka-acls.sh --bootstrap-server <broker-address> \
  --add --allow-principal User:clrg \
  --operation DESCRIBE \
  --topic <topic-name>
For consumer group access:
bash
Copy code
kafka-acls.sh --bootstrap-server <broker-address> \
  --add --allow-principal User:clrg \
  --operation DESCRIBE \
  --group <group-name>
Check Authentication:

Ensure User:clrg is authenticated correctly. Verify the configured SASL mechanism and credentials for the user in Kafka and client applications.
Audit Resource Names:

Confirm that the resource names (Topic, Group) match the intended access pattern. Mistakes in literal resource names (e.g., typos) can cause unnecessary denials.
Review the Kafka Logs:

Examine the broker logs for more context on the denials and confirm the DefaultDeny rule is the root cause.
Configure Logging:

Enable debug logging for Kafka authorization to gather more detailed information:
properties
Copy code
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
log.level=DEBUG
Would you like specific commands or further guidance on implementing these steps?









[# Kafka
https://learn.conduktor.io/kafka/kafka-topic-configuration-log-compaction/

## Getting Started

- Kafka using KRaft

```bash
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
bin/kafka-server-start.sh config/kraft/server.properties

# Create a topic

bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092

# Publish to topic
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092

# Read from topic
bin/kafka-console-consumer.sh \
--topic quickstart-events \
--from-beginning \
--bootstrap-server localhost:9092 \
--property print.offset=true \
--property print.partition=true \
--property print.headers=true \
--property print.timestamp=true \
--property print.key=true
```

## Concepts

- **What purpose do consumer-groups serve**
  - In Apache Kafka, the consumer group concept is a way of achieving two things:
    - Having consumers as part of the same consumer group means providing the "competing consumers" pattern with whom the messages from topic partitions are spread across the members of the group. Each consumer receives messages from one or more partitions ("automatically" assigned to it) and the same messages won't be received by the other consumers (assigned to different partitions). In this way, we can scale the number of the consumers up to the number of the partitions (having one consumer reading only one partition); in this case, a new consumer joining the group will be in an idle state without being assigned to any partition.
    - Having consumers as part of different consumer groups means providing the "publish/subscribe" pattern where the messages from topic partitions are sent to all the consumers across the different groups. It means that inside the same consumer group, we'll have the rules explained above, but across different groups, the consumers will receive the same messages. It's useful when the messages inside a topic are of interest for different applications that will process them in different ways. We want all the interested applications to receive all the same messages from the topic.
  - Another great advantage of consumers grouping is the rebalancing feature. When a consumer joins a group, if there are still enough partitions available (i.e. we haven't reached the limit of one consumer per partition), a re-balancing starts and the partitions will be reassigned to the current consumers, plus the new one. In the same way, if a consumer leaves a group, the partitions will be reassigned to the remaining consumers.

- **Data Replication**
  - e.g. 4 kafka brokers with a replication factor of 3
  - At all times, one broker "owns" a partition and is the node through which applications write/read from the partition. This is called a partition leader. It replicates the data it receives to N other brokers, called followers. They store the data as well and are ready to be elected as leader in case the leader node dies. This helps you configure the guarantee that any successfully published message will not be lost. Having the option to change the replication factor lets you trade performance for stronger durability guarantees, depending on the criticality of the data. In this way, if one leader ever fails, a follower can take his place.

- **Reason for near constant performance of kafka with increasing data**
  - Persistence
    - pagecache
      - Modern OSes cache the disk in free RAM. This is called pagecache.
    - contant time suffices
      - Linear reads/writes on a disk are fast. The concept that modern disks are slow is because of numerous disk seeks, something that is not an issue in big linear operations.
      - Said linear operations are heavily optimized by the OS, via read-ahead (prefetch large block multiples) and write-behind (group small logical writes into big physical writes) techniques.
  - Efficiency
    - pagefile
    - sendfile
      - Since Kafka stores messages in a standardized binary format unmodified throughout the whole flow (producer->broker->consumer), it can make use of the zero-copy optimization. That is when the OS copies data from the pagecache directly to a socket, effectively bypassing the Kafka broker application entirely.
  - End-to-End Batch compression
    - Efficient compression requires compressing multiple messages together rather than compressing each message individually. Kafka supports this with an efficient batching format. A batch of messages can be clumped together compressed and sent to the server in this form. This batch of messages will be written in compressed form and will remain compressed in the log and will only be decompressed by the consumer.
  - Producer
    - Load Balancing
    - Asynchronous Send

- **Topics and Logs**
  - leader
    - Each partition has one server which acts as the leader and zero or more servers which act as followers. The leader handles all read and write requests for the partition while the followers passively replicate the leader. If the leader fails, one of the followers will automatically become the new leader. Each server acts as a leader for some of its partitions and a follower for others so load is well balanced within the cluster
  - replicas is the list of nodes that replicate the log for this partition regardless of whether they are the leader or even if they are currently alive.
  - isr is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader.
  - offset is sequential id assigned to records in a partition that uniquely identifies each record in a partition
  - partition is an immutable collection (or sequence) of messages

- **Partition Log**
  - It's an abstraction which is divided internally by Kafka Broker into segments.
  - Segment
    - Segment are files stored in the filesystem in data directories, with names ending in .log
    - First offset of a segment is called base offset. The segment filename is always equal to the base offset
    - Last segment of a partition is called active segment. Only active segment of a log can receive newly produced message

Kafka broker will merge non-active segments and create a larger segment out of them

## Syntax : Working with secured cluster

- Config file

    ```txt
    security.protocol=SSL
    ssl.keystore.location=/tmp/wxx-xxx.com_keystore.jks
    ssl.keystore.password=XXX
    ssl.key.password=XXX
    ssl.truststore.location=/tmp/wxx-xxx.com_truststore.jks
    ssl.truststore.password=XXXXXX
    session.timeout.ms=360000
    heartbeat.interval.ms=30000
    request.timeout.ms=30000
    ssl.truststore.type=jks
    ssl.keystore.type=jks
    ```

- List topics

    ```txt
    kafka-topics --list --bootstrap-server kafka-816582278-1-1469960372.prod-az-southcentralus-12.prod.us.wxxx.net:9093,kafka-816582278-2-1469960375.prod.us.wxxx.net:9093,kafka-816582278-3-1469960378.prod.us.wxxx.net:9093,kafka-816582278-4-1469960381.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-5-1469960384.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-6-1469960387.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093 --command-config /tmp/config.properties
    ```

- Get `Topic` offsets

    ```txt
    kafka-get-offsets --bootstrap-server kafka-816582278-1-1469960372.prod-az-southcentralus-12.prod.us.wxxx.net:9093,kafka-816582278-2-1469960375.prod.us.wxxx.net:9093,kafka-816582278-3-1469960378.prod.us.wxxx.net:9093,kafka-816582278-4-1469960381.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-5-1469960384.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-6-1469960387.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093 --command-config /tmp/config.properties --topic kafka-gg-tns-sc
    ```

- Get `Consumer Group` offset details using consumer-groups library

    ```txt
    kafka-consumer-groups --bootstrap-server kafka-816582278-1-1469960372.prod-az-southcentralus-12.prod.us.wxxx.net:9093,kafka-816582278-2-1469960375.prod.us.wxxx.net:9093,kafka-816582278-3-1469960378.prod.us.wxxx.net:9093,kafka-816582278-4-1469960381.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-5-1469960384.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-6-1469960387.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093  --timeout 100000 --group spark-kafka-source-a70fadb6-8b50-402d-8c20-4528fc98167f-16033791-driver-0  --describe --command-config /tmp/config.properties
    ```

- Get consumer groups list

    ```txt

    kafka-consumer-groups --bootstrap-server kafka-816582278-2-1469960375.prod-az-southcentralus-12.prod.us.wxxx.net:9093,kafka-816582278-3-1469960378.prod.us.wxxx.net:9093,kafka-816582278-4-1469960381.prod.us.wxxx.net:9093,kafka-816582278-5-1469960384.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-6-1469960387.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093 --command-config /tmp/config.properties  --list --timeout 100000
    ```

- Read messages from kafka topic

    ```txt
    kafka-console-consumer --bootstrap-server kafka-816582278-1-1469960372.prod-az-southcentralus-12.prod.us.wxxx.net:9093,kafka-816582278-2-1469960375.prod.us.wxxx.net:9093,kafka-816582278-3-1469960378.prod.us.wxxx.net:9093,kafka-816582278-4-1469960381.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-5-1469960384.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093,kafka-816582278-6-1469960387.scus.kafka-v2-gg-tns-prod.ms-df-messaging.prod-az-southcentralus-12.prod.us.walmart.net:9093  --topic kafka-gg-tns-sc --consumer.config /tmp/config.properties --group spark-kafka-source-5f2461be-9fe3-42d9-9632-116da3d30d30--2069994981-driver-0
    ```

## Syntax : Non Secured Cluster

- Get `Consumer Group` offset Value

    ```txt
    kafka-consumer-groups --bootstrap-server kafka-498637915-1-1251069382.scus.kafka-v2-usgm-shared-stg.prod.us.wxxx.net:9092,kafka-498637915-2-1251069385.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.wxxx.net:9092,kafka-498637915-3-1251069388.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.wxxx.net:9092,kafka-498637915-4-1251069391.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-5-1251069394.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-6-1251069397.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-7-1251069400.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-8-1251069403.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-9-1251069406.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092  --timeout 100000 --group spark-consumer-sxxx-01  --describe
    ```

- Reset offset value

    ```txt
    kafka-consumer-groups --bootstrap-server kafka-498637915-1-1251069382.scus.kafka-v2-usgm-shared-stg.prod.us.wxxx.net:9092,kafka-498637915-2-1251069385.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.wxxx.net:9092,kafka-498637915-3-1251069388.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.wxxx.net:9092,kafka-498637915-4-1251069391.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-5-1251069394.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-6-1251069397.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-7-1251069400.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-8-1251069403.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-9-1251069406.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092  --topic snw-hyperloop-outbound-response --reset-offsets --to-earliest --group spark-consumer-sxxx-01 --execute
    ```

- Read messages from kafka topic

    ```txt
    kafka-console-consumer --bootstrap-server kafka-498637915-1-1251069382.scus.kafka-v2-usgm-shared-stg.prod.us.wxxx.net:9092,kafka-498637915-2-1251069385.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.wxxx.net:9092,kafka-498637915-3-1251069388.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.wxxx.net:9092,kafka-498637915-4-1251069391.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-5-1251069394.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-6-1251069397.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-7-1251069400.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-8-1251069403.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092,kafka-498637915-9-1251069406.scus.kafka-v2-usgm-shared-stg.ms-df-messaging.stg-az-southcentralus-2.prod.us.walmart.net:9092  --topic snw-hyperloop-outbound-response --group spark-consumer-sxxx-06
    ```


Po
Clearty, managing offsets has a big impact on the client application. The Ka £kaConsumex API provides multiple ways of committing offsets.
Last committed offset
Automatic Commit
The easiest way to commit offsets is to allow the consumer to do it for you. Ifyou configure enable.auto.commit-true, then every five: seconds the consumer willl commit the largest offset your client received from poli0. The five-second nterval is the default and control led by setting auto.commitintenval.r.ms. Just like everything else nthe consumer, the automatic commits are driven by the polll oop. Whenever you r poll, the consumer checks ifit is time to commit, and ifit is, it wll commit the offsets it retumned in the last poll.
 Before using this convenient option, however, it is important to understand the consequences, Consider that, by defaulit, automatic commits occur every five seconds. Suppose that we are three seconds after the most recent commit and a rebalance is triggered. After the rebalancing, all consumers will start consuming from the last offset committed. In this case, the offset is three seconds old, so all the events that amrived in those three seconds will be processed twice. It is possible to configure the commit interval to commit more frequently and reduce the window in which records will be duplicated, but it is impossible to completely eliminate them. With autocommit enabled, a call to poll will always commit the last offset retumed by the previous poll. It doesn't know which events were actually processed, so it is critical to always process all the events retumed by poll0 before calling poll0 again. (Just like poll0, close0 also commits offsets automatically.) This is usual Inot an issue, but pay attention when you handle exceptions or exit the poll loop prematurely. Automatic commits are convenient, but they don't give developers enough control to avoid duplicate messages.
 Asynchronous Commit One drawback of manua commit is that the application is blocked until the broker responds to the commit request This willimit the throughput of the application. Throughput can be improved by committing Iess frequently, but then we are increasing the number of potential duplicates that a rebalance will create. Another option is the asynchronous commit API. Instead of waiting for the broker to respond to a commit, we just send the request and continue on. The drawback is that while commitSync0 will retry the commit until it either succeeds or encounters a nonretriable failure, commitAsync0 will not retry. The reason it does not retry is that by the time commitAsync0 receives a response from the server, there may have been a ater commit that was aready successful. Imagine that we sent a request to commit offset 2000. There is a temporary communication problem, sO the broker never gets the request and therefore never responds. Meanwhile, we processed another batch and successfully committed offset 3000. If commitAsync now retries the previously failed commit, it might succeed i committing offset 2000 after offset 3000 was already processed and committed. In the case of a rebalance, this will cause more duplicates
 Cle
Type here to search
Be the frs: to like this
- Rolling Counter
  - `<command> | awk 'BEGIN{print "counter";m=0}{if($1~/(3|6)/)m++;print NR}'`

## Interview Questions

- **How does Kafka determine when to create new segment file in a Partition Log?**
  - **Solution** : This is controlled based upon either of the following 2 properties
    - By setting `segment.bytes` (default is 1GB) config during creation of topic. When your segment size becomes bigger than this value, a new active segment is created
    - Or by setting `segment.ms` config parameter. With this, when Kafka receives a new produce request, it will check if the active segment is older than `segment.ms` value. if yes, then it will create a new segment. This value is in milliseconds.
      - With smaller value for `segment.ms`, small files will be created, which after log compaction will be merged into larger files.

- **How does a producer/consumer know who the leader of  a partition is**
  - Producer and Consumers used to directly connect and talk to Zookeeper to get this (and other) information. Kafka has been moving away from this coupling and since versions 0.8 and 0.9 respectively, clients fetch metadata information from Kafka brokers directly, who themselves talk to Zookeeper.

## References

- <https://confluence.wxxx.com/display/STRMOPS/Apache+Kafka%3A+Best+Practices>
