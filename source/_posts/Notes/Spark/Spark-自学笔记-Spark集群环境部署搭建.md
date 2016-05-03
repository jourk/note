---
title: Spark-自学笔记-Spark集群环境部署搭建
tags:
  - spark
categories:
  - Notes
date: 2015-05-22 11:24:06
---
## 规划及系统配置
### 硬件资源

```
192.168.188.131 master
192.168.188.132 slave1
192.168.188.133 slave2
```

### 基本资料

- 用户：hadoop
- 目录：/home/hadoop/

### 同步时间

root登陆

``` bash
$ su –
$ ntpdate cn.pool.ntp.org
```

### 修改hostname

``` bash
$ vim /etc/sysconfig/network 
NETWORKING=yes
HOSTNAME=master
```

### 修改/etc/hosts

``` bash
$ vim /etc/hosts
192.168.188.131 master
192.168.188.132 slave1
192.168.188.133 slave2
```

### 永久关闭防火墙(非常重要,一定要确认)

``` bash
$ chkconfig iptables off (永久生效)
$ service iptables stop (临时有效)
```

7、关闭SELINUX,设置`SELINUX=disabled`

``` bash
$ vim /etc/selinux/config
```

## 安装必要安装包并配置环境变量

### 下载安装

下载`jdk-7u79-linux-x64.gz`、`scala-2.10.5.tgz`、`hadoop-2.6.0.tar.gz`（64位）、`spark-1.3.1-bin-hadoop2.6.tar`,并分别解压到`/usr/local/`重命名为jdk、scala、hadoop、spark。

``` bash
$ mv hadoop-2.6.0 hadoop 
$ mv spark-1.3.1-bin-hadoop2.6 spark
$ mv scala-2.10.5 scala
$ mv jdk1.7.0_79 jdk
```

### 修改权限

修改用户权限：

``` bash
$ chown hadoop:hadoop jdk –R
$ chown hadoop:hadoop scala -R
$ chown hadoop:hadoop hadoop –R
$ chown hadoop:hadoop spark –R
```

### 配置系统环境变量

``` bash
$ vim /etc/profile
```

加入：

```
#set java environment
export JAVA_HOME=/usr/local/jdk
export JRE_HOME=/usr/local/jdk/jre
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

#set hadoop
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_PID_DIR=/data/hadoop/pids
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS=”$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib/native”
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HDFS_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

#set scala
export SCALA_HOME=/usr/local/scala
export PATH=$PATH:$SCALA_HOME/bin

#set spark
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bins
```

``` bash
$ source /etc/profile
```

## 规划系统目录

安装目录和数据目录分开,且数据目录和hadoop的用户目录分开,如果需要重新格式化,则可以直接删除所有的数据目录,然后重建数据目录。

如果数据目录和安装目录或者用户目录放置在一起,则对数据目录操作时,存在误删除程序或者用户文件的风险。

|完整路径|说明|
|---|---|
|/usr/local/hadoop|hadoop的程序安装主目录|
|/data/hadoop/storage/tmp|临时目录|
|/data/hadoop/storage/hdfs/name|namenode上存储hdfs名字空间元数据|
|/data/hadoop/storage/hdfs/data|datanode上数据块的物理存储位置|
|/data/hadoop/storage/mapred/local|tasktracker上执行mapreduce程序时的本地目录|
|/data/hadoop/storage/mapred/system|这个是hdfs中的目录,存储执行mr程序时的共享文件|

### 开始建立目录

在`master`下,`root`用户

``` bash
$ mkdir -p /data/hadoop/{pids,storage}
$ mkdir -p /data/hadoop/storage/{hdfs,tmp,mapred}
$ mkdir -p /data/hadoop/storage/hdfs/{name,data}
$ mkdir -p /data/hadoop/storage/mapred/{local,system}
$ chown -R hadoop:hadoop /data
```

### 修改拥有着

修改目录`/home/hadoop`的拥有者（因为该目录用于安装hadoop,用户对其必须有`rwx`权限。）

``` bash
$ chown -R hadoop:hadoop /home/hadoop
```

## Hadoop配置

### 配置 hadoop-env.sh、mapred-env.sh、yarn-env.sh

分别修改：

``` bash
$ vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
$ vim /usr/local/hadoop/etc/hadoop/mapred-env.sh
$ vim /usr/local/hadoop/etc/hadoop/yarn-env.sh
```

添加内容：

