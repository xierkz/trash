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

![image-20210427160113921](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427160113921.png)

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

![image-20210427160514636](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427160514636.png)

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

![image-20210427163402025](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427163402025.png)

```http
localhost:9870
```

![image-20210427161836510](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427161836510.png)

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

![image-20210427183800863](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427183800863.png)

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

![image-20210427183236899](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427183236899.png)

##### 3.修改/usr/local/hbase/conf/下配置文件

```shell
vi hbase-env.sh
```

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

![image-20210427184344695](C:\Users\kz\AppData\Roaming\Typora\typora-user-images\image-20210427184344695.png)

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

##### 1.下载apache-hive-2.3.8-bin.tar.gz

```http
https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-2.3.8/apache-hive-2.3.8-bin.tar.gz
```

##### 2.解压apache-hive-2.3.8-bin.tar.gz

```shell
tar -zxvf apache-hive-2.3.8-bin.tar.gz
mv apache-hive-2.3.8-bin hive
sudo mv hive /usr/local
sudo vi /etc/profile
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${HIVE_HOME}/bin
source /etc/profile
```

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
service mysql start
mysql -u root -p
123456
grant all on *.* to 用户名@localhost identified by 'hive';		/* 此处为密码*/
flush privileges;
```

##### 7.启动 hive

```shell
start-all.sh
hive
```

##### 8.基本操作

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

##### 9.Hive 编程

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

##### 10.内部表和外部表介绍

```txt
内部表：会将数据移动到数据仓库指向的路径，删除时，会将元数据和数据一起删除。
外部表：仅记录数据所在的路径，删除时，只删除元数据，不删除数据本身，更安全，更方便数据共享。
```

