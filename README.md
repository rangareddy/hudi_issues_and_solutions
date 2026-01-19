# Apache Hudi: Common Issues and Their Solutions

## java.lang.NoSuchMethodError: 'org.apache.hadoop.hdfs.DFSInputStream$ReadStatistics org.apache.hadoop.hdfs.client.HdfsDataInputStream.getReadStatistics()'

**Details:**

* Spark Version: 3.5.1
* Hudi Version: 0.15.0 (Internally uses HBase 2.4.13)

**Problem:**

1. Hudi 0.13.0 enables metadata table by default, while previous versions do not. ( HoodieMetadataConfig)
2. The metadata table is stored in HBase's HFile format.
3. Hudi 0.15.0 depends on HBase 2.4.13 and HBase 2.4.13 depends on Hadoop 2.x by default. This is inconsistent with the Hadoop 3.x version we use, and there is a compatibility issue, resulting in a dependency conflict: NoSuchMethodError.

**Solution:**

**Temporary Solution:**

Disabling the metadata

```sh
hoodie.metadata.enable=false
```

**Permanent Solution:**

1. Build the HBase by specifying Hadoop 3 version first.
2. Then ReBuild the Hudi.

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

## Caused by: java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD

```java
org.apache.hudi.exception.HoodieUpsertException: Failed to upsert for commit time 20250513072207750
	at org.apache.hudi.table.action.commit.BaseWriteHelper.write(BaseWriteHelper.java:70)
	at org.apache.hudi.table.action.commit.SparkUpsertCommitActionExecutor.execute(SparkUpsertCommitActionExecutor.java:44)
	at org.apache.hudi.table.HoodieSparkCopyOnWriteTable.upsert(HoodieSparkCopyOnWriteTable.java:114)
	at org.apache.hudi.table.HoodieSparkCopyOnWriteTable.upsert(HoodieSparkCopyOnWriteTable.java:103)
	at org.apache.hudi.client.SparkRDDWriteClient.upsert(SparkRDDWriteClient.java:142)
	at org.apache.hudi.DataSourceUtils.doWriteOperation(DataSourceUtils.java:224)
	at org.apache.hudi.HoodieSparkSqlWriterInternal.liftedTree1$1(HoodieSparkSqlWriter.scala:504)
	at org.apache.hudi.HoodieSparkSqlWriterInternal.writeInternal(HoodieSparkSqlWriter.scala:502)
	at org.apache.hudi.HoodieSparkSqlWriterInternal.write(HoodieSparkSqlWriter.scala:204)
	at org.apache.hudi.HoodieSparkSqlWriter$.write(HoodieSparkSqlWriter.scala:121)
	at org.apache.hudi.DefaultSource.createRelation(DefaultSource.scala:150)
	at org.apache.spark.sql.execution.datasources.SaveIntoDataSourceCommand.run(SaveIntoDataSourceCommand.scala:47)
	at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult$lzycompute(commands.scala:75)
	at org.apache.spark.sql.execution.command.ExecutedCommandExec.sideEffectResult(commands.scala:73)
	at org.apache.spark.sql.execution.command.ExecutedCommandExec.executeCollect(commands.scala:84)
	at org.apache.spark.sql.execution.QueryExecution$$anonfun$eagerlyExecuteCommands$1.$anonfun$applyOrElse$1(QueryExecution.scala:98)
	at org.apache.spark.sql.execution.SQLExecution$.$anonfun$withNewExecutionId$6(SQLExecution.scala:118)
	at org.apache.spark.sql.execution.SQLExecution$.withSQLConfPropagated(SQLExecution.scala:195)
	at org.apache.spark.sql.execution.SQLExecution$.$anonfun$withNewExecutionId$1(SQLExecution.scala:103)
	at org.apache.spark.sql.SparkSession.withActive(SparkSession.scala:827)
	at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:65)
	at org.apache.spark.sql.execution.QueryExecution$$anonfun$eagerlyExecuteCommands$1.applyOrElse(QueryExecution.scala:98)
	at org.apache.spark.sql.execution.QueryExecution$$anonfun$eagerlyExecuteCommands$1.applyOrElse(QueryExecution.scala:94)
	at org.apache.spark.sql.catalyst.trees.TreeNode.$anonfun$transformDownWithPruning$1(TreeNode.scala:512)
	at org.apache.spark.sql.catalyst.trees.CurrentOrigin$.withOrigin(TreeNode.scala:104)
	at org.apache.spark.sql.catalyst.trees.TreeNode.transformDownWithPruning(TreeNode.scala:512)
	at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan.org$apache$spark$sql$catalyst$plans$logical$AnalysisHelper$$super$transformDownWithPruning(LogicalPlan.scala:31)
	at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper.transformDownWithPruning(AnalysisHelper.scala:267)
	at org.apache.spark.sql.catalyst.plans.logical.AnalysisHelper.transformDownWithPruning$(AnalysisHelper.scala:263)
	at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan.transformDownWithPruning(LogicalPlan.scala:31)
	at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan.transformDownWithPruning(LogicalPlan.scala:31)
	at org.apache.spark.sql.catalyst.trees.TreeNode.transformDown(TreeNode.scala:488)
	at org.apache.spark.sql.execution.QueryExecution.eagerlyExecuteCommands(QueryExecution.scala:94)
	at org.apache.spark.sql.execution.QueryExecution.commandExecuted$lzycompute(QueryExecution.scala:81)
	at org.apache.spark.sql.execution.QueryExecution.commandExecuted(QueryExecution.scala:79)
	at org.apache.spark.sql.execution.QueryExecution.assertCommandExecuted(QueryExecution.scala:133)
	at org.apache.spark.sql.DataFrameWriter.runCommand(DataFrameWriter.scala:856)
	at org.apache.spark.sql.DataFrameWriter.saveToV1Source(DataFrameWriter.scala:387)
	at org.apache.spark.sql.DataFrameWriter.saveInternal(DataFrameWriter.scala:360)
	at org.apache.spark.sql.DataFrameWriter.save(DataFrameWriter.scala:239)
	....
Caused by: org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 2.0 failed 4 times, most recent failure: Lost task 0.3 in stage 2.0 (TID 6) (172.18.0.11 executor 1): java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2301)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1431)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2437)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2355)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2213)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1669)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2431)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2355)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2213)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1669)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:503)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:461)
	at scala.collection.immutable.List$SerializationProxy.readObject(List.scala:527
	....
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:87)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:53)
	at org.apache.spark.TaskContext.runTaskWithListeners(TaskContext.scala:161)
	at org.apache.spark.scheduler.Task.run(Task.scala:139)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:554)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1529)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:557)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
```

