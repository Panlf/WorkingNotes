## 问题列表_2

### 1、使用IDEA出现Lombok的编译错误

我已经在IDEA中安装了Lombok插件，在编写代码中也能够成功导入，但是运行代码会报编译错误，然后需要在IDEA中设置才能使用。
```
file - >  Setting - Compiler - Annotation Processors - Enable annotation processing 
选中即可，运行成功。
```

### 2、将已有的项目上传到Github上
```
1、在github上新建的Repository -- 跟需上传项目的项目名称一致
2、在需上传项目的根目录在打开git，并进行以下命令
3、git init
4、git add .
5、git remote add origin [Repository地址]
6、git commit -m "提交说明"
7、git pull --rebase origin master
8、git push origin master
```

### 3、Eclipse中不需要源码包也可查看源码

今天我使用Eclipse时候，想要查看某个类的源码，但是提示我需要导入源码，这样非常麻烦，就需要一个反编译插件。

Eclipse的反编译插件可以使用 - Eclipse Class Decompiler

Eclipse操作
```
1、Help - > Eclipse Marketplace 
2、输入Decompiler，选择Eclipse Class Decompiler安装
3、安装完成重启Eclipse
4、选择Window -> Preferences -> Gerenal -> Editors -> File Associations 
5、选择File types的*.class without source 把Class Decompiler Viewer设置为默认的editors
6、测试使用
```

### 4、Linux下运行Jar包项目

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

### 5、Git放弃本地修改

本地增加、修改、删除了一些文件，现在想远程代码库覆盖本地，使用了`git check -- .`操作，但是只覆盖了修改的部分，新增和删除的部分没有覆盖。
所以需要使用如下操作：
```
git checkout . && git clean -df
```

 `git checkout . `是放弃本地修改，没有提交的可以回到未修改前版本；`git clean -df`删除当前目录下没有被track过的文件和文件夹。
