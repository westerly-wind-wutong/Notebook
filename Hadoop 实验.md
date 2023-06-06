Hadoop 实验

```bash
mkdir /usr/local/install_pack
mkdir /usr/local/software
tar -zxvf /usr/local/install_pack/jdk-8u351-linux-x64.tar.gz -C /usr/local/software/
vi ~/.bashrc
source ~/.bashrc
java -version
vi /etc/hosts
mkdir /root/bin
cd bin/
vi xcall
```

```bash
#!/bin/bash
if(($#==0))
then
	echo "请输入参数"
	exit;
fi
	echo "----------------master-----------------"
	ssh     master $@
for((i=1;i<3;i++))
do
	echo "----------------slave$i-----------------"
	ssh slave$i $@
done
	echo "命令已执行完毕"
```

```bash
chmod 744 xcall
vi ~/.bashrc
source ~/.bashrc
xcall jps
```

```bash
ssh-keygen -t rsa # 所有机器都执行
cd ~/.ssh
cat id_rsa.pub>>authorized_keys
ssh root@slave1 cat ~/.ssh/id_rsa.pub >> authorized_keys
ssh root@slave2 cat ~/.ssh/id_rsa.pub >> authorized_keys
scp /root/.ssh/authorized_keys known_hosts root@slave1:/root/.ssh/
scp /root/.ssh/authorized_keys known_hosts root@slave2:/root/.ssh/
```

```bash
xcall yum install -y rsync
cd ~/bin/
vi xsync
```

```bash
#!/bin/bash
if(($#!=1))
then
	echo "只能输入一个参数"
	exit;
fi
dirpath=$(cd -P `dirname $1`;pwd)
filename=$(basename $1)
echo "-----------------master--------------"
rsync -rvlt $dirpath/$filename root@master:$dirpath
for((i=1;i<3;i++))
do
	echo "-----------------slave$i----------------"
	rsync -rvlt $dirpath/$filename root@slave$i:$dirpath
done
echo "命令执行完毕"
```

```bash
xsync /usr/local/software/
xsync ~/.bashrc
xcall source ~/.bashrc
xcall jps
```





