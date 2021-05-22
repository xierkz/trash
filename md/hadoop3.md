# Hadoop3.1.4伪分布式安装

### 一、JDK 安装

##### 1.下载jdk-8u291-linux-x64.tar.gz

```http
https://download.oracle.com/otn/java/jdk/8u291-b10/d7fc238d0cbf4b0dac67be84580cfb4b/jdk-8u291-linux-x64.tar.gz?AuthParam=1619506783_24d5fb0e1997e152d50557e40c4c7766
<!--注：Oracle 下载jdk8其他版本链接-->
https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html
```

##### 2.解压jdk-8u291-linux-x64.tar.gz

```shell
tar -zxvf jdk-8u291-linux-x64.tar.gz
mv jdk1.8.0_291 jdk
sudo mv jdk /usr/local
sudo vi /etc/profile
export JAVA_HOME=/usr/local/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin
source /etc/profile
java -version
```

![https://github.com/xierkz/trash/blob/main/md/img/1.png](https://github.com/xierkz/trash/blob/main/md/img/1.png)

### 二、Hadoop 安装

##### 1.下载Hadoop3.1.4

```http
https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.1.4/hadoop-3.1.4.tar.gz
```

##### 2.解压hadoop-3.1.4.tar.gz

```shell
tar -zxvf hadoop-3.1.4.tar.gz
mv hadoop-3.1.4 hadoop
sudo mv hadoop /usr/local
sudo vi /etc/profile
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
source /etc/profile
hadoop version
```

![2](D:\Transh\MD\img\3.png)

##### 3.修改/usr/local/hadoop/etc/hadoop下配置文件

```shell
vi core-site.xml
```

```xml
<configuration>
</configuration>
修改为：
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

```shell
vi  hdfs-site.xml
```

```xml
<configuration>
</configuration>
修改为：
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```

```shell
vi hadoop-env.sh		——54行
```

```shell
export JAVA_HOME=/usr/local/jdk
```

```shell
vi mapred-site.xml
```

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

```shell
vi yarn-site.xml
```

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
        </property>
</configuration>
```

##### 4.格式化并启动、关闭Hadoop

```shell
格式化：hdfs namenode -format
```

```shell
启动：start-all.sh
```

```txt
两种方式检查是否成功？
```

```shell
jps
```

![4](D:\Transh\MD\img\7.png)

```http
localhost:9870
```

![](D:\Transh\MD\img\5.png)

```shell
关闭：stop-all.sh
```

### 三、Hdfs 编程

##### 1.Shell 编程

```shell
hadoop fs/hdfs dfs/hadoop dfs
hdfs dfs -mkdir /input
hdfs dfs -ls /
hdfs dfs -put ~/1.txt /input
hdfs dfs -cat /input/1.txt
hdfs dfs -mv /input/1.txt /input/2.txt
hdfs dfs -get /input/2.txt ~
hdfs dfs -rm -f /input/2.txt
```

##### 2.Java API

```shell
创建一个文件夹将/usr/local/hadoop/share/hadoop下.jar文件拷贝到~/jar文件夹中
mkdir ~/jar
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.junit.Before;
import org.junit.Test;
import org.junit.After;

public class HDFSDemo {
	FileSystem fs = null;
	@Before
	public void startup() throws IOException, InterruptedException, URISyntaxException{
		Configuration conf = new Configuration();
		fs = FileSystem.get(new URI("hdfs://localhost:9000"),conf,"用户名");
	}
	@Test
    /*
    * 从本地上传到hdfs
    */
    public void uploads() throws IOException {
        fs.copyFromLocalFile(false,true,new Path(""),new Path(""));
    }
    @Test
    /*
    * 从hdfs下载到本地
    */
    public void downloads() throws IOException {
        fs.copyToLocalFile(false,new Path(""),new Path(""),true);
    }
    @Test
    /*
    * 在hdfs创建文件夹
    */
    public void mkdirs() throws IOException {
        fs.mkdirs(new Path(""));
    }
    @Test
    /*
    * 删除hdfs文件夹
    */
    public void deleteds() throws IOException {
        fs.delete(new Path(""),true);
    }
    @After
    /*
    * 关闭服务
    */
    public void ends() throws IOException {
        fs.close();
    }
}

```

### 四、MapReduce Java API编程

##### 1.创建一个文件夹将/usr/local/hadoop/share/hadoop下.jar文件到~/jar文件夹中

```shell
mkdir ~/jar
```

##### 2.创建项目MapReduceDemo

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        String[] words = line.split(" ");
        for (String word : words) {
            context.write(new Text(word),new IntWritable(1));
        }
    }
}
```

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
public class WordCountReducer extends Reducer<Text,IntWritable,Text,IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for(IntWritable value:values){
            count += value.get();
        }
        context.write(key,new IntWritable(count));
    }
}
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {
    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(WordCount.class);
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setCombinerClass(WordCountReducer.class);
        FileInputFormat.setInputPaths(job,"HDFS上文件");
        FileOutputFormat.setOutputPath(job,new Path("HDFS输出文件"));
        job.waitForCompletion(true);
    }
}
```

