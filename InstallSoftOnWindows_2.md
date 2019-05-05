## 在Windows下安装软件_2

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

2、下载[Kafka](http://kafka.apache.org/downloads)，解压文件进入`config`文件夹，修改`erver.properties`中的日志地址
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
