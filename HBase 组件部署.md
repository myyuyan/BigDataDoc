# HBase 组件部署

硬件配置

三台Centos7系统的虚拟机，并配置root权限。

|  主机  |      IP      |
| :----: | :----------: |
| master | 172.16.30.61 |
| slave1 | 172.16.30.62 |
| slave2 | 172.16.30.63 |

软件配置

hadoop2.6.5

zookeeper3.4

jdk1.8



## 下载并配置Hbase2.1.5(在master主机上执行)

```shell
## 下载
cd /root/data
wget http://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.5/hbase-2.1.5-bin.tar.gz

## 解压
tar -zxvf hbase-2.1.5-bin.tar.gz -C /root/app

## 更改文件夹名称
cd /root/app/
mv hbase-2.1.5 hbase

## 配置hbase-env
cd /root/app/hbase
cp ./lib/client-facing-thirdparty/htrace-core-3.1.0-incubating.jar ./lib/
mkdir pids
vim conf/hbase-env.sh
##添加以下内容##
export JAVA_HOME=/root/app/jdk1.8
export HADOOP_HOME=/root/app/hadoop
export HBASE_CLASSPATH=$HADOOP_HOME/etc/hadoop
export HBASE_MANAGES_ZK=false
export HBASE_PID_DIR=/root/app/hbase/pids
####保存命令 esc :wq

## 配置regionservers
rm -rf conf/regionservers
vim conf/regionservers
##添加以下内容##
slave1
slave2
####保存命令 esc :wq

## 配置hbase-site.xml
cp /root/app/hadoop/etc/hadoop/core-site.xml ./conf
cp /root/app/hadoop/etc/hadoop/hdfs-site.xml ./conf
vim conf/hbase-site.xml
##添加以下内容##
```

```xml
<configuration>
        <!-- 指定ZooKeeper集群位置 -->
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>master,slave1,slave2</value>
        </property>
        <!-- Zookeeper写数据目录，与ZooKeeper集群上配置相一致 -->
        <property>
                <name>hbase.zookeeper.property.dataDir</name>
                <value>/root/app/zookeeper/tmp</value>
        </property>
        <!-- Zookeeper的端口号 -->
        <property>
                <name>hbase.zookeeper.property.clientPort</name>
                <value>2183</value>
        </property>
        <!-- RegionServers共享目录 -->
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://master:8020/hbase</value>
        </property>
        <!-- 开启分布式模式 -->
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
        <!-- 指定Hbase的master的位置 -->
        <property>
                <name>hbase.master</name>
                <value>hdfs://master:60000</value>
        </property>
        <!-- 使用本地文件系统设置为false，使用hdfs设置为true -->
        <property>
                <name>hbase.unsafe.stream.capability.enforce</name>
                <value>true</value>
        </property>
		<!-- 开启web ui  -->
        <property>
                <name>hbase.master.info.port</name>
                <value>16010</value>
        </property>
</configuration>
####保存命令 esc :wq
```

### 拷贝HBase

```shell
scp -r /root/app/hbase slave1:/root/app
scp -r /root/app/hbase slave2:/root/app
```





-----

## 配置HBase环境变量(在所有主机上配置)

```shell
vim /etc/profile
##添加以下内容##
export HBASE_HOME=/root/app/hbase
export PATH=$PATH:$HBASE_HOME/bin
####保存命令 esc :wq

## 使配置环境生效
source /etc/profile


## 此步也要执行 ##
cp /root/app/hive/lib/jline-2.12.jar /root/app/hadoop/share/hadoop/yarn/lib
rm /root/app/hadoop/share/hadoop/yarn/lib/jline-0.9.94.jar
##
```





---------

## 启动HBase(在master主机上执行)

```shell
start-hbase.sh
```