##### 3.运用jar文件跑MapReduce

```shell
hadoop jar WordCount.jar
```

##### 4.Hadoop与Hbase版本兼容性

![9](D:\Transh\MD\img\9.png)

### 五、Hbase 安装

##### 1.下载hbase-2.2.7-bin.tar.gz

```http
https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.2.7/hbase-2.2.7-bin.tar.gz
```

##### 2.解压下载hbase-2.2.7-bin.tar.gz

```shell
tar -zxvf hbase-2.2.7-bin.tar.gz
mv hbase-2.2.7 hbase
sudo mv hbase /usr/local
sudo vi /etc/profile
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${HBASE_HOME}/bin
source /etc/profile
hbase version
```

![image-20210516014420080](D:\Transh\MD\img\image-20210516014420080.png)

##### 3.修改/usr/local/hbase/conf/下配置文件

```shell
vi hbase-env.sh
```

并将最后一行注释去除

```shell
export JAVA_HOME=/usr/local/jdk
export HBASE_CLASSPATH=/usr/local/hadoop/conf 
export HBASE_MANAGES_ZK=true
```

```shell
vi hbase-site.xml
```

```xml
<configuration>
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://localhost:9000/hbase</value>
        </property>
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
</configuration>
```

##### 4.Hbase 启动和关闭

```shell
必须先启动hadoop
start-all.sh
再启动hbase
start-hbase.sh
打开浏览器：localhost:16010
再关闭hbase
stop-hbase.sh
再关闭hadoop
stop-all.sh
```

##### 5.Hbase Shell

```shell
hbase shell
```

![](D:\Transh\MD\img\10.png)

```txt
list
scan '表名'
describe '表名'
create '表名',{NAME=>'列名',VERSIONS=>X},{NAME=>'列名',VERSIONS=>X}
put '表名','行键','列名','具体值'
get '表名','行键',{COLUMN=>'列名'，VERSIONS=>x}

disable '表名'
delete/drop '表名'

delete '表名','行键','列名'
deleteall '表名','行键'
cout '表名'
exit
```

##### 6.Java API

```shell
创建一个文件夹将/usr/local/hadoop/share/hadoop,/usr/local/hbase/lib下.jar文件拷贝到~/jar文件夹中
mkdir ~/jar
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;

import java.io.IOException;

public class HbaseDemo {
    public static Configuration conf;
    public static Connection con;
    public static Admin ad;

    //Link
    public static void init() throws IOException {
        conf = HBaseConfiguration.create();
        conf.set("hbase.rootdir", "hdfs://localhost:9000/hbase");
        con = ConnectionFactory.createConnection(conf);
        ad = con.getAdmin();
    }

    //Close
    public static void close() {
        try {
            if (ad != null) {
                ad.close();
            }
            if (con != null) {
                con.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Create Table
     */
    public static void create(String tableName, String[] lzm) throws IOException {
        init();
        TableName tn = TableName.valueOf(tableName);
        if (ad.tableExists(tn)) {   //is or no
            System.out.println(tn + "is exits!");
        } else {
            HTableDescriptor htd = new HTableDescriptor(tn);
            for (String str :
                    lzm) {
                HColumnDescriptor hcd = new HColumnDescriptor(str);
                htd.addFamily(hcd);
            }
            ad.createTable(htd);
            System.out.println(tableName + " is success.");
        }
    }

    /**
     * add
     */
    public static void insert(String tableName, String hj, String lzm, String lm, String z) throws IOException {
        init();
        Table table = con.getTable(TableName.valueOf(tableName));
        Put put = new Put(hj.getBytes());
        put.addColumn(lzm.getBytes(), lm.getBytes(), z.getBytes());
        table.put(put);
        table.close();
        close();
    }

    /**
     * deleteT:table
     * deleteD:data
     */
    public static void deleteT(String tableName) throws IOException {
        init();
        TableName tn = TableName.valueOf(tableName);
        if (ad.tableExists(tn)) {
            ad.disableTable(tn);
            ad.deleteTable(tn);
            System.out.println(tableName + " is deleted!");
        }
        close();
    }

    public static void deleteD(String tableName, String hj, String lzm, String lm) throws IOException {
        init();
        Table table = con.getTable(TableName.valueOf(tableName));
        Delete del = new Delete(hj.getBytes());
        //deleteall
        del.addFamily(lzm.getBytes());
        //delete
        del.addColumn(lzm.getBytes(), lm.getBytes());
        table.delete(del);
        table.close();
        close();
    }

    /**
     * listTable
     * getData
     */
    public static void listTable() throws IOException {
        init();
        HTableDescriptor htd[] = ad.listTables();
        for (HTableDescriptor td : htd) {
            System.out.println(td.getNameAsString());
        }
        close();
    }

    public static void getData(String tableName, String hj, String lzm, String lm) throws IOException {
        init();
        Table table = con.getTable(TableName.valueOf(tableName));
        Get get = new Get(hj.getBytes());
        get.addColumn(lzm.getBytes(), lm.getBytes());
        Cell[] cells = table.get(get).rawCells();
        for (Cell cell : cells) {
            System.out.println("RowName:" + new String(CellUtil.cloneRow(cell)) + " ");
            System.out.println("Timetamp:" + cell.getTimestamp() + " ");
            System.out.println("column Family:" + new String(CellUtil.cloneFamily(cell)) + " ");
            System.out.println("row Name:" + new String(CellUtil.cloneQualifier(cell)) + " ");
            System.out.println("value:" + new String(CellUtil.cloneValue(cell)) + " ");
        }
        table.close();
        close();
    }
}
```