The above issue mainly occurred in Spark Standalone or Kubernetes clusters.

**Solution:**

Copy all required jars to the `$SPARK_HOME/jars/` directory or add the jars by specifying them as follows:

```python
.setJars(new String[]{"jar_path/sample.jar"})
```

## java.lang.ClassNotFoundException: org.apache.spark.sql.execution.datasources.Spark35NestedSchemaPruning

**Exception:**

```java
Exception in thread "main" org.apache.hudi.exception.HoodieException: Unable to load class
        at org.apache.hudi.common.util.ReflectionUtils.lambda$getClass$0(ReflectionUtils.java:55)
        at java.base/java.util.concurrent.ConcurrentHashMap.computeIfAbsent(ConcurrentHashMap.java:1737)
        at org.apache.hudi.common.util.ReflectionUtils.getClass(ReflectionUtils.java:51)
        at org.apache.hudi.common.util.ReflectionUtils.loadClass(ReflectionUtils.java:62)
        at org.apache.spark.sql.hudi.analysis.HoodieAnalysis$.customOptimizerRules(HoodieAnalysis.scala:179)
        at org.apache.spark.sql.hudi.HoodieSparkSessionExtension.apply(HoodieSparkSessionExtension.scala:43)
        at org.apache.spark.sql.hudi.HoodieSparkSessionExtension.apply(HoodieSparkSessionExtension.scala:28)
        at org.apache.spark.sql.SparkSession$.$anonfun$applyExtensions$2(SparkSession.scala:1370)
        at org.apache.spark.sql.SparkSession$.$anonfun$applyExtensions$2$adapted(SparkSession.scala:1365)
        at scala.collection.mutable.ResizableArray.foreach(ResizableArray.scala:62)
        at scala.collection.mutable.ResizableArray.foreach$(ResizableArray.scala:55)
        at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:49)
        at org.apache.spark.sql.SparkSession$.org$apache$spark$sql$SparkSession$$applyExtensions(SparkSession.scala:1365)
        at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:1104)
        at org.apache.spark.sql.SQLContext$.getOrCreate(SQLContext.scala:1023)
        at org.apache.spark.sql.SQLContext.getOrCreate(SQLContext.scala)
        at org.apache.hudi.client.common.HoodieSparkEngineContext.<init>(HoodieSparkEngineContext.java:72)
        at org.apache.hudi.utilities.streamer.HoodieStreamer.<init>(HoodieStreamer.java:166)
        at org.apache.hudi.utilities.streamer.HoodieStreamer.<init>(HoodieStreamer.java:150)
        at org.apache.hudi.utilities.streamer.HoodieStreamer.<init>(HoodieStreamer.java:136)
        at org.apache.hudi.utilities.streamer.HoodieStreamer.main(HoodieStreamer.java:606)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:566)
        at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
        at org.apache.spark.deploy.SparkSubmit.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:1032)
        at org.apache.spark.deploy.SparkSubmit.doRunMain$1(SparkSubmit.scala:194)
        at org.apache.spark.deploy.SparkSubmit.submit(SparkSubmit.scala:217)
        at org.apache.spark.deploy.SparkSubmit.doSubmit(SparkSubmit.scala:91)
        at org.apache.spark.deploy.SparkSubmit$$anon$2.doSubmit(SparkSubmit.scala:1124)
        at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:1133)
        at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
Caused by: java.lang.ClassNotFoundException: org.apache.spark.sql.execution.datasources.Spark35NestedSchemaPruning
        at java.base/java.net.URLClassLoader.findClass(URLClassLoader.java:476)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:594)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:527)
        at java.base/java.lang.Class.forName0(Native Method)
        at java.base/java.lang.Class.forName(Class.java:315)
        at org.apache.hudi.common.util.ReflectionUtils.lambda$getClass$0(ReflectionUtils.java:53)
        ... 32 more
```

