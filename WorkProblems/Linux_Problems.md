## 问题列表_Linux

### 1、Linux下运行Jar包项目

今天需要在Linux下运行一个小项目程序，很多东西都不太熟悉，查询修改终于完成了。
```
1、基于Maven的SpringBoot项目开发成功之后
2、使用mvn clean package  -Dmaven.test.skip=true，在项目下执行打成Jar包
3、将Jar包上传到服务器上，执行nohup java -jar ...jar > /dev/null 2>&1	& 开启程序
```

**注意**
```
nohup java -jar ...jar > /dev/null 2>&1 &

nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行
当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了输出文件。

/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失。
那么执行了>/dev/null之后，标准输出就会不再存在，没有任何地方能够找到输出的内容。

0 表示stdin标准输入
1 表示stdout标准输出
2 表示stderr标准错误
2>&1 的意思就是将标准错误重定向到标准输出。这里标准输出已经重定向到了 /dev/null。那么标准错误也会输出到/dev/null

最后一个&代表在后台运行
```

##### 其他小知识点
```
mkdir [文件夹]    创建文件夹

tail -f [文件名]  查看文件

netstat -anp | grep [端口号]  查看占用端口号

kill -9 [进程Id]    杀进程

rm -rf [文件夹/文件名]    删除文件

find / -name [文件名]   查找文件名

ps -a  查看所有进程

ps -ef | grep [进程名] 显示进程名
```

### 2、apt-get update和upgrade的区别

apt-get update就是访问服务器，更新可获取的软件及其版本信息，但仅仅给出一个可更新的list，具体更新需要通过apt-get upgrade，apt-get upgrade可将软件进行更新。update是更新软件列表，upgrade是更新软件。

### 3、测试网络端口
因为平时需要测试服务器是否跟其他服务器的端口连通，但是有些服务器没有装telnet，所以这个功能用不了，但是有个比较简单的方式
```
ssh -v -p [port] [username]@[ip]
```

prot选择服务器的端口

username可以随意

ip选择服务器的IP

如果显示`connection established`即为连通。

### 4、K8s查看Docker应用日志
因为我司使用K8s管理docker应用，有时候需要查看线上的日志，就需要专门的命令来查看。
```
# 查看所有namespace
kubectl get ns

# 查看所有pod所属的命名空间
kubectl get pod --all-namespaces

# 查看指定的命名空间的
kubectl get pod -n <namespace>

# 模糊搜索某个应用的<pod_name>
kubectl get pod --all-namespaces | awk '{ print $2 }' | grep user

# 查看某个命名空间的容器日志
kubectl logs <pod_name> -n <namespace> 
# 实时查看，等同于tail -f
kubectl logs -f <pod_name> -n <namespace> 

```

主要是获取`<pod_name>`、`<namespace> `,然后通过这两个参数即可查看日志。

### 5、实时监控Tomcat应用并重启

最近发现某一台服务器上面的Tomcat应用会莫名其妙崩掉，而且找不到原因，为了服务能正常，只能先用实时监控应用，如果应用崩溃就重启的笨办法。

在Linux的crontab中加入以下一条命令即可
```
* * * * *  root ps -ef | grep <应用名> | grep -v 'grep' || bash <应用地址>/apache-tomcat-9.0.36/bin/startup.sh
```

### 6、自定义命令

在linux服务器中部署了datax，但是datax的前缀命令很长且是固定，所以可以使用自定义命令简化一下
```
> vim /root/.bashrc
alias datax='python2 /data/soft/datax/bin/datax.py'
```

### 7、netstat用法

netstat命令的功能是显示网络连接、路由表和网络接口信息。

#### 语法
netstat [选项]
```
-a或--all：显示所有连线中的Socket； 
-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址； 
-c或--continuous：持续列出网络状态； 
-C或--cache：显示路由器配置的快取信息； 
-e或--extend：显示网络其他相关信息； 
-F或--fib：显示FIB； 
-g或--groups：显示多重广播功能群组组员名单； 
-h或--help：在线帮助； 
-i或--interfaces：显示网络界面信息表单； 
-l或--listening：显示监控中的服务器的Socket； 
-M或--masquerade：显示伪装的网络连线； 
-n或--numeric：直接使用ip地址，而不通过域名服务器； 
-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称； 
-o或--timers：显示计时器； 
-p或--programs：显示正在使用Socket的程序识别码和程序名称； 
-r或--route：显示Routing Table； 
-s或--statistice：显示网络工作信息统计表； 
-t或--tcp：显示TCP传输协议的连线状况； 
-u或--udp：显示UDP传输协议的连线状况； 
-v或--verbose：显示指令执行过程； 
-V或--version：显示版本信息； 
-w或--raw：显示RAW传输协议的连线状况； 
-x或--unix：此参数的效果和指定"-A unix"参数相同； 
--ip或--inet：此参数的效果和指定"-A inet"参数相同。
```

#### 实例
```
netstat -ntlp # 查看监听中的程序
```

### 8、Linux上批量修改文件的内容

修改当前目录下json文件中的内容替换
```
find . -name "*.json" | xargs perl -pi -e 's|ds=1|ds=0|g'
```

### 9、ll命令根据时间排序

#### 时间倒叙
```
ll -t
```

#### 时间正序
```
ll -t | tac
```

#### 取前几个
```
ll -t | head -n 5
```

### 10、shell脚本中特殊变量
- $0	当前脚本的文件名
- $n	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
- $#	传递给脚本或函数的参数个数。
- $*	传递给脚本或函数的所有参数。
- $@	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。
- $?	上个命令的退出状态，或函数的返回值。
- $$	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。

### 11、Linux文件夹的意义
- bin 是binary的缩写，主要存放一些常用的命令，比如ls、cp、mv之类的
- boot 主要存放一些linux启动时需要用到的核心文件
- dev 是device的缩写，主要存放一些linux的设备文件
- etc 主要存放系统用户所需要的配置文件和子目录
- home 主要存放用户目录
- lib 是library的缩写，主要存放一些动态库，供应用程序调用
- lost+found 一般是空的，当系统非法关机后，相关文件会存放在此目录
- media 自动挂载一些linux系统自动识别的设备，比如U盘，光驱等
- mnt 提供给用户的用于挂载临时别的文件系统（手动挂载），比如另外的硬盘等
- opt 提供给主机额外安装软件所需要的目录
- proc 这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息
- root 超级用户的主目录
- sbin s是super user的简称，此目录主要存放一些系统管理员所用到的系统管理程序
- srv 主要存放一些系统服务启动之后所要用到的数据
- run 主要存放一些系统运行时需要用到的数据
- usr 主要存放一些用户的应用程序及文件，类似于windows下的program files
- tmp 存放一些临时文件
- var 主要存放一些经常被修改的文件，比如日志文件、电子邮件等
