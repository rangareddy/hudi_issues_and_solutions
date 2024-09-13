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
