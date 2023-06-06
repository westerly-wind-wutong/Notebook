# 22年hadoop实验1

## 实验二：Hive创建表

五、实验步骤：
（一）启动Hive

```bash
hive
```

（二）演练Hive创建表的三种方式

创建基础表：以常用的日志为例来创建page_view表：

```hive
CREATE TABLE IF NOT EXISTS default.page_view(
viewTime INT, 
userid BIGINT,
page_url STRING,
referrer_url STRING,
ip STRING COMMENT 'IP Address of the User')
COMMENT 'This is the page view table'
PARTITIONED BY(dt STRING, country STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
STORED AS SEQUENCEFILE;
```

创建子表：创建page_view_sl表如下

```hive
create table IF NOT EXISTS default.page_view_sl as select viewTime,userid,ip from default.page_view;
```

创建相同结构的表：创建default.page_view_s2

```hive
create table page_view_s2 like  page_view;
```

（三）对Hive表的基础操作

显示Hive中的所有表

```hive
show tables;
```

查看表详细属性

```hive
desc formatted page_view;
```

## 实验三：Hive分区操作

（一）数据准备
创建数据存放路径

```bash
mkdir /usr/local/software/hive-3.1.2/data/1
cd /usr/local/software/hive-3.1.2/data/1
```

创建partition.txt文件

```bash
vim partition.txt
```

将下述内容添加到文件中

```
2018	ddd
2018	eee
2018	fff
2018	ggg
```

创建partition2017.txt文件

```bash
vim partition2017.txt
```

将下述内容添加到文件中

```
2017	aaa
2017	bbb
2017	ccc
```

（二）启动Hive

```bash
cd /root/
hive
```

（三）操作流程
创建数据库

```hive
create database hive;
use hive;
```

创建分区表(parthive为表名，按年进行分区)

```hive
create table parthive (createdate string, value string) partitioned by (year string) row format delimited fields terminated by '\t';
```

分区:给parthive表创建2017与2018两个分区：

```hive
alter table parthive add partition(year='2017');
alter table parthive add partition(year='2018');
```

如果删除表分区，则对应的文件目录消失，但分区的数据会移动到别的分区。
查看parthive的表结构：

```hive
describe parthive;
```

向year=2018分区导入本地数据：

```hive
load data local inpath '/usr/local/software/hive-3.1.2/data/partition.txt' into table parthive partition(year='2018');
```

向year=2017分区导入本地数据：

```hive
load data local inpath '/usr/local/software/hive-3.1.2/data/partition2017.txt' into table parthive partition(year='2017');
```

（四）查询与统计分区表

查看数据内容

```hive
select * from parthive;
```

（五）查看分区存储目录

```hive
quit;
```

在命令行查看HDFS目录

```bash
hadoop fs -ls /user/hive/warehouse/hive.db/parthive
```

查看分区为2017的表的数据

```bash
hadoop fs -cat /user/hive/warehouse/hive.db/parthive/year=2017/*
```

打开Web端查看（ip+端口号）(根据自己NameNode所在虚拟机IP设置)
http://192.168.10.128:50070

## 实验四：Hive数据类型

（一）数据准备
创建arraytype.txt文件

```bash
cd /usr/local/software/hive-3.1.2/data
vim arraytype.txt
```

将下述内容添加到文件中

```
bisu beijing,shanghai,tianjin,hangzhou
linan changchu,chengdu,wuhan
```

创建maptype.txt文件

```bash
vim maptype.txt
```

将下述内容添加到文件中

```bash
bisu '数学':80,'语文':89,'英语':95
jobs '语文':60,'数学':80,'英语':99
```

创建structtype.txt文件

```bash
vim structtype.txt
```

将下述内容添加到文件中

```
1 english,80
2 math,89
3 chinese,95
```

（二）启动Hive
```bash
cd /root/
hive
```

（三）建表及数据入库
选择hive数据

```
use hive;
```

建立使用arraytype.txt数据的表

```hive 
create table  arraytype(name string,work_locations array<string>)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
COLLECTION ITEMS TERMINATED BY ',';
```

导入数据

```hive
LOAD DATA LOCAL INPATH '/usr/local/software/hive-3.1.2/data/arraytype.txt' OVERWRITE INTO TABLE arraytype; 
```

建立使用maptype.txt数据的表

```hive
create table maptype(name string, score map<string,int>)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':';
```

导入数据

```hive
LOAD DATA LOCAL INPATH '/usr/local/software/hive-3.1.2/data/maptype.txt' OVERWRITE INTO TABLE maptype;
```

建立使用structtype.txt数据的表

```hive
CREATE TABLE structtype(id int,course struct<course:string,score:int>)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
COLLECTION ITEMS TERMINATED BY ',';
```

导入数据

```hive
LOAD DATA LOCAL INPATH '/usr/local/software/hive-3.1.2/data/structtype.txt' OVERWRITE INTO TABLE structtype;
```

（四）查询及显示表内容
查询数据

```hive
select * from arraytype;
select * from maptype;
select * from structtype;
```

退出

```hive
quit;
```

