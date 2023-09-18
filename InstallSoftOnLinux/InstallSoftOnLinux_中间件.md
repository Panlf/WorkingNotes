## 在Linux下安装软件_中间件

### 1、Redis安装

环境

- Redis 6.0.8
- Ubuntu 18.04

[官网](https://redis.io/download)上提供编译安装
```
$ wget http://download.redis.io/releases/redis-6.0.8.tar.gz
$ tar xzf redis-6.0.8.tar.gz
$ cd redis-6.0.8
$ make
```

但是编译安装，前提是需要安装`make`和`gcc`命令，不然无法编译。

Ubuntu安装gcc
```
默认的Ubuntu存储库包含一个名为build-essential的元包，它包含GCC编译器以及编译软件所需的许多库和其他实用程序。

1、sudo apt update
2、sudo apt install build-essential
3、gcc --version
```

编译过程中也发现了一个错误
```
d src && make all
make[1]: Entering directory '/opt/redis/redis-6.0.8/src'
/bin/sh: 1: pkg-config: not found
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory
 #include <jemalloc/jemalloc.h>
          ^~~~~~~~~~~~~~~~~~~~~
compilation terminated.
Makefile:317: recipe for target 'adlist.o' failed
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory '/opt/redis/redis-6.0.8/src'
Makefile:6: recipe for target 'all' failed
make: *** [all] Error 2
```

原因是jemalloc重载了Linux下的ANSI C的malloc和free函数。解决办法：make时添加参数。
```
make MALLOC=libc
```

上述即可启动Redis。


### 2、Hadoop安装
1、配置jdk11

优先配置JDK，这里采用的是JDK11
```
/usr/local/jdk-11
```

2、配置hadoop
```
export HADOOP_HOME=/usr/local/hadoop-3.3.1
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

3、修改Hadoop的核心配置文件 【core-site.xml】

core-site.xml是Hadoop的核心配置文件，里面可以配置HDFS（Hadoop分布式文件系统）的NameNode的地址和数据存储目录
```
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
```
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
```
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
```
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
```
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
```
useradd hadoop
passwd hadoop
密码panlf123
```

权限授予
```
chown -R hadoop:hadoop /usr/local/hadoop-3.3.1/

```


```
localhost: Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
Starting secondary namenodes [localhost.localdomain]
localhost.localdomain: Warning: Permanently added 'localhost.localdomain' (ECDSA) to the list of known hosts.
localhost.localdomain: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```
是因为没有配置SSH免密码登录录的问题，所以配置SSH免密码登录

```
ssh-keygen -t rsa
回车执行该命令后再接着三个回车 然后将公钥拷贝给自己

ssh-copy-id localhost
输入当前hadoop用户的密码，完成SSH免密码登录配置
```

启动HDFS
```
start-dfs.sh
```

启动YARN
```
start-yarn.sh
```

出现问题
```
192.168.164.130: ERROR: JAVA_HOME is not set and could not be found.
```

hadoop-env.sh、yarn-env.sh、mapred-env.sh 配置 
```
JAVA_HOME=/usr/local/jdk-11
```

8、访问

打开Hadoop HDFS的管理页面
使用浏览器访问hdfs管理界面：
```
192.168.164.130:50070
```
打开Hadoop YARN的管理页面
使用浏览器访问yarn管理界面：
```
192.168.164.130:8088
```

9、关闭防火墙
```
systemctl stop firewalld      #关闭防火墙服务网
systemctl disable firewalld   #设置防火墙服务开机不启动
```

### 3、MinIO分布式部署
部署脚本
```
RUNNING_USER=root
MINIO_HOME=/opt/minio
MINIO_HOST=192.168.22.10
#accesskey and secretkey
ACCESS_KEY=minio
SECRET_KEY=minio123

for i in {01..04}; do
    START_CMD="MINIO_ACCESS_KEY=${ACCESS_KEY} MINIO_SECRET_KEY=${SECRET_KEY} nohup ${MINIO_HOME}/minio  server --address "${MINIO_HOST}:90${i}" http://${MINIO_HOST}:9001/opt/min-data1 http://${MINIO_HOST}:9002/opt/min-data2 http://${MINIO_HOST}:9003/opt/min-data3 http://${MINIO_HOST}:9004/opt/min-data4 > ${MINIO_HOME}/minio-90${i}.log 2>&1 &"
    su - ${RUNNING_USER} -c "${START_CMD}"
done
```

- 所有运行分布式 MinIO 的节点需要具有相同的访问密钥和秘密密钥才能连接。建议在执行 MINIO 服务器命令之前，将访问密钥作为环境变量，MINIO access key 和 MINIO secret key 导出到所有节点上 。
- Minio 创建 4 到 16 个驱动器的擦除编码集。
- Minio 选择最大的 EC 集大小，该集大小除以给定的驱动器总数。例如，8 个驱动器将用作一个大小为 8 的 EC 集，而不是两个大小为 4 的 EC 集 。
- 建议所有运行分布式 MinIO 设置的节点都是同构的，即相同的操作系统、相同数量的磁盘和相同的网络互连 。
- 运行分布式 MinIO 实例的服务器时间差不应超过 15 分钟。

```
upstream http_minio {
    server 192.168.22.10:9001;
    server 192.168.22.10:9002;
    server 192.168.22.10:9003;
    server 192.168.22.10:9004;
}

server{
    listen       8888;
    server_name  192.168.22.10;

    ignore_invalid_headers off;
    client_max_body_size 0;
    proxy_buffering off;

    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-Host  $host:$server_port;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto  $http_x_forwarded_proto;

        proxy_connect_timeout 300;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_ignore_client_abort on;

        proxy_pass http://http_minio;
    }
}
```