```
export JAVA_HOME=/usr/local/jdk
export CLASS_PATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_PID_DIR=/data/hadoop/pids
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS=”$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib/native”
export HADOOP_PREFIX=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HDFS_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

### core-site.xml

``` bash
$ vim $HADOOP_CONF_DIR/core-site.xml
```

``` xml
<configuration>

<property>
/*默认HDFS路径*/
<name>fs.defaultFS</name>
<value>hdfs://master:9000</value>
</property>

<property>
/*缓冲区大小*/
<name>io.file.buffer.size</name>
<value>131072</value>
</property>

<property>
/*临时文件夹路径*/
<name>hadoop.tmp.dir</name>
<value>file:/data/hadoop/storage/tmp</value>
</property>

<property>
<name>hadoop.proxyuser.hadoop.hosts</name>
<value>*</value>
</property>

<property>
<name>hadoop.proxyuser.hadoop.groups</name>
<value>*</value>
</property>

<property>
<name>hadoop.native.lib</name>
<value>true</value>
</property>

</configuration>
```

### hdfs-site.xml

``` bash
$ vim $HADOOP_CONF_DIR/hdfs-site.xml
```

``` xml
<configuration>

<property>
<name>dfs.namenode.secondary.http-address</name>
<value>master:9001</value>
</property>

<property>
<name> dfs.namenode.name.dir </name>
<value>/data/hadoop/storage/hdfs/name</value>
</property>

<property>
<name> dfs.datanode.data.dir </name>
<value>/data/hadoop/storage/hdfs/data</value>
</property>

/*配置副本数*/
<property>
<name>dfs.replication</name>
<value>2</value>
</property>

<property>
<name>dfs.webhdfs.enabled</name>
<value>true</value>
</property>

</configuration>
```

### mapred-site.xml

``` bash
$ vim $HADOOP_CONF_DIR/mapred-site.xml
```

``` xml
<?xml version=”1.0″?>
<?xml-stylesheet type=”text/xsl” href=”configuration.xsl”?>

<configuration>

<property>
<name>mapreduce.cluster.local.dir</name>
<value>/data/hadoop/storage/mapred/local</value>
</property>

<property>
<name>mapreduce.cluster.system.dir</name>
<value>/data/hadoop/storage/mapred/system</value>
</property>

<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>

<property>
<name>mapreduce.jobhistory.address</name>
<value>master:10020</value>
</property>

<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value> master:19888</value>
</property>

</configuration>
```

### yarn-site.xml

``` bash
$ vim $HADOOP_CONF_DIR/yarn-site.xml
```

``` xml
<configuration>

<!– Site specific YARN configuration properties –>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<property>
<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>

<property>
<name>yarn.resourcemanager.address</name>
<value>master:8032</value>
</property>

<property>
<name>yarn.resourcemanager.scheduler.address</name>
<value>master:8030</value>
</property>

<property>
<name>yarn.resourcemanager.resource-tracker.address</name>
<value>master:8031</value>
</property>

<property>
<name>yarn.resourcemanager.admin.address</name>
<value>master:8033</value>
</property>

<property>
<name>yarn.resourcemanager.webapp.address</name>
<value>master:8088</value>
</property>