```java
import java.io.IOException;

public class Demo {
    public static void main(String[] args) throws IOException {
        /**
         * table:Student
         */
        Hbaseo.create("Student", new String[]{"S_No", "S_Name", "S_Sex", "S_Age"});

        /**
         * add:Student
         */
        Hbaseo.insert("Student","1","S_No","","2015001");
        Hbaseo.insert("Student","1","S_Name","","Zhangan");
        Hbaseo.insert("Student","1","S_Sex","","male");
        Hbaseo.insert("Student","1","S_Age","","23");

        Hbaseo.insert("Student","2","S_No","","2015003");
        Hbaseo.insert("Student","2","S_Name","","Mary");
        Hbaseo.insert("Student","2","S_Sex","","female");
        Hbaseo.insert("Student","2","S_Age","","22");

        Hbaseo.insert("Student","3","S_No","","2015003");
        Hbaseo.insert("Student","3","S_Name","","Lisi");
        Hbaseo.insert("Student","3","S_Sex","","nale");
        Hbaseo.insert("Student","3","S_Age","","24");
    }
}
```

```java
import java.io.IOException;

public class deleteTable {
    public static void main(String[] args) throws IOException {
        Hbaseo.deleteT("Student");
    }
}
```

### 六、Hive 安装

##### 1.下载apache-hive-3.1.2-bin.tar.gz

```http
https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
```

##### 2.解压apache-hive-3.1.2-bin.tar.gz

```shell
tar -zxvf apache-hive-3.1.2-bin.tar.gz
mv apache-hive-3.1.2-bin hive
sudo mv hive /usr/local
sudo vi /etc/profile
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${HIVE_HOME}/bin
source /etc/profile
hive --version
```

![image-20210511201459619](D:\Transh\MD\img\image-20210511201459619.png)

##### 3.修改/usr/local/hive/conf下配置文件

```shell
MV hive-default.xml.template hive-default.xml
vi hive-site.xml
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
    <!-- 此处数据库为hive-->
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value><!-- 此处为设置密码hive-->
  </property>
</configuration>
```

##### 4.安装MySQL

```shell
sudo apt-get install mysql-server
service mysql start
sudo netstat -tap | grep mysql
mysql -u root -p
123456
exit;
service mysql stop
```

##### 5.下载MySQL驱动

```http
https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-5.1.49.tar.gz
<!--其他驱动自己选择-->
https://downloads.mysql.com/archives/
```

```shell
tar -zxvf mysql-connector-java-5.1.49.tar.gz
cd mysql-connector-java-5.1.49
sudo mv mysql-connector-java-5.1.49.jar /usr/local/hive/lib
```

##### 6.启动mysql shell

```shell
sudo gedit /etc/mysql/mysql.conf.d/mysqld.cnf		//配置文件
service mysql start
sudo mysql -u root -p
123456
grant all on *.* to root@localhost identified by 'hive';		/* 此处为密码*/
flush privileges;
system clear MySQL Linux 清屏命令
```

##### 7.一个很无聊的错误

