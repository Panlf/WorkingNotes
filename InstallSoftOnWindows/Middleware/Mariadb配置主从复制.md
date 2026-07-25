# Mariadb配置主从复制

## 环境及版本

- Mariadb 10.4.6 下载可访问[官网](https://mariadb.org)
- Windows7 64位

## 安装主数据库

### 主数据库配置文件

```xml
[client]
port = 3310
socket = D:/Mariadb/mariadb_master/tmp/mysql.sock


[mysqld]
port = 3310
socket = D:/Mariadb/mariadb_master/tmp/mysql.sock
datadir = D:/Mariadb/mariadb_master/data
server-id=1
key_buffer_size = 384M
max_allowed_packet= 1M
table_open_cache=512
sort_buffer_size=64k
read_buffer_size = 1M
read_rnd_buffer_size = 512K
thread_cache_size = 8
query_cache_size = 16M
log-bin=mysql-bin
lower_case_table_names=1
```

### 主数据库安装服务

```cmd
mysqld.exe --install Mmariadb
```

### 主数据库初始化

```cmd
mysql_install_db
```

### 启动主数据库

```cmd
net start Mmariadb
```

### 设置主数据库ROOT密码

```text
1、mysql -uroot 
2、use mysql;
3、SET password for 'root'@'localhost'=password('root'); 
4、exit; 
5、重启即可 - net stop Mmariadb |  net start Mmariadb
```

### 设置主数据库备份账号

```text
1、登录主数据库
2、grant replication slave on *.* to 'slaver'@'%' identified by '123456';
3、flush privileges;
4、重启
```

### 查看主数据库状态

```text
1、登录
2、show master status\G
3、以下结果为正常
*************************** 1. row ***************************
            File: mysql-bin.000003
        Position: 342
    Binlog_Do_DB:
Binlog_Ignore_DB:
```

## 安装从数据库

### 从数据库配置文件

主要修改端口、server-id、datadir及socket等相关参数

```xml
[client]
port = 3311
socket = D:/Mariadb/mariadb_slave/tmp/mysql.sock


[mysqld]
port = 3311
socket = D:/Mariadb/mariadb_slave/tmp/mysql.sock
datadir = D:/Mariadb/mariadb_slave/data
server-id=2
key_buffer_size = 384M
max_allowed_packet= 1M
table_open_cache=512
sort_buffer_size=64k
read_buffer_size = 1M
read_rnd_buffer_size = 512K
thread_cache_size = 8
query_cache_size = 16M
log-bin=mysql-bin
lower_case_table_names=1
```

### 从数据库安装服务

```cmd
mysqld.exe --install Smariadb
```

### 从数据库初始化

```cmd
mysql_install_db
```

### 启动从数据库

```cmd
net start Smariadb
```

### 设置从数据库ROOT密码

```text
1、mysql -uroot 
2、use mysql;
3、SET password for 'root'@'localhost'=password('root'); 
4、exit; 
5、重启即可 - net stop Smariadb |  net start Smariadb
```

## 主从配置

```text
1、登录从数据库
2、stop slave；
3、change master to master_host='127.0.0.1',master_user='slaver',master_port=3310,master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=342;
4、start slave
5、show slave status\G
6、结果
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 127.0.0.1
                   Master_User: slaver
                   Master_Port: 3310
                 Connect_Retry: 60
               Master_Log_File: mysql-bin.000003
           Read_Master_Log_Pos: 610
                Relay_Log_File: Panlf-PC-relay-bin.000002
                 Relay_Log_Pos: 823
         Relay_Master_Log_File: mysql-bin.000003
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table:
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 610
               Relay_Log_Space: 1135
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: 0
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 1
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: conservative
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for the sl
ave I/O thread to update it
              Slave_DDL_Groups: 1
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 0



Slave_IO_Running: Yes
Slave_SQL_Running: Yes
这两个值都是YES即表示主从配置已经成功
```

参数解释

```text
 1)master_host是指主服务器的IP

 2)master_user是指使用哪个用户登录主服务器

 3)master_password是指登录密码

 4)master_log_file是指在第4个步骤中的File名称

 5)master_log_pos是指在第4个步骤中的Position

 6)master_port是指主服务器的端口，默认是3306，如果不是3306则需要自己指定
```

## 注意问题

```text
如果Slave_SQL_Running:No

1、程序可能在slaves上进行了写操作
2、也可能是slave机器重启后，事务回滚造成的

解决方案一
一般是事务回滚造成的
stop slave
。。。。
start slave

解决方案二
首先停掉Slave服务 slave stop
到主服务器查看主机状态
记录file和positiond对应的值

然后到slave服务器上执行手动同步
change 。。。。
start slave
```
