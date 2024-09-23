# Apache Hudi: Common Issues and Their Solutions

## java.lang.NoSuchMethodError: 'org.apache.hadoop.hdfs.DFSInputStream$ReadStatistics org.apache.hadoop.hdfs.client.HdfsDataInputStream.getReadStatistics()'

**Details:**

* Spark Version: 3.5.1
* Hudi Version: 0.15.0 (Internally uses HBase 2.4.13)

```sh
export SPARK_VERSION=3.5.1
HUDI_VERSION=0.15.0
git clone --branch release-${HUDI_VERSION} https://github.com/apache/hbase.git

HBASE_VERSION=$(cat hbase/pom.xml | grep '<hbase.version>' | sed 's/.*>\(.*\)<.*/\1/')
git clone --branch rel/${HBASE_VERSION} https://github.com/apache/hbase.git
cd hbase
mvn clean install -Denforcer.skip -DskipTests -Dhadoop.profile=3.0 -Psite-install-step -U

cd ../hudi

SPARK_MAJOR_VERSION=$(echo "${SPARK_VERSION}" | grep -Eo '^[0-9]+\.[0-9]*')
mvn clean package -DskipTests -Dspark${SPARK_MAJOR_VERSION} -U
```
## WARN "Unable to get Instrumentation. Dynamic Attach failed" while booting SDC service.

**Solution:**

Use java8 version

or

```sh
export JAVA_TOOL_OPTIONS="-Djdk.attach.allowAttachSelf=true"
```

## java.lang.NoSuchMethodError: java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer;

```sh
org.apache.spark.SparkException: Job aborted due to stage failure: Task serialization failed: java.lang.NoSuchMethodError: java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer;
java.lang.NoSuchMethodError: java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer;
	at org.apache.spark.util.io.ChunkedByteBufferOutputStream.toChunkedByteBuffer(ChunkedByteBufferOutputStream.scala:115)
	at org.apache.spark.broadcast.TorrentBroadcast$.blockifyObject(TorrentBroadcast.scala:369)
	at org.apache.spark.broadcast.TorrentBroadcast.writeBlocks(TorrentBroadcast.scala:161)
	at org.apache.spark.broadcast.TorrentBroadcast.<init>(TorrentBroadcast.scala:99)
	at org.apache.spark.broadcast.TorrentBroadcastFactory.newBroadcast(TorrentBroadcastFactory.scala:38)
	at org.apache.spark.broadcast.BroadcastManager.newBroadcast(BroadcastManager.scala:78)
	at org.apache.spark.SparkContext.broadcastInternal(SparkContext.scala:1657)
	at org.apache.spark.SparkContext.broadcast(SparkContext.scala:1639)
	at org.apache.spark.scheduler.DAGScheduler.submitMissingTasks(DAGScheduler.scala:1585)
	at org.apache.spark.scheduler.DAGScheduler.submitStage(DAGScheduler.scala:1402)
	at org.apache.spark.scheduler.DAGScheduler.handleJobSubmitted(DAGScheduler.scala:1337)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.doOnReceive(DAGScheduler.scala:3003)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:2994)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:2983)
	at org.apache.spark.util.EventLoop$$anon$1.run(EventLoop.scala:49)
```

**Solution:**

Use Java11 or higher version

## Unable to connect hudi from Hive CLI

When using the Hive CLI to connect to Hive and query the corresponding Hive table mapped by Hudi, the following exception is found:

```sql
hive> select * from test_table;
FAILED: RuntimeException java.lang.ClassNotFoundException: org.apache.hudi.hadoop.HoodieParquetInputFormat
```

**Solution:**

Put Hudi `hudi-hadoop-mr-bundle` dependencies under $HIVE_HOME/auxlib and restart hivemetastore and hiveserver2. We can find this jar under `packaging/hudi-hadoop-mr-bundle/target` folder.


**Reference:**
* https://hudi.apache.org/docs/syncing_metastore/#hive-environment

