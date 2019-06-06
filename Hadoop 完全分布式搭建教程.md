# Hadoop 完全分布式搭建教程

硬件配置

三台Centos7系统的虚拟机，并配置root权限。

|  主机  |      IP      |
| :----: | :----------: |
| master | 172.16.30.61 |
| slave1 | 172.16.30.62 |
| slave2 | 172.16.30.63 |

## 前期配置（每台主机都要进行的操作）

------

### 关闭防火墙

```shell
systemctl stop firewalld
systemctl disable firewalld
```

----

### 关闭 selinux

修改/etc/selinux/config 文件

将SELINUX=enforcing改为SELINUX=disabled

---------

### 配置主机名映射

```shell
vim /etc/hosts

172.16.30.61 master
172.16.30.62 slave1
172.16.30.63 slave2
```

---------------

### 配置主机名

```shell
vim /etc/sysconfig/network

NETWORKING=yes
HOSTNAME=master ####每台主机填写对应的主机名称 slave1 slave2 master
```

----------

### 配置免密登陆

```shell
ssh-keygen
ssh-copy-id master
ssh-copy-id slave1
ssh-copy-id slave2
```

-----------------

### 重启主机使配置生效

```shell
reboot
```

-----------



## 下载java和Hadoop安装包

------

新建一个存放安装包的目录

```shell
mkdir /root/data
```

将jdk1.8 和 hadoop-2.6.5 的安装包下载到/root/data目录下

新建一个存放软件的目录

```shell
mkdir /root/app
```

将安装包分别解压到指定目录

```shell
tar -zxvf /root/data/jdk-8u211-linux-x64.tar.gz -C /root/app
tar -zxvf /root/data/hadoop-2.6.5.tar.gz -C /root/app
mv /root/app/jdk-8u211-linux-x64 /root/app/jdk1.8
mv /root/app/hadoop-2.6.5 /root/app/hadoop
```

-----------



## 配置JAVA_HOME 和 HADOOP_HOME

-------------------

打开/etc/profile文件并追加以下内容

```shell
export JAVA_HOME=/root/app/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin

export HADOOP_HOME=/root/app/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
```

使配置文件生效

```shell
source /etc/profile
```

------------



## 配置Hadoop

-----

修改/root/app/hadoop/etc/hadoop/core-site.xml

```xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://master:8020</value>
	</property>
	<property>
		<name>io.file.buffer.size</name>
		<value>131072</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>file:/root/app/hadoop/tmp</value>
	</property>
</configuration>
```

修改/root/app/hadoop/etc/hadoop/hadoop-env.sh

```shell
修改 export JAVA_HOME=/root/app/jdk1.8
```

修改/root/app/hadoop/etc/hadoop/slaves

```shell
slave1
slave2
```

修改/root/app/hadoop/etc/hadoop/hdfs-site.xml

```xml
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>master:9001</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/root/app/hadoop/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/root/app/hadoop/dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
    <!-- 配置zookeeper
        <property>
                <name>ha.zookeeper.quorum</name>
                <value>master:2183,slave1:2183,slave2:2183</value>
        </property>
	-->
</configuration>
```

修改/root/app/hadoop/etc/hadoop/mapred-site.xml

```shell
cp mapred-site.xml.template mapred-site.xml
```

```xml
<configuration>
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
                <value>master:19888</value>
        </property>
</configuration>
```

修改/root/app/hadoop/etc/hadoop/yarn-site.xml

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
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

---------



## 初始化Hadoop

------

创建以下几个目录

```shell
mkdir /root/app/hadoop/dfs
mkdir /root/app/hadoop/dfs/name
mkdir /root/app/hadoop/dfs/data
mkdir /root/app/hadoop/tmp
## 初始化Hadoop
hadoop namenode -format
```

-------------



## 启动Hadoop

--------------

```shell
cd /root/app/hadoop/sbin
./start-all.sh
```

### 可使用jps查看各个节点启动的进程



-------------------------END-------------------------