**Problem Statement:**

The `java.lang.ClassNotFoundException: org.apache.spark.sql.execution.datasources.Spark35NestedSchemaPruning` error indicates that the Hudi Spark integration is attempting to use a Spark 3.5-specific class (Spark35NestedSchemaPruning) which is not available in the current Spark environment. This typically occurs when there's a mismatch between the Spark version Hudi was built against and the Spark version being used at runtime.  Specifically, the Hudi JARs being used were compiled with Spark 3.5 support, but that support is not available in the environment where the application is running.

**Solution:**

To resolve this issue, ensure that the Hudi JARs are compatible with your Spark runtime environment. There are a couple of ways to achieve this:

1. Build a Hudi JAR that is compatible with your Spark version:

   * When building Hudi from source, use the appropriate Spark and Scala profiles to create a JAR that matches your environment.
   * For Spark 3.5 and Scala 2.12, use:
        `mvn clean package -DskipTests -Dspark3.5 -Dscala-2.12`
   * This will create a hudi-utilities-bundle JAR that includes the necessary Spark 3.5 support.  Ensure that this is the JAR used in your Spark application.

2. Use the correct Hudi JARs for your Spark version:

   * If you are not building Hudi from source, ensure you are using the correct pre-built Hudi JARs for your Spark version.
   * For example, if you have this error, make sure you are including  both `hudi-utilities-slim-bundle_2.12-0.15.0.jar` and `hudi-spark3.5-bundle_2.12-0.15.0.jar` (or the correct version for your setup)
   * Using a "slim" JAR in conjunction with a specific Spark version JAR provides the necessary components without including unnecessary dependencies that might cause conflicts.
     
## AvroTypeException: Cannot encode decimal with precision 31 as max precision 30

```sh
export HUDI_SPARK_BUNDLE_JAR=/tmp/hudi-spark3.5-bundle_2.12-0.15.0-SNAPSHOT.jar

spark-shell --master "local[2]" \
  --jars $HUDI_SPARK_BUNDLE_JAR \
  --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
  --conf 'spark.sql.catalog.spark_catalog=org.apache.spark.sql.hudi.catalog.HoodieCatalog' \
  --conf 'spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension' \
  --conf 'spark.kryo.registrator=org.apache.spark.HoodieSparkKryoRegistrar'
```

```scala
import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.sql.types._
import java.math.BigDecimal

// 1. Define a schema with Decimal precision 31
// The error specifically mentions a precision of 31 and a scale adjustment
val schema = StructType(Seq(
  StructField("id", StringType, false),
  StructField("amount", DecimalType(31, 2), true), // Precision 31 triggers the MDT limit
  StructField("ts", LongType, true)
))

// 2. Create sample data
val rows = Seq(
  Row("1", new BigDecimal("12345678901234567890123456789.12"), 1705661400L)
)

val df = spark.createDataFrame(spark.sparkContext.parallelize(rows), schema)
df.show()

val tableName = "hudi_decimal_repro"
val basePath = f"/tmp/$tableName"

// 3. Hudi Options - Metadata and Column Stats MUST be enabled to trigger this
val hudiOptions = Map(
  "hoodie.table.name" -> tableName,
  "hoodie.datasource.write.recordkey.field" -> "id",
  "hoodie.datasource.write.precombine.field" -> "ts",
  "hoodie.metadata.enable" -> "true",
  "hoodie.metadata.index.column.stats.enable" -> "true", // This index uses the Avro schema with max precision 30
  "hoodie.datasource.write.operation" -> "insert"
)

// 4. Write the data to Hudi table
df.write.format("hudi").options(hudiOptions).mode("Overwrite").save(basePath)
```

Disable the metadata index column stats:

```scala
val hudiOptions = Map(
  "hoodie.table.name" -> tableName,
  "hoodie.datasource.write.recordkey.field" -> "id",
  "hoodie.datasource.write.precombine.field" -> "ts",
  "hoodie.metadata.enable" -> "true",
  "hoodie.metadata.index.column.stats.enable" -> "false",
  "hoodie.datasource.write.operation" -> "insert"
)

df.write.format("hudi").options(hudiOptions).mode("Overwrite").save()

spark.read.format("hudi").load(basePath).show(false)
```

```scala
import scala.math.BigDecimal
val amount: BigDecimal = BigDecimal("12345678901234567890123456789.12")
val precision = amount.precision // prints 31
val scale = amount.scale         // prints 2

import scala.math.BigDecimal
val amount: BigDecimal = BigDecimal("1234567890134567890123456789.12")
val precision = amount.precision // prints 30
val scale = amount.scale         // prints 2
```
