###Environment
|    HostName   |            IP             | CPU | Mem | Disk | usr/pwd |      OS         |
| :-----------: |:-------------------------:| :--:| :--:| :---:| :----:| :--------------:|
|  baas-test(master)    | 9.186.91.104/192.168.5.217  | 4c  | 8g | 50g | crluser/passw0rd| ubuntu16.04 x86 |
|  baas-test2(slave1)    | 9.186.90.93/192.18.5.34  | 4c  |  8g | 20g | crluser/passw0rd| ubuntu16.04 x86 |
|  baas-test3(slave2)    | 9.186.90.92/192.168.5.35  | 4c  | 8g | 20g | crluser/passw0rd| ubuntu16.04 x86 |

### Software
```python
Ubuntu 16.04
Hadoop:2.7.3
java:1.8.0
scala:2.10.4
spark:2.1.0
```

### Configuration Vms

#### Modify hostname
At each nodes config /etc/hosts
```sh
$ sudo vi /etc/hosts
192.168.5.217 baas-test
192.168.5.34 baas-test2
192.168.5.35 baas-test3

```
Test if the configuration can be work
```sh
ping baas-test2
ping baas-test3
```

#### Ssh configuration 

* Generate ssh key at each vms
```sh
ssh-keygen -t rsa
```
* Send slaves' id_rsa.pub to master
```sh
scp ~/.ssh/id_rsa.pub crluser@baas-test:~/.ssh/id_rsa.pub.slave1
```
* At master node, add slaves' ssh key to authorized_keys
```sh
cat ~/.ssh/id_rsa.pub* >> ~/.ssh/authorized_keys

```

* Cop authorized_keys to each slaves
```sh
scp ~/.ssh/authorized_keys crluser@baas-test2:~/.ssh/
scp ~/.ssh/authorized_keys crluser@baas-test3:~/.ssh/
```

* Test
```sh
ssh baas-test
ssh baas-test2
ssh baas-test3
```

### Install java

```sh
sudo vi /etc/profile

export WORK_SPACE=/home/crluser/workspace
export JAVA_HOME=$WORK_SPACE/jdk1.8.0_111
export JRE_HOME=/home/crluser/workspace/jdk1.8.0_111/jre
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

$ source /etc/profile
$ java -version
```
	
### Install scala

```sh
$ sudo vi /etc/profile
  export SCALA_HOME=$WORK_SPACE/scala-2.10.4
  export PATH=$PATH:$SCALA_HOME/bin

$ source /etc/profile
#test
$ scala -version
```

### Install spark

#### Install spark on master

* Edit /etc/profile add environment variables
 ```sh
export SPARK_HOME=/home/crluser/workspace/spark-2.1.0-bin-hadoop2.7
export PATH=$SPARK_HOME/bin:$SPARK_HOME/sbin
 ```
 
* Edit spark-env.sh
 
 Create a copy of template of spark-env.sh and rename it:
 ```sh
 $cp spark-env.sh.template spark-env.sh
 
SPARK_MASTER_HOST=192.168.5.217
SPARK_WORKER_CORES=3
SPARK_WORKER_MEMORY=2G
SPARK_WORKER_INSTANCES=1
export SCALA_HOME=/home/crluser/workspace/scala-2.10.4
export JAVA_HOME=/home/crluser/workspace/jdk1.8.0_111
SPARK_LOCAL_DIRS=/home/crluser/workspace/spark-2.1.0-bin-hadoop2.7
SPARK_DRIVER_MEMORY=1G
export STANDALONE_SPARK_MASTER_HOST=baas-test

 ```
 
* Add slaves:

Create configurations file slaves(in $SPARK_HOME/conf) and add following entries:
```sh
baas-test
baas-test2
baas-test3
```

* Change log level:

In file conf/log4j.properties, set variable log4j.rootCategory=WARN 


#### Install spark on slaves

* Copy setups from master to slaves

```sh
scp $SPARK_HOME crluser@baas-test2:~/workspace
scp $SPARK_HOME crluser@baas-test3:~/workspace
```

