# Hadoop实验之HBase

## 实验一：安装与配置

（一）HBase安装
进入install_pack目录查看Hive压缩包

```bash
cd /usr/local/install_pack/
ls -lrt
```

解压Hbase压缩包

```bash
tar -zxvf hbase-2.2.3-bin.tar.gz -C /usr/local/software/
```

进程software目录查看解压好的文件

```bash
cd /usr/local/software
ls -l
```

（二）配置安装Hbase
修改HBase的配置文件hbase-env.sh

```bash
cd /usr/local/software/hbase-2.2.3/conf
vim hbase-env.sh
```

在文件末尾追加下述内容

```bash
export JAVA_HOME=/usr/local/software/jdk1.8.0_351
export HBASE_MANAGES_ZK=flase
```

修改HBase的配置文件hbase-site.xml

```bash
cd /usr/local/software/hbase-2.2.3/conf
vim hbase-site.xml
```

在configuration标签间添加下述配置
```xml
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://master:8020/hbase</value>
</property>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>master,slave1,slave2</value>
</property>
<property>
    <name>hbase.master.maxclockskew</name>
    <value>2700000</value>
</property>
<property>
    <name>hbase.tmp.dir</name>
    <value>/usr/local/software/hbase-2.2.3/data/tmp</value>
</property>
<property>
    <name>zookeeper.znode.parent</name>
    <value>/hbase/master</value>
</property>
<property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
</property>
```

新建一个放临时文件的文件夹，此处新建的文件为配置文件里面配置的路径

```bash
mkdir -p /usr/local/software/hbase-2.2.3/data/tmp
```

在HDFS文件系统新建文件夹hbase，此处新建的HDFS文件为配置文件里面配置的路径

```bash
hadoop fs -mkdir /hbase
```

将hadoop中hdfs-site.xml拷贝到HBASE_HOME/conf下

```bash
cp /usr/local/software/hadoop-3.2.1/etc/hadoop/hdfs-site.xml /usr/local/software/hbase-2.2.3/conf/
```

修改HBase的配置文件配置regionservers

```bash
vim /usr/local/software/hbase-2.2.3/conf/regionservers
```

删除里面的内容，添加slave1、slave2的主机名

```
slave1
slave2
```

Hbase配置HA，新建backup-masters文件,

```bash
cd /usr/local/software/hbase-2.2.3/conf
vim backup-masters
```

添加slave1的主机名

```
slave1
```

把整个hbase安装目录发送给子节点，slave1与slave2

```bash
cd /usr/local/software
xsync /usr/local/software/hbase-2.2.3
```

配置HBASE_HOME三个节点都需要

```
vim /etc/profile
```

将下述内容追加进文件末尾

```bash
# 配置HBASE_HOME
export HBASE_HOME=/usr/local/software/hbase-2.2.3
export PATH=$PATH:$HBASE_HOME/bin
```

加载设置三个节点都需要

```bash
source /etc/profile
```

## 实验二：数据的导入

（一）环境准备
启动集群（master节点运行）（如果已启动则不需要再运行下列命令）

```bash
zk start
start-all.sh
start-hbase.sh
```

查看进程

```bash
xcall jps
```

启动Hbsee Shell

```bash
hbase shell
```

（二）数据准备
新建数据文件夹（文件夹已存在就不需要创建）

```bash
mkdir /usr/local/software/hbase-2.2.3/data
cd /usr/local/software/hbase-2.2.3/data
```

创建测试数据

```bash
vim f.csv
```

将下列内容输入文件中

```
1,"aaa"
2,"bbb"
3,"ccc"
4,"ddd"
5,"eee"
```

将文件上传到hdfs文件系统

```bash
hadoop fs -mkdir -p /data/hbase/case	
hadoop fs -put /usr/local/software/hbase-2.2.3/data/f.csv /data/hbase/case
```

（三）使用importTsv功能将csv文件数据导入HBase
启动Hbsee Shell

```bash
hbase shell
```

hbase中创建要导入数据的表

```shell
create 'test' , 'f1'
```

数据导入(需要打开另一个终端)

```bash
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator="," -Dimporttsv.columns=HBASE_ROW_KEY,f1 test /data/hbase/case/f.csv
```

MapReduce执行成功之后会显示下图结果：

查看导入HBase中的数据

```shell
scan 'test'
```

（四）使用import功能，将数据导入HBase
使用import功能进行数据导入，导入的文件必须是sequence文件。与import相对的还有export功能，export功能导出的文件为sequence文件。
使用export功能将HBase表中的数据导出，代码如下：(需要打开另一个终端)

```bash
hbase org.apache.hadoop.hbase.mapreduce.Export test /output/hbase/data
```

查看得到的结果：(需要打开另一个终端)

```bash
hadoop fs -ls /output/hbase/data
```

创建表：

```shell
create 'test2','f1'
```

执行import的MapReduce导入数据(需要打开另一个终端)

```bash
hbase org.apache.hadoop.hbase.mapreduce.Import test2 /output/hbase/data
```

查看HBase表中的数据

```shell
scan 'test2'
```

（五）使用BulkLoad功能将数据导入HBase
通过importTsv生成HFile文件(需要打开另一个终端)

```bash
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator="," -Dimporttsv.bulk.output=/output/hbase/data1 -Dimporttsv.columns=HBASE_ROW_KEY,f1 test3 /data/hbase/case/f.csv
```

此过程会主动创建HBase表和HDFS上对应的目录

将数据导入到HBase表中(需要打开另一个终端)

```bash
hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /output/hbase/data1 test3
```

查看test3表

```shell
scan 'test3'
```

补充：HBase Table DDL

1. 创建表

```shell
create 'table name','column family'
# create '[表名]','[列簇名]'
# 例：
create 'test' , 'f1'
```

2. 插入/更新操作

```shell
put 'table name','row','column family: column name', 'value'
# put '[表名]','[行键]','[列簇名]: [列名]', '[值]'
# 例：
put 'test','6','f1:','"fff"'
```

3. 删除操作

删除一条数据

```shell
delete 'table name', 'row', 'column family: column name', 'timestamp'
# delete '[表名]','[行键]','[列簇名]: [列名]','[时间戳]'
delete 'test','6','f1:'
```

删除表
      先用“disable”让表变为禁用状态，然后进行删除操作。若表不是禁用状态，无法删除。
```shell
disable 'table name'
drop 'table name'
```

## 实验三：API使用