```java
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
	at org.apache.hadoop.conf.Configuration.set(Configuration.java:1357)
	at org.apache.hadoop.conf.Configuration.set(Configuration.java:1338)
	at org.apache.hadoop.mapred.JobConf.setJar(JobConf.java:518)
	at org.apache.hadoop.mapred.JobConf.setJarByClass(JobConf.java:536)
	at org.apache.hadoop.mapred.JobConf.<init>(JobConf.java:430)
	at org.apache.hadoop.hive.conf.HiveConf.initialize(HiveConf.java:5141)
	at org.apache.hadoop.hive.conf.HiveConf.<init>(HiveConf.java:5099)
	at org.apache.hadoop.hive.common.LogUtils.initHiveLog4jCommon(LogUtils.java:97)
	at org.apache.hadoop.hive.common.LogUtils.initHiveLog4j(LogUtils.java:81)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:699)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:683)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:318)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:232)
```

​		反正很无赖

```text
HIVE和HADOOP内置的jar包版本不一致导致，
jar包分别在：hive/lib/guava-19.0.jar和share/hadoop/common/lib/guava-27.0-jre.jar
我们删除掉旧版本的19，将新版本拷贝进去，再次启动hive。
```

##### 8.启动 hive

```shell
start-all.sh
hive
```

##### 9.基本操作

```sql
create database if not exists hive;/*创建hive数据库*/
show databases;
describe database hive;
use hive;
#内部表
create table if not exists hive.usr(
       name string,
       pwd string,
       address struct<street:string,city:string,state:string,zip:int>
       identify map<int,tinyint> comment 'number,sex') 
       tblproperties('creator'='me','time'='2016.1.1');
#外部表     
create external table if not exists hive.usr2(
       name string,
       pwd string,
       address struct<street:string,city:string,state:string,zip:int>,
       identify map<int,tinyint>) 
       row format delimited fields terminated by ','
       location '/usr/local/hive/warehouse/hive.db/usr';
#分区表      
create table if not exists hive.usr3(
       name string,
       pwd string,
       address struct<street:string,city:string,state:string,zip:int>,
       identify map<int,tinyint>) 
       partitioned by(city string,state string); 
```

##### 10.Hive 编程

```sql
create table docs(line string);
load data inpath 'HDFS文件路径' overwrite into table docs;
create table word_count as 
select word, count(1) as count from
(select explode(split(line,' '))as word from docs) w
group by word
order by word;

select * from word_count;
```

##### 11.内部表和外部表介绍

```txt
内部表：会将数据移动到数据仓库指向的路径，删除时，会将元数据和数据一起删除。
外部表：仅记录数据所在的路径，删除时，只删除元数据，不删除数据本身，更安全，更方便数据共享。
```

##### 12.伪分布式毛病

```shell
Query ID = kz_20210516034201_b7bda3c8-7b60-4ec8-9126-789f035adc11
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
2021-05-16 03:42:03,004 INFO  [c9fb57f7-e60c-435c-8a58-7982947f72d1 main] client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2021-05-16 03:42:03,031 INFO  [c9fb57f7-e60c-435c-8a58-7982947f72d1 main] client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
Starting Job = job_1621107063277_0002, Tracking URL = http://kpl:8088/proxy/application_1621107063277_0002/
Kill Command = /home/kz/zip/hadoop/bin/mapred job  -kill job_1621107063277_0002
Hadoop job information for Stage-1: number of mappers: 0; number of reducers: 0
2021-05-16 03:42:07,238 Stage-1 map = 0%,  reduce = 0%
Ended Job = job_1621107063277_0002 with errors
Error during job, obtaining debugging information...
FAILED: Execution Error, return code 2 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 0 HDFS Write: 0 FAIL
Total MapReduce CPU Time Spent: 0 msec
```

```shell
set hive.exec.mode.local.auto=true;		——namenode内存空间不够，jvm不够新job启动
```

### 七、Spark安装

##### 1. 下载spark(python3.8以上选择3.x.x)

```http
https://archive.apache.org/dist/spark/spark-2.4.8/spark-2.4.8-bin-without-hadoop.tgz
https://archive.apache.org/dist/spark/spark-3.1.1/spark-3.1.1-bin-without-hadoop.tgz
```

##### 2. 安装spark

```shell
tar -zxvf spark-2.4.0-bin-without-hadoop.tgz
mv spark-2.4.0-bin-without-hadoop spark
sudo mv spark /usr/local/
sudo vi /etc/profile
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${SPARK_HOME}/bin
```

##### 3. 配置spark

```shell
cd /usr/local/spark/conf
cp spark-env.sh.template spark-env.sh
vi spark-env.sh
insert
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)
```

##### 4. 启动spark命令

```shell
spark-shell
```

![image-20210522234228371](D:\Transh\MD\img\image-20210522234228371.png)

```shell
:quit			--退出spark-shell
```

```shell
pyspark
```

![image-20210523002626468](D:\Transh\MD\img\image-20210523002626468.png)

```python
exit()/quit()
```
