## Windows下IT软件安装_2

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
