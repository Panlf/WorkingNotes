# Linux的知识点

## 查询应用的端口
### lsof

语法格式
```
lsof -i:端口
```

显示信息
```
COMMAND: 进程的名称
PID: 进程标识符
USER: 进程所有者
FD: 文件描述符，应用程序通过文件描述符识别该文件。
TYPE: 文件类型
DEVICE: 指定磁盘的名称
SIZE: 文件的大小
NODE: 索引节点（文件在磁盘上的标识）
NAME: 打开文件的确切名称
```

### netstat

语法格式
```
netstat -tunlp | grep 端口号
```

```
-t 显示tcp连接信息
-u 显示udp相关信息
-n 使用数字形式的IP、端口号、用户ID替代主机、协议、用户等名称信息
-l 仅显示正在监听的sockets接口信息
-p 显示进程名称及对应进程ID
```
