## 在Windows下安装软件_1

有时候在测试本地代码的时候，需要自己搭建环境，然后可能没有Linux环境就需要在Windows下安装一些软件进行测试。


### 1、在Windows下单机部署Zookeeper的伪集群模式

1、在[Zookeeper官网](http://zookeeper.apache.org/releases.html)下载Windows版本

2、在对应的位置解压缩，我是在D:\zookeeper\Cluster解压出来三个Zookeeper

3、分别取名zookeeper-3.5.2-alpha-0、zookeeper-3.5.2-alpha-1、zookeeper-3.5.2-alpha-3

4、修改conf/zoo.cfg

添加数据和日志的文件夹
```
dataDir=D:/zookeeper/Cluster/zookeeper-3.5.2-alpha-0/data
dataLogDir=D:/zookeeper/Cluster/zookeeper-3.5.2-alpha-0/logs
```
修改端口号
```
clientPort=2182
```
集群配置
```
server.0=127.0.0.1:2888:3888
server.1=127.0.0.1:2889:3889
server.2=127.0.0.1:2890:3890
```
上述解释
```
server.A=B:C:D

A 表示这个是第几号服务器。
B 表示这个服务器的 ip 地址。
C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口。
D 表示的是万一集群中的 Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，
  而这个端口就是用来执行选举时服务器相互通信的端口。

如果是伪集群的配置方式，由于B都是一样，所以不同的Zookeeper实例通信端口号不能一样，所以要给它们分配不同的端口号。
```

在data目录新建一个myid的文件夹，并根据server后面的数字填入。

根据以上的步骤，配置其他两个Zookeeper即可启动。

5、启动即可

直接编写bat文件一键启动
```
start cmd /k D:\zookeeper\Cluster\zookeeper-3.5.2-alpha-0\bin\zkServer.cmd 
start cmd /k D:\zookeeper\Cluster\zookeeper-3.5.2-alpha-1\bin\zkServer.cmd 
start cmd /k D:\zookeeper\Cluster\zookeeper-3.5.2-alpha-2\bin\zkServer.cmd 
```

#### zoo.cfg的配置解释
```
tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，
          也就是每个 tickTime 时间就会发送一个心跳。
dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
dataLogDir：log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和dataDir相同的设置。
clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
maxClientCnxns：限制连接到zookeeper的客户端数量，并且限制并发连接数量，它通过ip区分不同的客户端。
initLimit：集群中的follower服务器与leader服务器之间初始连接时能容忍的最多心跳数（tickTime的数量）。
syncLimit：集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。

# 下面两个参数实现自动清理
autopurge.purgeInterval：这个参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，
                        默认是0，表示不开启自己清理功能。
autopurge.snapRetainCount：这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。
```

### 2、在Windows下单机部署Redis的伪集群模式

Redis Cluster 

Redis 3.0之后版本支持redis-cluster集群，Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

特点
- 1、所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽。
- 2、节点的fail是通过集群中超过半数的节点检测失效时才生效。
- 3、客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。
- 4、redis-cluster把所有的物理节点映射到[0-16383]slot上（不一定是平均分配）,cluster 负责维护node<->slot<->value。
- 5、Redis集群预分好16384个桶，当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中。
	

这里使用的是三主三从，即六个节点，主节点需要3个以上且奇数个，然后在Windows部署Redis CLuster需要需要四个部分

- [Redis](https://github.com/MSOpenTech/redis/releases/) - 这里使用的3.2.100版本
- [Ruby](http://railsinstaller.org/en)运行环境 - 这里使用的2.2.4版本
- Ruby环境下的Redis驱动Redis.xxx.gem - 运行gem install redis即可安装
- 创建Redis集群的ruby脚本文件redis-trib.rb - 这个文件在Redis的源码包src目录下

1、在一个目录(本文的目录为`D:\Redis\higher\cluster`)复制6份Redis，取名为Redis7001、Redis7002、Redis7003、Redis7004、Redis7005、Redis7006。其实只有一份也是可以的，就是在这一份中配置6份配置文件，然后用`redis-server.exe redis.windows.conf`启动不同的配置文件即可。

2、配置redis.windows.conf
```
port 7001     
loglevel notice   
logfile "D:/Redis/higher/cluster/log/redis7001_log.txt"      
appendonly yes
appendfilename "appendonly.7001.aof"  
cluster-enabled yes                            
cluster-config-file nodes.7001.conf
cluster-node-timeout 15000
cluster-slave-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage yes
```
上述的配置文件在六个文件中分别修改配置，只要把7001修改成对应端口即可。

配置解释
```
port 7001      #配置端口
loglevel notice    #日志级别
logfile "D:/Redis/higher/cluster/log/redis7001_log.txt" #日志文件    
appendonly yes  # 开启aof
appendfilename "appendonly.7001.aof"   #aof文件
cluster-enabled yes       # 开启集群                     
cluster-config-file nodes.7001.conf # Redis群集节点每次发生更改时自动保留群集配置（基本上为状态）的文件，以便能够 在启动时重新读取它。 该文件列出了群集中其他节点，它们的状态，持久变量等等。 由于某些消息的接收，通常会将此文件重写并刷新到磁盘上。 
cluster-node-timeout 15000 #Redis群集节点可以不可用的最长时间，而不会将其视为失败。
cluster-slave-validity-factor 10 # 如果设置为0，无论主设备和从设备之间的链路保持断开连接的时间长短，从设备都将尝试故障切换主设备。 如果该值为正值，则计算最大断开时间作为节点超时值乘以此选项提供的系数，如果该节点是从节点，则在主链路断开连接的时间超过指定的超时值时，它不会尝试启动故障切换。
cluster-migration-barrier 1 # 主设备将保持连接的最小从设备数量，以便另一个从设备迁移到不受任何从设备覆盖的主设备。
cluster-require-full-coverage yes # 如果将其设置为yes，则默认情况下，如果key的空间的某个百分比未被任何节点覆盖，则集群停止接受写入。 如果该选项设置为no，则即使只处理关于keys子集的请求，群集仍将提供查询。
```

为了启动方便在本目录下新建一个bat文件。

```
title redis-7001
redis-server.exe redis.windows.conf
```
其他只要修改7001为相应端口即可，到时候点击bat文件即可。

3、安装Ruby，默认安装即可

4、安装redis-xxx.gem，cmd执行`gem install redis` ，如果已经有了文件也可以直接`gem install --local C:\Ruby22-x64\redis-3.2.2.gem`

5、将`redis-trib.rb`移动到redis7001目录，点击各个bat文件，启动redis。再在`redis-trib.rb`的目录下用`cmd窗口`执行`ruby redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 `

出现以下结果即可
```
>>> Creating cluster
Connecting to node 127.0.0.1:7001: OK
Connecting to node 127.0.0.1:7002: OK
Connecting to node 127.0.0.1:7003: OK
Connecting to node 127.0.0.1:7004: OK
Connecting to node 127.0.0.1:7005: OK
Connecting to node 127.0.0.1:7006: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7001
127.0.0.1:7002
127.0.0.1:7003
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
Adding replica 127.0.0.1:7006 to 127.0.0.1:7003
M: 920ca3dca787db11534656a09e3698745eca93bc 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
M: 36bc2ce8371d1fea2d236d65f6061d3338d6cb2f 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
M: e8de5364d612a95fa7668950a46f6e8420e5fe15 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
S: 2400db5a46d462e6c8a33a2f71bef0ae788668be 127.0.0.1:7004
   replicates 920ca3dca787db11534656a09e3698745eca93bc
S: 7131b06ca88902d5930c12d511a3c68f447c0a71 127.0.0.1:7005
   replicates 36bc2ce8371d1fea2d236d65f6061d3338d6cb2f
S: 5a6ec6f760c16aea79ce1777dd1aae695216d4ec 127.0.0.1:7006
   replicates e8de5364d612a95fa7668950a46f6e8420e5fe15
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: 920ca3dca787db11534656a09e3698745eca93bc 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
M: 36bc2ce8371d1fea2d236d65f6061d3338d6cb2f 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
M: e8de5364d612a95fa7668950a46f6e8420e5fe15 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
M: 2400db5a46d462e6c8a33a2f71bef0ae788668be 127.0.0.1:7004
   slots: (0 slots) master
   replicates 920ca3dca787db11534656a09e3698745eca93bc
M: 7131b06ca88902d5930c12d511a3c68f447c0a71 127.0.0.1:7005
   slots: (0 slots) master
   replicates 36bc2ce8371d1fea2d236d65f6061d3338d6cb2f
M: 5a6ec6f760c16aea79ce1777dd1aae695216d4ec 127.0.0.1:7006
   slots: (0 slots) master
   replicates e8de5364d612a95fa7668950a46f6e8420e5fe15
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
中间需要输入`yes`

7、查看是否集群成功，每个节点都是互相连通的，只要登录其中一个即可。在redis7001目录的cmd窗口执行`redis-cli.exe -c -p 7001`，输入`cluster info`，出现以下结果
```
127.0.0.1:7001> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_sent:237
cluster_stats_messages_received:237`
```


#### cluster的命令使用
```
集群
cluster info ：打印集群的信息
cluster nodes ：列出集群当前已知的所有节点（ node），以及这些节点的相关信息。
节点
cluster meet <ip> <port> ：将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
cluster forget <node_id> ：从集群中移除 node_id 指定的节点。
cluster replicate <master_node_id> ：将当前从节点设置为 node_id 指定的master节点的slave节点。只能针对slave节点操作。
cluster saveconfig ：将节点的配置文件保存到硬盘里面。
槽(slot)
cluster addslots <slot> [slot ...] ：将一个或多个槽（ slot）指派（ assign）给当前节点。
cluster delslots <slot> [slot ...] ：移除一个或多个槽对当前节点的指派。
cluster flushslots ：移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
cluster setslot <slot> node <node_id> ：将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给
另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
cluster setslot <slot> migrating <node_id> ：将本节点的槽 slot 迁移到 node_id 指定的节点中。
cluster setslot <slot> importing <node_id> ：从 node_id 指定的节点中导入槽 slot 到本节点。
cluster setslot <slot> stable ：取消对槽 slot 的导入（ import）或者迁移（ migrate）。
键
cluster keyslot <key> ：计算键 key 应该被放置在哪个槽上。
cluster countkeysinslot <slot> ：返回槽 slot 目前包含的键值对数量。
cluster getkeysinslot <slot> <count> ：返回 count 个 slot 槽中的键 。
```
### 3、在Windows下单机部署Redis-Sentinel高可用模式

配置一Master二Slave三Sentinel的模式
```
master：127.0.0.1:6000 【初始化master】

slave：127.0.0.1:6001  127.0.0.1:6002

sentinel：127.0.0.1:26000  127.0.0.1:26001  127.0.0.1:26002
```

1、准备Windows下的Redis文件，这次采用一个软件多配置文件的模式。

2、指定Master，修改redis.conf文件
```
port 6000
```
Master为6000端口

3、指定Slave，复制redis.conf修改为redis6001.conf和redis6002.conf.
```
port 6001
# 指定master为本机的6000端口
slaveof 127.0.0.1 6000  
```
修改6001为6002端口即可

4、指定Sentinel，新建sentinel.conf、sentinel26001.conf和sentinel26002.conf

```
port 26000 // 当前Sentinel服务运行的端口
sentinel monitor mymaster 127.0.0.1 6000 2 
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 15000
```
同理修改sentinel26001.conf和sentinel26002.conf，只需修改端口即可

上述配置的解释
```
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
```
start cmd /k redis-server.exe redis.conf
start cmd /k redis-server.exe redis6001.conf
start cmd /k redis-server.exe redis6002.conf
```

startSentinel
```
start cmd /k redis-server.exe sentinel.conf --sentinel
start cmd /k redis-server.exe sentinel26001.conf --sentinel
start cmd /k redis-server.exe sentinel26002.conf --sentinel
```

6、点击启动查看状态

在`redis-cli.exe`目录中，`cmd`窗口输入`redis-cli.exe -h 127.0.0.1 -p 6000`，输入`info replication`
```
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
```
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

sentinel可以让redis实现主从复制，当一个集群中的master失效之后，sentinel可以选举出一个新的master用于自动接替master的工作，集群中的其他redis服务器自动指向新的master同步数据。一般建议sentinel采取奇数台，防止某一台sentinel无法连接到master导致误切换
