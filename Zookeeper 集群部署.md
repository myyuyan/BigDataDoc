# Zookeeper 集群部署

硬件配置

三台Centos7系统的虚拟机，并配置root权限。

|  主机  |      IP      |
| :----: | :----------: |
| master | 172.16.30.61 |
| slave1 | 172.16.30.62 |
| slave2 | 172.16.30.63 |

软件配置

hadoop2.6.5

jdk1.8





## 下载 Zookeeper 安装包

```sh
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```





## 安装 Zookeeper 集群（在master上配置）

解压

```shell
tar -zxvf zookeeper-3.4.14.tar.gz -C /root/app
mv /root/app/zookeeper-3.4.14 /root/app/zookeeper
```

修改配置

```shell
cp /root/app/zookeeper/conf/zoo_sample.cfg /root/app/zookeeper/conf/zoo.cfg
vim /root/app/zookeeper/conf/zoo.cfg
##配置内容如下
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/root/app/zookeeper/tmp
# the port at which the clients will connect
clientPort=2183
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
##END
```

创建一个tmp文件夹 存储zookeeper产生的数据

```shell
mkdir /root/app/zookeeper/tmp
## 创建myid
touch /root/app/zookeeper/tmp/myid
echo 1 > /root/app/zookeeper/tmp/myid
```

给其他节点分发文件

```shell
scp -r /root/app/zookeeper slave1:/root/app
scp -r /root/app/zookeeper slave2:/root/app
```

在slave1上修改myid的内容为2

```shell
echo 2 > /root/app/zookeeper/tmp/myid
```

在slave2上修改myid的内容为3

```shell
echo 3 > /root/app/zookeeper/tmp/myid
```





## 启动zookeeper 集群（在所有节点执行）

```shell
cd /root/app/zookeeper/bin
./zkServer.sh start
```





--------------



# END 

