```bash
cd /usr/local/install_pack/
ls -l
```

## 实验一：集群搭建

（一）安装包准备
进入install_pack目录

```bash
cd /usr/local/install_pack/
```

解压Kafka压缩包

```bash
tar -zxvf kafka_2.12-2.4.0.tgz -C /usr/local/software/
```

配置Kafka
修改Kafka的配置文件 server.properties

```bash
cd /usr/local/software/kafka_2.12-2.4.0/config
vim server.properties
```

修改Zookeeper的连接节点

```bash
zookeeper.connect=master:2181,slave1:2181,slave2:2181
```

修改监听端口

```bash
listeners=PLAINTEXT://master:9092
```

修改日志路径

```bash
log.dirs=/usr/local/software/kafka_2.12-2.4.0/logs
```

复制master的Kafka到slave1、slave2

```bash
xsync /usr/local/software/kafka_2.12-2.4.0/
```

配置slave1的配置文件（修改broker.id和listeners,注意listeners的ip地址为slave1自己的ip地址）

```bash
cd /usr/local/software/kafka_2.12-2.4.0/config/
vim server.properties
```

```bash
broker.id=1
listeners=PLAINTEXT://slave1:9092
```

配置slave2的配置文件（修改broker.id和listeners,注意listeners的ip地址为slave2自己的ip地址）

```bash
cd /usr/local/software/kafka_2.12-2.4.0/config/
vim server.properties
```

```bash
broker.id=2
listeners=PLAINTEXT://slave2:9092
```

配置环境变量（master,slave1,slave2都需要操作）

```bash
vim /etc/profile
```

```bash
# 配置KAFAK_HOME
export KAFKA_HOME=/usr/local/software/kafka_2.12-2.4.0
export PATH=$PATH:$KAFKA_HOME/bin
```

使环境变量生效,同理在slave1,slave2节点也需要运行该命令

```bash
source /etc/profile
```

启动KAFKA
启动Zookeeper：

```bash
zk start
```

查看进程

```bash
xcall jps
```

注意：启动kafka要在zookeeper启动之后，kafka必须要在zookeeper关闭之前关闭。
启动Kafka集群，分别在master、slave1、slave2上的启动Kafka：

```bash
kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
```

查看进程

```bash
xcall jps
```

关闭Kafka集群，分别在master、slave1、slave2上的关闭Kafka：

```bash
kafka-server-stop.sh -daemon $KAFKA_HOME/config/server.properties
```

kafka启动脚本编写

```bash
cd /root/bin
vim kf
```

将下述内容输入到文件中

```bash
#!/bin/bash
if(($#!=1))
then
        echo "本脚本只支持参数start，stop"
        exit;
fi
        echo "-------------------------master-----------------------------"
        ssh master kafka-server-$@.sh -daemon $KAFKA_HOME/config/server.properties
        echo "kafka $@ completed"
for((i=1;i<3;i++))
do
        echo "-------------------------slave1-----------------------------"
        ssh slave$i kafka-server-$@.sh -daemon $KAFKA_HOME/config/server.properties
        echo "kafka $@ completed"
done
        echo "命令执行完毕"
```

```bash
chmod 744 kf
```

启动：```kf start```
关闭：```kf stop```

## 实验二：发布订阅消息系统

（一）环境检测

启动zookeeper与kafka

```bash
zk start
kf start
xcall jps
```

（二）Topic操作

创建Topic（名为my_repl5_topic，5个分区，复制因子为3）

```bash
kafka-topics.sh --zookeeper master:2181,slave1:2181,slave2:2181 --create --topic my_repl5_topic --replication-factor 3 --partitions 5
```

查看创建的Topic描述

```bash
kafka-topics.sh --zookeeper master:2181,slave1:2181,slave2:2181 --describe --topic my_repl5_topic
```

（三）启动生产者Producer
在**master**节点启动Producer（处于等待状态）：

```bash
kafka-console-producer.sh --broker-list master:9092,slave1:9092,slave2:9092 --topic my_repl5_topic
```

启动消费者Consumer
在**slave1**节点启动Consumer（处于等待状态）：

```bash
kafka-console-consumer.sh --bootstrap-server master:9092,slave1:9092,slave2:9092 --topic my_repl5_topic --from-beginning
```

演示发布订阅消息系统
切换到**master**终端，输入下列内容
hello
hadoop
kafka

切换到slave1终端，查看消费者消费内容