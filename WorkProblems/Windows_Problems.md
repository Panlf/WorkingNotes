## 问题列表_Windows

### 1、Windows查找端口是否被占用

今天使用`Hexo`的时候，本地`Hexo s`，然后访问`http:\\localhost:4000`，发现不能访问，猜想4000端口被占用。

Widows下查找占用端口：
```
//查找占用端口4000
netstat -ano | findstr "4000"  

//返回
TCP   127.0.0.1:4000  0.0.0.0:0    LISTENING       2084
```

查看进程：
```
tasklist | findstr "2084" 

//返回
FoxitProtect.exe   2084 Services    0      6,412 K
```

杀掉进程：
```
taskkill /im /t /f FoxitProtect.exe 

//也可以使用pid杀
taskkill /pid pid /t /f
```

### 2、Windows下更改npm全局模块和cache默认安装位置

1、在指定位置新建node_global、node_cache文件夹

2、命令指定位置
```
npm config set prefix "XXX\node_global"
npm config set cache "XXXs\node_cache"
```

3、配置环境变量
```
NODE_PATH = XXX\Node\nodejs
PATH = %NODE_PATH%\;%NODE_PATH%\node_modules;%NODE_PATH%\node_global;
```

### 3、Windows批处理

Maven有时候下载的包不全的，需要删除重下。但是很多包分布在不同的地方，一个个找太麻烦，就采用Windows的批处理方式
```
@echo off
rem create by Panlf
  
rem 这里写你的仓库路径
set REPOSITORY_PATH=C:\Users\Panlf\.m2\repository
rem 正在搜索...
for /f "delims=" %%i in ('dir /b /s "%REPOSITORY_PATH%\*lastUpdated*"') do (
    del /s /q %%i
)
rem 搜索完毕
pause
```

下面是删除整个目录
```
@echo off
setlocal enabledelayedexpansion
::set /p input_path=请输入要要操作的文件夹:
::echo 获取到的地址:%input_path%
::*.lastUpdated
for /r C:\Users\Panlf\.m2\repository %%i in (*.lastUpdated) do (
	set pan_name=%%~di
	set del_name=%%~pi
	set true_name=!pan_name!!del_name!
	echo !true_name!
	rd /s /q !true_name!
)
pause
```
有时候很多批处理会看不过来，就聚合一下
```
set /p a=enter num(1 open eclipse 2 start mysql 3 start mongodb 4 start ActiveMQ 5 start Zookeeper 6 start Redis):
if %a%==1  start D:\Eclipse\java\eclipse-jee-neon-2-win32-x86_64
if %a%==2  net start mysql
if %a%==3  net start mongodb
if %a%==4  call D:\ActiveMQ\apache-activemq-5.15.2\bin\win64\activemq.bat
if %a%==5  call D:\zookeeper\zookeeper-3.5.2-alpha\bin\zkServer.cmd
if %a%==6  call D:\Redis\stand\redis-server.exe
::pause
else exit
```