## 实验五：数据定义操作DDL

（一）启动Hive
```bash
cd /root/
hive
```

Database DDL操作
创建数据库test:对应的HDFS目录为：/user/hive/warehouse的文件夹，命名格式：数据库名.db

```hive
create database test;
```

查看HDFS上的文件：注意（此命令为另建终端执行）

```bash
hadoop fs -ls /user/hive/warehouse
```

创建数据库test1:如果test1数据库不存在再创建

```hive
create database if not exists test1;
```

创建数据库test2:在指定路径下创建数据库（mydb：数据库名称）

```hive
create database test2 location '/myhive/mydb';
```

查看HDFS上的文件：注意（此命令为另建终端执行）

```bash
hadoop fs -ls /myhive/
```

创建数据库test3:创建数据库，并为数据库添加描述信息
```hive
create database test3 comment 'my test db';
```

查询所有的数据库
```hive
show databases;
```

查看以test开头的数据库
```hive
show databases like 'test*';
```

查看数据库的详细信息
```hive
desc database test3;
```

查看更详细的数据库信息
```hive
desc database extended test3;
```

切换数据库:如果不指定数据库，那么默认使用的就是default
```hive
use test3;
```

修改数据库的modifier
```hive
alter database test3 set dbproperties ('modifier'='shaonaiyi');
```

修改成功，重新查看详情
```hive
desc database extended test3;
```

删除数据库test3
```hive
drop database if exists test3;
```

删除成功查看所有数据库
```hive
show databases;
```

如果想要删除含有表的数据库，在删除时加上cascade，表示级联删除（慎用），可以使用如下命令。

```hive
drop database if exists test3 cascade;
```

Table DDL操作

修改表名emp为emp_bak
```hive
alter table emp rename to emp_bak;
```

清空表中所有数据

```hive
truncate table emp_bak;
```

删除表emp_bak

```hive> 
drop table if exists emp_bak;
```

## 实验六：数据操作DML

（一）启动Hive
```bash
cd /root/
hive
```
（二）数据库准备
创建数据库class
```hive
create database class;
```

选择数据库class
```hive
use class;
```

数据准备
创建目录（如果存在即直接进入data目录）(需要打开另一个终端)

```bash
mkdir /usr/local/software/hive-3.1.2/data
cd /usr/local/software/hive-3.1.2/data
```

新建文件student1.txt：

```bash
vim student1.txt
```

将下述内容添加到文件中

```
1       zhao    18
2       jun     19
```

新建文件student2.txt

```bash
vim student2.txt
```

将下述内容添加到文件中

```
3       feng    17
4       xiang   16
5       bin     15
```

将student2.txt 上传到hdfs（需要启动hadoop集群）

```
hadoop fs -mkdir -p /usr/hive/data/class	
hadoop fs -put /usr/local/software/hive-3.1.2/data/student2.txt /usr/hive/data/class
```

 创建Hive表并导入数据集

加载本地文件到student表
```hive
load data local inpath '/usr/local/software/hive-3.1.2/data/student1.txt' into table student partition(year='2019-2020');
```

加载hdfs上的文件到student表
```hive
load data inpath '/usr/hive/data/class/student2.txt' into table student partition(year='2019-2020');
```

覆盖上传数据到student表
```hive
load data local inpath '/usr/local/software/hive-3.1.2/data/student1.txt' overwrite into table student;
```

插入数据(Insert)
基本插入数据
hive> insert into table student partition(year='2019-2020') values(11, 'zzz',10);
根据单张表的查询结果插入数据
hive> insert into table student partition(year='2018-2019') select id,name,age from student where year='2019-2020';
根据多张表的查询结果插入数据
hive> insert into table student partition(year='2019-2020')
select id,name,age from student where year='2017-2018'
union
select id,name,age from student where year='2018-2019';
查看数据集：
hive> select * from student;

查询数据(Select)
查询语句中创建表并加载数据（As Select）（查询的结果会添加到新创建的表中）
hive> create table if not exists student1 as select id,name,age from student where year in ('2017-2018','2018-2019','2019-2020','2020-2021');
hive> select * from student1;

数据导出
将查询结果导出到本地
hive> insert overwrite local directory '/usr/local/software/hive-3.1.2/data/student1' select * from student;

查看数据内容：在另一个终端查看
 cd /usr/local/software/hive-3.1.2/data/student1
 cat 000000_0

将查询结果格式化导出到本地
hive> insert overwrite local directory '/usr/local/software/hive-3.1.2/data/student2' row format delimited fields terminated by '\t' select * from student;
查看数据内容：在另一个终端查看
cd /usr/local/software/hive-3.1.2/data/student2
cat 000000_0

将查询结果格式化导出到HDFS
hive> insert overwrite directory '/output/hive/student1' row format delimited fields terminated by '\t' select * from student;
查看数据内容：在另一个终端查看
hadoop fs -ls /output/hive/student1
hadoop fs -cat /output/hive/student1/*

清空数据（Truncate）删除表(Drop)
仅删除student表中数据，保留表结构
hive> truncate table student;
删除表student1
hive> drop table if exists student1;