#### Start and test spark cluster

* Start spark cluster

```sh
$ sbin/start-all.sh (at baas-test)
```

* Check daemons on master

```sh
$ jps
14867 Worker
16154 Jps
14461 Master

```

* Check daemons on slaves

```sh
14790 Worker
16349 Jps
```

* Browse the Spark UI for more information about the cluster resources.details of worker nodes, running application, etc.

	http://9.186.91.104:8080/ 
   
#### Simple Test 

```sh
# Run a Python application on a Spark standalone cluster
./bin/run-example SparkPi 10
./bin/spark-submit  --master spark://9.186.91.104:7077 examples/src/main/python/pi.py 1000
./bin/spark-submit  --master spark://9.186.91.104:7077 examples/src/main/python/wordcount.py test/filetest.txt
```
### Install Hadoop

Use for yarn mode, and use HDFS as distributed storage system, yarn as resource management.  Installation reference [link](http://linoxide.com/cluster/setup-hadoop-multi-node-cluster-ubuntu/)

supplement: Add the following config in yarn-site.xml
```note
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
</property>
```
Total configration(~/.bashrc or /etc/profile)

```sh
export WORK_SPACE=/home/crluser/workspace
export JAVA_HOME=$WORK_SPACE/jdk1.8.0_111
export JRE_HOME=/home/crluser/workspace/jdk1.8.0_111/jre
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

export SCALA_HOME=$WORK_SPACE/scala-2.10.4
export PATH=$PATH:$SCALA_HOME/bin

export HADOOP_OPTS="-Djava.library.path=/home/crluser/workspace/hadoop-2.7.3/lib/native"
export HADOOP_COMMON_LIB_NATIVE_DIR=/home/crluser/workspace/hadoop-2.7.3/lib/native
export HADOOP_PREFIX=/home/crluser/workspace/hadoop-2.7.3
export HADOOP_HOME=/home/crluser/workspace/hadoop-2.7.3
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_CONFDIR=$HADOOP_HOME/etc/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export YARN_CONF_DIR=$YARN_HOME/etc/hadoop

export SPARK_HOME=/home/crluser/workspace/spark-2.1.0-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

```

### WEB UI

Spark WebUI  [http://9.186.91.104:8080/](http://9.186.91.104:8080/)

Hadoop WebUI [http://9.186.91.104:8088/cluster/nodes](http://9.186.91.104:8088/cluster/nodes)

HDFS Cluster [http://9.186.91.104:50070/dfshealth.html#tab-overview](http://9.186.91.104:50070/dfshealth.html#tab-overview)

### Scheduling mode: standalone and yarn

hadoop 中集成了分布式文件管理系统hdfs和分布式资源管理系统yarn

spark中集成了资源管理系统standalone 

Spark on YARN 支持两种运行模式，分别为yarn-cluster和yarn-client，具体的区别可以看[这篇博文](http://www.cnblogs.com/MOBIN/p/5857314.html)，从广义上讲，yarn-cluster适用于生产环境；而yarn-client适用于交互和调试

Application 在Yarn ResourceManager为其分配的一个随机节点上运行；

而在yarn-client模式中，SparkContext运行在本地，该模式适用于应用程序本身需要在本地进行交互的场合。

其中yarn模式下不能执行python 的应用

### Filesystem: local/hdfs

对于文件系统有两种选择 hdfs 和local

hadoop/etc/hadoop/core-site.xml 默认是从hdfs读取数据文件,
也可以指定sc.textFile("路径")，在路径前面加上hdfs://表示从hdfs文件系统上读，
本地文件读取 sc.textFile("路径")，在路径前面加上file:// 表示从本地文件系统读，如file:///home/user/spark/README.md

读取HDFS文件，Spark则会根据数据的存储位置，分配离数据存储最近的Executor去执行任务。

hdfs无法很好的处理小文件(<128M), 适合处理大文件，
hdfs的存储单元是Block。partition指的就是数据分片的数量，默认以128M分割文件，并分别复制到每个datanode节点 每一次task只能处理一个partition的数据，
通过spark.default.parallelism可以设置默认的分片数量

dfs.blockSize=128M 


如果是本地文件系统，那么这个文件必须在所有的worker节点上能够以相同的路径访问到。所以要么把文件复制到所有worker节点上同一路径下，要么挂载一个共享文件系统


### Hdfs-shell 
hadoop:
fs -ls

fs -put

fs -get

fs -rm (-r)

fs -du (-s -h)

fs -df (-h)

```sh
hadoop fs -mkdir -p hdfs://baas-test:9000/user/crluser/test

bin/hadoop fs -put /home/crluser/workspace/spark-2.1.0-bin-hadoop2.7/test/filetest.txt  hdfs://baas-test:9000/user/crluser/test

hadoop fs -ls hdfs://baas-test:9000/user/crluser/test

```
more infomation see: hadoop fs -help

### Spark-submit


#### spark-submit applications on yarn use hdfs

```sh
# Run java application on a YARN cluster 
# $ export HADOOP_CONF_DIR=XXX
$ ./bin/spark-submit \
  --master yarn \
  #--master  spark://192.168.5.217:7077 \ # choice resource management,spark://host:port(standalone), messos://host:port,yarn or local 
  --deploy-mode cluster \ # can be client
  --class <main-class> \ # for java/scala such as org.apache.spark.examples.SparkPi 
  --name "Example Program" \
  --jars dep1.jar, dep2.far,dep3.jar \
  --total-executor-cores 20
  --executor-memory 10G \
  /path/to/myApp.jar \ 
  [arguments]

# Run a python application
$ ./bin/spark-submit \
  --master yarn \
  --py-files somelib.egg,otherlib.zip,other-file.py \
  --deploy-mode client \
  --name "example program" \
  --queue examplequeue \
  --num-executors 40 \
  --executor-memory 10G \
  /path/to/my_scripy.py \
  [arguments]

```
For Python applications, simply pass a .py file in the place of <application-jar> instead of a JAR, and add Python .zip, .egg or .py files to the search path with --py-files.


#### spark-submit  applications on standalone use hdfs 

standalone mode does not support cluster  for python applications

```pyton
spark-submit  --master spark://9.186.91.104:7077 examples/src/main/python/wordcount.py test/filetest.txt
```
spark-submit examples: http://spark.apache.org/docs/latest/submitting-applications.html,  and run "spark-submit --help" for more information.

### Spark-shell 

Simple test(scala):

```sh
$ ./bin/spark-shell (or path/to/spark-shell)
$ var textfile=sc.textFile("file:///home/crluser/workspace/spark-2.1.0-bin-hadoop2.7/test/airlines.csv")
or use file in hdfs
$ textfile=sc.textFile("hdfs://9.186.91.104:9000/user/crluser/test/filetest.txt")

$ textfile.count()
$ textfile.first()
```
Simple test(python):

```sh
$ ./bin/pyspark
$ textfile=sc.textFile("file:///home/crluser/workspace/spark-2.1.0-bin-hadoop2.7/test/airlines.csv")
or use file in hdfs
$ textfile=sc.textFile("hdfs://9.186.91.104:9000/user/crluser/test/filetest.txt")

$ textfile.count()
$ textfile.first()
```
More infomation in WEB [http://9.186.91.104:4040](http://9.186.91.104:4040/jobs/)

$ spark-shell --master yarn-client --executor-memory 1G --num-executors 10 ((必须是yarn-client 运行在本地(master)) 属于调试)

之后在resourcemanager(http://9.186.91.104:8088)的web 页面上查看 application (hadoop-yarn)

在http:baas-test:4040 会跳转到此job上 查看job信息

### Spark-python program

1.命令行与spark交互
2.使用python写spark
3.提交spark作业到集群

#### pyspark

#### spark-submit

#### Using IPython Notebook with Spark

* Install

* Confige in PySpark


### Spark Streaming

Real time streaming data analysis


### Spark SQL

Spark SQL is a Spark module for structured data processing

### MLLib 

Machine learning 


### GraphX

Graph Processing 
