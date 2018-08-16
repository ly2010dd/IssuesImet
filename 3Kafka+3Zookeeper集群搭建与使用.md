# Kafka + Zookeeper 集群搭建（3broker 3zookeeper）

## 1.Zookeeper集群
---
### 环境：云服务器环境
- zookeeper1 (101.251.***.126/C#k***)
- zookeeper2 (103.241.***.154/C#k***)
- zookeeper3 (114.112.***.173/C#k***)

### 配置文件
conf/zoo.cfg

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/root/zookeeper-3.4.9/data
dataLogDir=/root/zookeeper-3.4.9/datalog
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
# servers
server.1=101.251.***.126:2888:3888
server.2=103.241.***.154:2888:3888
server.3=114.112.***.173:2888:3888

```

dataDir/myid

```
1 #每个zookeeper的id号，跟server.1对应
```

3个zookeeper都要按上面配置zoo.cfg和myid

### 启动于停止服务

```
bin/zkServer.sh start/stop/status
```
eg: status
```
root@localhost:~/zookeeper-3.4.9# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: follower

```

### 客户端脚本
```
root@localhost:~/zookeeper-3.4.9# bin/zkCli.sh

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
  
[zk: localhost:2181(CONNECTED) 0] ls /
[pms_data, zookeeper, kafka, lock, pms_node]
zk: localhost:2181(CONNECTED) 1] create /test
[zk: localhost:2181(CONNECTED) 2] ls /
[pms_data, zookeeper, kafka, lock, pms_node]
[zk: localhost:2181(CONNECTED) 3] create /test 123
Created /test
[zk: localhost:2181(CONNECTED) 4] ls /            
[pms_data, zookeeper, test, kafka, lock, pms_node]

[zk: localhost:2181(CONNECTED) 5] delete /test
[zk: localhost:2181(CONNECTED) 6] ls /        
[pms_data, zookeeper, kafka, lock, pms_node]

[zk: localhost:2181(CONNECTED) 8] create /test 123      
Created /test
[zk: localhost:2181(CONNECTED) 9] create /test/child 456
Created /test/child
[zk: localhost:2181(CONNECTED) 10] delete /test          
Node not empty: /test
[zk: localhost:2181(CONNECTED) 11] ls /                  
[pms_data, zookeeper, test, kafka, lock, pms_node]
[zk: localhost:2181(CONNECTED) 12] rmr /test             
[zk: localhost:2181(CONNECTED) 13] ls /     
[pms_data, zookeeper, kafka, lock, pms_node]
```



## 2.Kafka集群
---
### 环境：云服务器环境
- kafka1 (101.251.***.125/C#k***)
- kafka2 (103.241.***.155/C#k***)
- kafka3 (101.251.***.94/C#k***)

### 配置文件

config/server.properties
```
############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=1

# Switch to enable topic deletion or not, default value is false
#delete.topic.enable=true

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092
listeners=PLAINTEXT://101.251.***.125:9092 #用于内网通信

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
advertised.listeners=PLAINTEXT://101.251.***.125:9092 #用于外网producer、consumer通信

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma seperated list of directories under which to store log files
log.dirs=/root/sata500g/kafka_data

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to exceessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=24

# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
# segments don't drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000


############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=101.251.***.126:2181,103.***.230.154:2181,***.112.94.173:2181/kafka

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=10000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0

```

config/producer.properties

```
############################# Producer Basics #############################

# list of brokers used for bootstrapping knowledge about the rest of the cluster
# format: host1:port1,host2:port2 ...
bootstrap.servers=101.251.201.125:9092,103.241.230.155:9092,101.251.197.94:9092

# specify the compression codec for all data generated: none, gzip, snappy, lz4
compression.type=none

# name of the partitioner class for partitioning events; default partition spreads data randomly
#partitioner.class=

# the maximum amount of time the client will wait for the response of a request
#request.timeout.ms=

# how long `KafkaProducer.send` and `KafkaProducer.partitionsFor` will block for
#max.block.ms=

# the producer will wait for up to the given delay to allow other records to be sent so that the sends can be batched together
#linger.ms=

# the maximum size of a request in bytes
#max.request.size=

# the default batch size in bytes when batching multiple records sent to a partition
#batch.size=

# the total bytes of memory the producer can use to buffer records waiting to be sent to the server
#buffer.memory=

```

config/consumer.properties

```
# Zookeeper connection string
# comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"
zookeeper.connect=101.251.***.126:2181,103.241.***.154:2181,114.112.**.173:2181/kafka

# timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=10000

#consumer group id
group.id=test-consumer-group

#consumer timeout
#consumer.timeout.ms=5000

```



3台kafka必须都按如上配置brokerid和ip根据自己情况来定


### 启动3台kafka

```
#启动
nohup bin/kafka-server-start.sh config/server.properties &

#新建topic
bin/kafka-topics.sh --create --zookeeper 101.251.***.126:2181,103.241.***.154:2181,114.112.***.173:2181/kafka --replication-factor 3 --partitions 1 --topic test-liy-topic

#查看刚才建立的topic
bin/kafka-topics.sh --describe --zookeeper 101.251.***.126:2181,103.241.***.154:2181,114.112.***.173:2181/kafka --topic test-liy-topic
14.112.94.173:2181/kafka --topic test-liy-topic
Topic:test-liy-topic	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: test-liy-topic	Partition: 0	Leader: 3	Replicas: 2,3,1	Isr: 3,1,2


#启动producer脚本
bin/kafka-console-producer.sh --broker-list 101.251.***.94:9092 --topic test-liy-topic
>liyang
>ceshi
>lalal
注意：此处的broker-list必须为Leader的ip，不能填所有的broker的ip:port。localhost也不行

#启动consumer脚本
bin/kafka-console-consumer.sh --bootstrap-server 101.251.***.125:9092,103.241.***.155:9092,101.251.***.94:9092 --from-beginning --topic test-liy-topic
liyang
test
kafka
3 cluster
.
lalala
liyang
ceshi
lalal

```

