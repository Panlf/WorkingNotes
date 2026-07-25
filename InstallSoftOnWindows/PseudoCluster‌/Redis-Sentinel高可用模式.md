# Redis-Sentinel高可用模式

配置一Master二Slave三Sentinel的模式

```text
master：127.0.0.1:6000 【初始化master】

slave：127.0.0.1:6001  127.0.0.1:6002

sentinel：127.0.0.1:26000  127.0.0.1:26001  127.0.0.1:26002
```

1、准备Windows下的Redis文件，这次采用一个软件多配置文件的模式。

2、指定Master，修改redis.conf文件

```conf
port 6000
```

Master为6000端口

3、指定Slave，复制redis.conf修改为redis6001.conf和redis6002.conf.

```conf
port 6001
# 指定master为本机的6000端口
slaveof 127.0.0.1 6000  
```

修改6001为6002端口即可

4、指定Sentinel，新建sentinel.conf、sentinel26001.conf和sentinel26002.conf

```cmd
port 26000 // 当前Sentinel服务运行的端口
sentinel monitor mymaster 127.0.0.1 6000 2 
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 15000
```

同理修改sentinel26001.conf和sentinel26002.conf，只需修改端口即可

上述配置的解释

```text
sentinel monitor [master-group-name] [ip] [port] [quorum]
 master-group-name：master名称（可以自定义）
 ip port : IP地址和端口号
 quorun：票数，Sentinel需要协商同意master是否可到达的数量。
 
sentinel <选项的名字> <主服务器的名字> <选项的值>

down-after-milliseconds 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。
parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步。
failover-timeout 选项指定如果在该时间（ms）内未能完成failover操作，则认为该failover失败
```

5、启动程序

startMaterSlave.bat

```cmd
start cmd /k redis-server.exe redis.conf
start cmd /k redis-server.exe redis6001.conf
start cmd /k redis-server.exe redis6002.conf
```

startSentinel

```cmd
start cmd /k redis-server.exe sentinel.conf --sentinel
start cmd /k redis-server.exe sentinel26001.conf --sentinel
start cmd /k redis-server.exe sentinel26002.conf --sentinel
```

6、点击启动查看状态

在`redis-cli.exe`目录中，`cmd`窗口输入`redis-cli.exe -h 127.0.0.1 -p 6000`，输入`info replication`

```conf
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6001,state=online,offset=15200,lag=1
slave1:ip=127.0.0.1,port=6002,state=online,offset=15333,lag=1
master_repl_offset:15599
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:15598
```

在新的`cmd`窗口，输入`redis-cli.exe -h 127.0.0.1 -p 26000`，输入`info sentinel`

```conf
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6000,slaves=2,sentinels=3
```

上述可知已经配置完毕。

sentinel是redis高可用的解决方案，sentinel系统可以监视一个或者多个redis master服务，以及这些master服务的所有从服务；当某个master服务下线时，自动将该master下的某个从服务升级为master服务替代已下线的master服务继续处理请求。

sentinel可以让redis实现主从复制，当一个集群中的master失效之后，sentinel可以选举出一个新的master用于自动接替master的工作，集群中的其他redis服务器自动指向新的master同步数据。一般建议sentinel采取奇数台，防止某一台sentinel无法连接到master导致误切换。
