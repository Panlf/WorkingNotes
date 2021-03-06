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
