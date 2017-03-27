title: Hadoop, Hive, Tez 搭建笔记
category: 大数据
date: 2016-03-15
tags: [hadoop, Hive, Tez ]
author: Weiyi WANG

---
详细介绍了一个Hadoop，Hive和Tez集群的搭建过程。

<!--more-->

---


# Hadoop Hive Tez 搭建

集群192.168.100.[205-220],主节点205,其余为从节点，以用户experiment为例

## Hadoop

### /etc/hosts

hadoop是以主机名进行通信的，需要配置所有机器的/etc/hosts

```
192.168.100.205 slave205
192.168.100.206 slave206
192.168.100.207 slave207
192.168.100.208 slave208
192.168.100.209 slave209
192.168.100.210 slave210
192.168.100.211 slave211
192.168.100.212 slave212
192.168.100.213 slave213
192.168.100.214 slave214
192.168.100.215 slave215
192.168.100.216 slave216
192.168.100.217 slave217
192.168.100.218 slave218
192.168.100.219 slave219
192.168.100.220 slave220

```

### ssh免密码登陆

```
ssh-keygen -t rsa

cat id_rsa.pub >> ~/.ssh/authorized_keys
```

### jdk

openjdk和oracle-jdk都行

### 下载并修改配置文件

slaves

```
slave206
slave207
slave208
slave209
slave210
slave211
slave212
slave213
slave214
slave215
slave216
slave217
slave218
slave219
slave220

```

core-site.xml

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://slave205:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/home/experiment/hadoop/tmp</value>
    </property>
</configuration>

```

hdfs-site.xml

```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/home/experiment/hadoop/name</value>
    </property>
    <property>
        <name>dfs.namenode.data.dir</name>
        <value>file:/home/experiment/hadoop/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>slave205:50090</value>
    </property>
</configuration>

```

yarn-site.xml

```
<configuration>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>slave205:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>slave205:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>slave205:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>slave205:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>slave205:8088</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.nodemanager.delete.debug-delay-sec</name>
        <value>1200</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
</configuration>

```

mapred-site.xml

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>slave205:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>slave205:19888</value>
    </property>
</configuration>

```

### 环境变量

vim ~/.bashrc

```
HADOOP_HOME=/home/experiment/hadoop-2.6.4
JAVA_HOME=/usr/java/jdk1.8.0_73
CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib
PATH=$PATH:$JAVA_HOME/bin:$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"

export JAVA_HOME CLASSPATH HADOOP_HOME PATH

```

### scp 到从节点对应的目录


### 启动

```
hdfs namenode -format
start-dfs.sh
start-yarn.sh

```

## Hive

### hive下载解压

### 环境变量

```
HIVE_HOME=/home/experiment/apache-hive-1.2.1-bin
PATH=$PATH:$HIVE_HOME/bin

export HADOOP_USER_CLASSPATH_FIRST=true
export HIVE_HOME PATH

```

### mysql安装

```
# 下载源码解压，然后执行下面的命令以生成Makefile

cmake \
-DCMAKE_INSTALL_PREFIX=/home/experiment/mysql \
-DMYSQL_UNIX_ADDR=/home/experiment/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DMYSQL_home/experimentDIR=/home/experiment/mysql/home/experiment \
-DMYSQL_TCP_PORT=3306 \
-DENABLE_DOWNLOADS=1

# 如果报错找不到CMakeCache.txt，则需要安装ncurses-devel，安装后进行编译

make && make install

sudo ln -s /home/experiment/mysql/lib/libmysqlclient.so.18 /usr/lib/libmysqlclient.so.18
sudo ln -s /home/experiment/mysql.sock /tmp/mysql.sock
sudo cp /home/experiment/mysql/support-files/my-default.cnf /etc/my.cnf
sudo cp /home/experiment/mysql/support-files/mysql.server /etc/init.d/mysqld
sudo chmod +x /etc/init.d/mysqld
sudo ln -s /home/experiment/mysql/bin/mysql /usr/bin/

/home/experiment/mysql/scripts/mysql_install_db --user=experiment --defaults-file=/etc/my.cnf --basedir=/home/experiment/mysql --datadir=/home/experiment/mysql/data

service mysqld start

mysql -uroot -h127.0.0.1 -p

mysql> SET PASSWORD = PASSWORD('123456');
CREATE USER 'hive' IDENTIFIED BY 'hive';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'slave205' WITH GRANT OPTION;

flush privileges;

mysql -h hadoop-master -uhive
mysql>set password = password('hive');
create database hive;
alter database hive character set latin1
```

### 修改hive配置文件

先cp hive-default.xml.template hive-default.xml

再vim hive-site.xml

```
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://slave205:3306/hive?createDatabaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>username to use against metastore database</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value>
    <description>password to use against metastore database</description>
</property>

```

### 下载jdbc

cp mysql-connector-java-5.1.33-bin.jar apache-hive-1.2.1-bin/lib/

### 让hive支持update

修改hive-site.xml

```
<property>
    <name>hive.support.concurrency</name>
    <value>true</value>
</property>
<property>
    <name>hive.enforce.bucketing</name>
    <value>true</value>
</property>
<property>
    <name>hive.exec.dynamic.partition.mode</name>
    <value>nonstrict</value>
</property>
<property>
    <name>hive.txn.manager</name>
    <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
<property>
    <name>hive.compactor.initiator.on</name>
    <value>true</value>
</property>
<property>
    <name>hive.compactor.worker.threads</name>
    <value>2</value>
</property>

```

### 启动hive metastore和hiveserver2

hive metastore不启动也可以使用hive，metastore的作用是给hive的客户端使用

hiveserver2是hive的thrift服务器可以让远程java、python等客户端调用hive的API，需要在hive-site.xml中修改hive.server2.thrift.bind.host为slave205，默认是localhost

## Tez

[官方安装指南](http://tez.apache.org/install.html)

### 编译

下载tez源码解压，修改pom.xml中的hadoop.version为具体的hadoop版本号

编译

```
mvn clean package -DskipTests=true -Dmaven.javadoc.skip=true
```

编译需要jdk6+，maven3+和protocol buffers 2.5.0，编译期间可能会遇到下载失败或者ssl超时，多编译几次应该可以解决。其它常见问题参考[wiki](https://cwiki.apache.org/confluence/display/TEZ/Build+errors+and+solutions)

### 环境变量

```
export TEZ_JARS=/home/experiment/tez-0.8.2
export TEZ_CONF_DIR=/home/experiment/hadoop-2.6.4/etc/hadoop
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${TEZ_CONF_DIR}:${TEZ_JARS}/*:${TEZ_JARS}/lib/*

```

### 放置编译好的tez-x.y.z.tar.gz

位于tez-dist/target目录下

```
hadoop dfs -mkdir /apps
hadoop dfs -copyFromLocal tez-x.y.z.tar.gz /apps/

```

### 修改tez-site.xml

```
<configuration>
    <property>
        <name>tez.lib.uris</name>
        <value>${fs.defaultFS}/apps/tez-0.8.2.tar.gz</value>
    </property>
    <property>
        <name>tez.use.cluster.hadoop-libs</name>
        <value>false</value>
    </property>
</configuration>

```

### 在hive中使用tez

```
hive

hive > set hive.execution.engine=tez;
	 >
```
