## 问题列表_1

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

### 2、MYSQL查询正在执行的事务
```
查询正在执行的事务:
SELECT * FROM information_schema.INNODB_TRX

杀掉线程:
kill 线程id(trx_mysql_thread_id)


查看正在锁的事务:
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS; 

查看等待锁的事务:
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```

### 3、MYSQL删除数据时发生错误

今天使用`delete from xxx where xxx`时候发生如下错误`The total number of locks exceeds the lock table size`，主要是因为数据表里数据太大了，缓存不够了，发生了如此错误。只要调大`innodb_buffer_pool_size`参数即可，该参数的默认值是8M。

命令：
```
# 3G  3*1024*1024*1024
SET GLOBAL innodb_buffer_pool_size=3221225472;
```

