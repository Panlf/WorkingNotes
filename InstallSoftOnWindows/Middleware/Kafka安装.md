# Kafka安装

1、安装配置`Zookeeper`，下载[Zookeeper](https://zookeeper.apache.org/releases.html)，修改`zoo.cfg`中的`dataDir`地址，也可修改端口。

2、点击`zkServer.cmd`，启动`Zookeeper`

2、下载[Kafka](http://kafka.apache.org/downloads)，解压文件进入`config`文件夹，修改`server.properties`中的日志地址

```properties
log.dirs=D:/kafka/kafka_2.11-2.2.0/kafka-logs
```

3、修改zookeeper.properties

```properties
dataDir=D:/kafka/kafka_2.11-2.2.0/data
clientPort=127.0.0.1:2181
```

4、在kafka目录(D:\kafka\kafka_2.11-2.2.0)中写启动文件

startkafka.bat

```cmd
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

点击上述bat文件即可，前提是Zookeeper已经启动
