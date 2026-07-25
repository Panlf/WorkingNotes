# Hadoop安装

1、配置jdk11

优先配置JDK，这里采用的是JDK11

```shell
/usr/local/jdk-11
```

2、配置hadoop

```shell
export HADOOP_HOME=/usr/local/hadoop-3.3.1
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

3、修改Hadoop的核心配置文件 【core-site.xml】

core-site.xml是Hadoop的核心配置文件，里面可以配置HDFS（Hadoop分布式文件系统）的NameNode的地址和数据存储目录

```xml
<property>
 <name>fs.defaultFS</name>
 <value>hdfs://192.168.164.130:9000</value>
 <description>指定HDFS Master的通信地址，默认端口</description>
</property>
<property>
 <name>hadoop.tmp.dir</name>
 <value>/usr/local/hadoop-3.3.1/tmp</value>
 <description>指定hadoop运行时产生文件的存储路径</description>
</property>
```

4、修改HDFS的配置文件 【hdfs-site.xml】

hdfs-site.xml是HDFS的配置文件，由于当前在一台机器上配置的Hadoop伪分布式，所以这里修改HDFS的副本为1即数据只保存一份

```xml
<configuration>
    <!-- HDFS的副本为1，即数据只保存一份 -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
 <property>
  <name>dfs.http.address</name>
  <value>0.0.0.0:50070</value>
 </property>
</configuration>
```

5、 修改MapReduce的配置文件  【mapred-site.xml】

mapred-site.xml里面可以配置MapReduce框架运行在YARN资源调度系统上

```xml
<configuration>
    <!-- 指定MapReduce运行在YARN上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

6、修改YRAR的配置文件 【yarn-site.xml】

yarn-site.xml是配置YARN的相关配置的文件

```xml
<configuration>
    <!-- 分别指定ResouceManager的地址 -->
    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>192.168.164.130</value>
    </property>
    <!-- 分别指定MapReduce的方式 -->
    <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

7、用户

```text
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting secondary namenodes [localhost.localdomain]
ERROR: Attempting to operate on hdfs secondarynamenode as root
ERROR: but there is no HDFS_SECONDARYNAMENODE_USER defined. Aborting operation.
```

需要定义一个用户，并配置密码

```shell
useradd hadoop
passwd hadoop
```

权限授予

```shell
chown -R hadoop:hadoop /usr/local/hadoop-3.3.1/
```

```text
localhost: Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
Starting secondary namenodes [localhost.localdomain]
localhost.localdomain: Warning: Permanently added 'localhost.localdomain' (ECDSA) to the list of known hosts.
localhost.localdomain: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```

是因为没有配置SSH免密码登录录的问题，所以配置SSH免密码登录

```shell
ssh-keygen -t rsa
# 回车执行该命令后再接着三个回车 然后将公钥拷贝给自己

ssh-copy-id localhost
# 输入当前hadoop用户的密码，完成SSH免密码登录配置
```

启动HDFS

```shell
start-dfs.sh
```

启动YARN

```shell
start-yarn.sh
```

出现问题

```text
192.168.164.130: ERROR: JAVA_HOME is not set and could not be found.
```

hadoop-env.sh、yarn-env.sh、mapred-env.sh 配置

```shell
JAVA_HOME=/usr/local/jdk-11
```

8、访问

打开Hadoop HDFS的管理页面
使用浏览器访问hdfs管理界面：

```text
192.168.164.130:50070
```

打开Hadoop YARN的管理页面
使用浏览器访问yarn管理界面：

```text
192.168.164.130:8088
```

9、关闭防火墙

```shell
systemctl stop firewalld      #关闭防火墙服务网
systemctl disable firewalld   #设置防火墙服务开机不启动
```
