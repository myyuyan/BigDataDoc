# Hive 组件部署

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





## 安装MySQL（在master节点操作）

```bash
cd /root/data
wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.16/mysql-connector-java-8.0.16.jar
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
yum install mysql80-community-release-el7-3.noarch.rpm
yum install mysql-server
## 修改配置文件，跳过密码登录
vim /etc/my.cnf
## 在[mysqld]的后追加一行，内容为skip-grant-tables，保存退出。
## 启动MySQL服务
service mysqld restart
```

创建数据库

```shell
mysql -u root -p
create database hive;
exit
```



配置数据库可以远程登录并修改密码

```mysql
mysql -u root -p
use mysql
update user set host = '%' where user = 'root';
alter user'root'@'%' IDENTIFIED BY 'Tjs2019.';
exit
## 修改配置文件
vim /etc/my.cnf
## 在[mysqld]的后一行，删除内容为skip-grant-tables，保存退出。
## 重新启动MySQL服务
service mysqld restart
```





## 配置Hive（在master节点操作）

```shell
tar -zxvf /root/data/apache-hive-2.3.5-bin.tar.gz -C /root/app
cd /root/app
mv apache-hive-2.3.5-bin hive
## 修改配置文件
vim /etc/profile
## 添加以下内容
export HIVE_HOME=/root/app/hive
export PATH=$PATH:$HIVE_HOME/bin
## 生效配置文件
source /etc/profile
```

修改 hive 配置文件

```shell
cd /root/app/hive/conf
vim hive-site.xml
```

配置文件内容如下

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
    <name>javax.jdo.option.ConnectionURL</name>    <value>jdbc:mysql://localhost:3306/hive?characterEncoding=UTF-8</value> 
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>Tjs2019.</value>
</property>
</configuration>
```

vim /root/app/hive/conf/hive-env.sh

```shell
## 追加以下内容

HADOOP_HOME=/root/app/hadoop
export HIVE_CONF_DIR=/root/app/hive/conf
export HIVE_AUX_JARS_PATH=/root/app/hive/lib
```



拷贝 JDBC 驱动包

```she
cp /root/data/mysql-connector-java-8.0.16.jar /root/app/hive/lib/
```

给子节点分发hive

```shell
scp -r  hive slave1:/root/app
scp -r  hive slave2:/root/app
```





## 配置子节点的hive（在slave1和slave2执行）

修改 hive 配置文件

```shell
cd /root/app/hive/conf
vim hive-site.xml
```

配置文件内容如下

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
    <name>javax.jdo.option.ConnectionURL</name>    <value>jdbc:mysql://master:3306/hive?characterEncoding=UTF-8</value> 
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>Tjs2019.</value>
</property>
<property>  
    <name>hive.metastore.uris</name>  
    <value>thrift://master:9083</value>  
</property>
</configuration>
```

在子节点上配置hive环境变量

```shell
## 修改配置文件
vim /etc/profile
## 添加以下内容
export HIVE_HOME=/root/app/hive
export PATH=$PATH:$HIVE_HOME/bin
## 生效配置文件
source /etc/profile
```





## 初始化 hive 数据库（在master节点操作）

```shell
schematool -dbType mysql -initSchema
```





# END

