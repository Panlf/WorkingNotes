## 在Windows下安装软件_中间件

### 1、RabbitMQ安装

RabbitMQ安装需要Erlang环境，所以需要准备RabbitMQ、Erlang两个软件，两个软件都可以在官网下载。

- [RabbitMQ](https://www.rabbitmq.com/download.html)
- [Erlang](http://www.erlang.org/downloads)

1、Erlang安装，按照提示安装即可

2、配置Erlang环境变量，新建ERLANG_HOME为D:\erl10.3，添加到path为%ERLANG_HOME%/bin;

3、在cmd下输入erl即可查看是否配置成功

3、安装RabbitMQ，按照提示安装即可

4、进入RabbitMQ的sbin目录-D:\RabbitMQ Server\rabbitmq_server-3.7.14\sbin

5、在此cmd下输入rabbitmq-server 启动RabbitMQ服务

6、继续在cmd下输入rabbitmq-plugins.bat enable rabbitmq_management 开启可视化插件

7、访问127.0.0.1:15672即可访问，用户名和密码都是guest

#### 其他小点
- rabbitmq-plugins list 查看已经安装的插件
- rabbitmq-plugins.bat disable rabbitmq_management 禁用可视化插件
- rabbitmq-plugins.bat enable rabbitmq_management 启用可视化插件
- rabbitmqctl status 检查RabbitMQ运行状态
- rabbitmqctl stop 停止RabbitMQ运行
- rabbitmq-server -detached 重新启动RabbitMQ并在后台运行

### 2、Kafka安装

1、安装配置`Zookeeper`，下载[Zookeeper](https://zookeeper.apache.org/releases.html)，修改`zoo.cfg`中的`dataDir`地址，也可修改端口。

2、点击`zkServer.cmd`，启动`Zookeeper`

2、下载[Kafka](http://kafka.apache.org/downloads)，解压文件进入`config`文件夹，修改`server.properties`中的日志地址
```
log.dirs=D:/kafka/kafka_2.11-2.2.0/kafka-logs
```

3、修改zookeeper.properties
```
dataDir=D:/kafka/kafka_2.11-2.2.0/data
clientPort=127.0.0.1:2181
```

4、在kafka目录(D:\kafka\kafka_2.11-2.2.0)中写启动文件

startkafka.bat
```
.\bin\windows\kafka-server-start.bat .\config\server.properties
```
点击上述bat文件即可，前提是Zookeeper已经启动

### 3、ELK安装

ELK即Elasticsearch、Logstash、 Kibana，在windows下安装比较简单，下载解压即可，不过使用需要具有Java环境，目前官网是7版本需要1.8以上Jdk。

1、[官网下载](https://www.elastic.co/cn/downloads/)

2、下载Elasticsearch并解压，启动`\Elasticesarch\bin\elasticsearch.bat`，然后访问`localhost:9200`，观察启动成功

3、下载Logstash，并在`\logstash\bin`新建`logstash.conf`。这里我是监控Nginx日志，所以以下配置文件是关于监控Nginx日志
```
input {  
	file {
		path => "D:/Nginx/nginx-1.14.1-images/logs/*.log"
		start_position => "beginning"
        ignore_older => 0
		type => "nginx_log"
	}
}

filter{  
  
}

output {

	elasticsearch {     
		hosts => ["127.0.0.1:9200"]
	}
	stdout {codec => rubydebug}
}
```
配置完毕启动` logstash -f logstash.conf >> C:\Users\Panlf\Desktop\log.txt `， 因为有时候配置文件错误会导致启动不成功，所以我自己会在命令结束后添加信息到日志文件中。之后访问`localhost:9600`

4、下载Kibana，并启动`\kibana\bin\kibana.bat`，访问`localhost:5601`，并配置`index-patterns`即可观察到可视化的Nginx日志数据。

### 4、Mariadb配置主从复制
#### 环境及版本

- Mariadb 10.4.6 下载可访问[官网](https://mariadb.org)
- Windows7 64位

#### 安装主数据库

##### 配置文件
```
[client]
port	= 3310
socket	= D:/Mariadb/mariadb_master/tmp/mysql.sock


[mysqld]
port	= 3310
socket	= D:/Mariadb/mariadb_master/tmp/mysql.sock
datadir = D:/Mariadb/mariadb_master/data
server-id=1
key_buffer_size = 384M
max_allowed_packet= 1M
table_open_cache=512
sort_buffer_size=64k
read_buffer_size = 1M
read_rnd_buffer_size = 512K
thread_cache_size = 8
query_cache_size = 16M
log-bin=mysql-bin
lower_case_table_names=1
```

##### 安装服务
```
mysqld.exe --install Mmariadb
```

##### 数据库初始化
```
mysql_install_db
```

##### 启动数据库
```
net start Mmariadb
```

##### 设置ROOT密码
```
1、mysql -uroot 
2、use mysql;
3、SET password for 'root'@'localhost'=password('root'); 
4、exit; 
5、重启即可 - net stop Mmariadb |  net start Mmariadb
```

##### 设置备份账号
```
1、登录主数据库
2、grant replication slave on *.* to 'slaver'@'%' identified by '123456';
3、flush privileges;
4、重启
```

##### 查看状态
```
1、登录
2、show master status\G
3、以下结果为正常
*************************** 1. row ***************************
            File: mysql-bin.000003
        Position: 342
    Binlog_Do_DB:
Binlog_Ignore_DB:
```

#### 安装从数据库

##### 配置文件

主要修改端口、server-id、datadir及socket等相关参数

```
[client]
port	= 3311
socket	= D:/Mariadb/mariadb_slave/tmp/mysql.sock


[mysqld]
port	= 3311
socket	= D:/Mariadb/mariadb_slave/tmp/mysql.sock
datadir = D:/Mariadb/mariadb_slave/data
server-id=2
key_buffer_size = 384M
max_allowed_packet= 1M
table_open_cache=512
sort_buffer_size=64k
read_buffer_size = 1M
read_rnd_buffer_size = 512K
thread_cache_size = 8
query_cache_size = 16M
log-bin=mysql-bin
lower_case_table_names=1
```

##### 安装服务
```
mysqld.exe --install Smariadb
```

##### 数据库初始化
```
mysql_install_db
```

##### 启动数据库
```
net start Smariadb
```

##### 设置ROOT密码
```
1、mysql -uroot 
2、use mysql;
3、SET password for 'root'@'localhost'=password('root'); 
4、exit; 
5、重启即可 - net stop Smariadb |  net start Smariadb
```

##### 主从配置
```
1、登录从数据库
2、stop slave；
3、change master to master_host='127.0.0.1',master_user='slaver',master_port=3310,master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=342;
4、start slave
5、show slave status\G
6、结果
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 127.0.0.1
                   Master_User: slaver
                   Master_Port: 3310
                 Connect_Retry: 60
               Master_Log_File: mysql-bin.000003
           Read_Master_Log_Pos: 610
                Relay_Log_File: Panlf-PC-relay-bin.000002
                 Relay_Log_Pos: 823
         Relay_Master_Log_File: mysql-bin.000003
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table:
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 610
               Relay_Log_Space: 1135
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: 0
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 1
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: conservative
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for the sl
ave I/O thread to update it
              Slave_DDL_Groups: 1
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 0



Slave_IO_Running: Yes
Slave_SQL_Running: Yes
这两个值都是YES即表示主从配置已经成功
```

**参数解释**
```
 1)master_host是指主服务器的IP

 2)master_user是指使用哪个用户登录主服务器

 3)master_password是指登录密码

 4)master_log_file是指在第4个步骤中的File名称

 5)master_log_pos是指在第4个步骤中的Position

 6)master_port是指主服务器的端口，默认是3306，如果不是3306则需要自己指定
```

#### 注意问题

```
如果Slave_SQL_Running:No

1、程序可能在slaves上进行了写操作
2、也可能是slave机器重启后，事务回滚造成的

解决方案一
一般是事务回滚造成的
stop slave
。。。。
start slave

解决方案二
首先停掉Slave服务 slave stop
到主服务器查看主机状态
记录file和positiond对应的值

然后到slave服务器上执行手动同步
change 。。。。
start slave
```
