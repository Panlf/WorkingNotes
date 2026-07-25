# Zookeeper的伪集群模式

## 配置

1、在[Zookeeper官网](http://zookeeper.apache.org/releases.html)下载Windows版本

2、在对应的位置解压缩，我是在D:\zookeeper\Cluster解压出来三个Zookeeper

3、分别取名zookeeper-3.5.2-alpha-0、zookeeper-3.5.2-alpha-1、zookeeper-3.5.2-alpha-3

4、修改conf/zoo.cfg

添加数据和日志的文件夹

```xml
dataDir=D:/zookeeper/Cluster/zookeeper-3.5.2-alpha-0/data
dataLogDir=D:/zookeeper/Cluster/zookeeper-3.5.2-alpha-0/logs
```

修改端口号

```xml
clientPort=2182
```

集群配置

```xml
server.0=127.0.0.1:2888:3888
server.1=127.0.0.1:2889:3889
server.2=127.0.0.1:2890:3890
```

上述解释

```text
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

```cmd
start cmd /k D:\zookeeper\Cluster\zookeeper-3.5.2-alpha-0\bin\zkServer.cmd 
start cmd /k D:\zookeeper\Cluster\zookeeper-3.5.2-alpha-1\bin\zkServer.cmd 
start cmd /k D:\zookeeper\Cluster\zookeeper-3.5.2-alpha-2\bin\zkServer.cmd 
```

## zoo.cfg的配置解释

```text
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