</configuration>
```

### 配置slaves

``` bash
$ vim $HADOOP_CONF_DIR/slaves
```

删除localhost,加入所有slave的主机名

```
slave1
slave2
```

## Spark配置

### 配置spark-env.sh

``` bash
$ cd /usr/local/spark/conf
$ mv spark-env.sh.template spark-env.sh
$ vim spark-env.sh
```

```
export JAVA_HOME=/usr/local/jdk
export SCALA_HOME=/usr/local/scala
export HADOOP_HOME=/usr/local/hadoop
export SPARK_WORKER_MEMORY=2g
export SPARK_MASTER_IP=192.168.188.131
export MASTER=spark://192.168.188.131:7070
```

### 配置slaves

添加`worker`节点的主机名列表

``` bash
$ vim slaves
```

```
slave1
slave2
```

### 配置log4j

``` bash
$ mv log4j.properties.template log4j.properties
```

## 集群网络环境介绍及快速部署

集群包含三个节点：`1个master`,`2个slave`,节点之间局域网连接,可以`相互ping通`。

### 拷贝虚拟机文件夹

关闭master虚拟机,把master文件夹,拷贝2份,并命名为`slave1`,`slave2`。

### 设置虚拟机名称

用VMware打开每个slave,设置其虚拟机的名字

打开操作系统,当弹出对话框时,选择”我已复制该虚拟机”

### 修改hostname

``` bash
$ vim /etc/sysconfig/network
```

分别修改为`slave1`、`slave2`

### Centos克隆后网卡问题

由于克隆原因,克隆出来的虚拟机默认连接eth1网卡,所以需要动手改为`eth0`

#### 修改`/etc/udev/rules.d/70-persistent-net.rules`

- 拷贝eth1的硬件地址到eth0
- 删除eth1信息

#### 配置/etc/sysconfig/network-scripts/ifcfg-eth0

将`/etc/udev/rules.d/70-persistent-net.rules`中的`ATTR地址`拷贝到本文件的`HWADDR`中

```
DEVICE=”eth0″
BOOTPROTO=”static”
HWADDR=”00:0C:29:91:42:2C”
MTU=”1500″
NM_CONTROLLED=”yes”
ONBOOT=”yes”
IPADDR=192.168.152.101
NETMASK=255.255.255.0
GATEWAY=192.168.152.2
```

#### reboot

## 拷贝hadoop配置文件

如果配置文件书写错误,可以在namenode上改好之后使用以下命令拷贝到slave1、slave2节点上。

``` bash
for target in slave1 slave2
do
scp -r /usr/local/hadoop/etc/hadoop $target:/usr/local/hadoop/etc
done
```

## ssh免密码登录

### 生成rsa密钥对

所有节点用`hadoop用户`登陆

``` bash
$ su hadoop
```

并执行命令`ssh-keygen -t rsa`,这将在`/home/hadoop/.ssh/`目录下生成一个私钥`id_rsa`和一个公钥`id_rsa.pub`。

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ hadoop /.ssh/id_rsa): 默认路径
Enter passphrase (empty for no passphrase): 回车,空密码
Enter same passphrase again:
Your identification has been saved in /home/ hadoop /.ssh/id_rsa.
Your public key has been saved in /home/ hadoop /.ssh/id_rsa.pub.
```

这将在`/home/hd_space/.ssh/`目录下生成一个私钥`id_rsa`和一个公钥`id_rsa.pub`。

### 生成authorized_keys

``` bash
$ cd /home/hadoop/.ssh/
$ cat id_rsa.pub >> authorized_keys
$ chmod 644 authorized_keys
```

### 将`authorized_keys`复制到`slave`

使用SSH协议将master的公钥信息`authorized_keys`复制到所有slave的`.ssh`目录下。

``` bash
$ scp authorized_keys data节点ip地址:/home/hd_space/.ssh
```

``` bash
$ scp authorized_keys hadoop@slave1:/home/hadoop/.ssh/authorized_keys
$ scp authorized_keys hadoop@slave2:/home/hadoop/.ssh/authorized_keys
```

### 使用ssh命令测试是否成功

配置完毕,在master上执行`ssh slave,所有数据节点`命令,因为ssh执行一次之后将不会再询问。在各个slave上也进行`ssh master`命令。

## Hadoop集群启动

### 格式化

``` bash
$ hadoop namenode -format
```

注：因为配置了环境变量,此处不需要输入hadoop命令的全路径/hadoop/bin/hadoop

执行后的结果中会提示`dfs/namehas been successfully formatted`。否则格式化失败。

### 启动hadoop

``` bash
$ start-dfs.sh
$ start-yarn.sh
```

启动成功后,分别在master和slave所在机器上使用`jps `命令查看

- 会在namenode机器上看到`namenode`,`secondaryNamenode`,`ResourceManager`
- 会在slave1所在机器上看到`DataNode`, `NodeManager`
- 会在slave2所在机器上看到`DataNode`,`NodeManager`

否则启动失败,检查配置是否有问题。

### 查看集群状态

``` bash
$ hdfs dfsadmin -report
```

### 停止hadoop

``` bash
$ ./sbin/stop-dfs.sh
$ ./sbin/stop-yarn.sh
```

### 查看HDFS

http://192.168.188.132:50070/dfshealth.jsp

## Spark集群启动

### 启动Spark

在`Master`节点上执行

``` bash
$ cd /usr/local/spark && ./sbin/start-all.sh
```

### 检查进程是否启动

``` bash
$ jps
```

在master节点上出现`Master`,在slave节点上出现`Worker`.

### 相关测试

访问监控页面URL

