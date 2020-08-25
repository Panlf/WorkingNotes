## 问题列表_MySQL

### 1、MYSQL查询正在执行的事务
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

### 2、MYSQL删除数据时发生错误

今天使用`delete from xxx where xxx`时候发生如下错误`The total number of locks exceeds the lock table size`，主要是因为数据表里数据太大了，缓存不够了，发生了如此错误。只要调大`innodb_buffer_pool_size`参数即可，该参数的默认值是8M。

命令：
```
# 3G  3*1024*1024*1024
SET GLOBAL innodb_buffer_pool_size=3221225472;
```

### 3、MYSQL导出每个表前几条数据
```
mysqldump -u用户名 -p密码 -h IP地址 -P端口 --databases 数据库名 --where="true limit 100" > test.sql
```

### 4、MYSQL出现连接错误次数过多
当连接错误次数过多时，mysql会禁止客户机连接。我们选择两种方式

- `mysqladmin  -u  root  -p  flush-hosts `清除缓存
- 修改mysql配置文件，在`[mysqld]`下面添加`max_connect_errors=1000`，然后重启MYSQL

### 5、MYSQL多个字段拼接
#### concat连接无NULL
```
select concat(1,2,3)
```
结果`123`

#### concat连接带NULL
```
select concat(1,2,3,null)
```
结果`null`

#### concat_ws连接无NULL
```
concat_ws(separator,str1,str2...)

separator表示分隔符
```

案例
```
select concat_ws(':',1,2,3)
```
结果`1:2:3`

#### concat_ws连接带NULL
```
select concat_ws(':',1,2,3,null)
```
结果`1:2:3`

#### group_concat
```
group_concat(distinct expression
    order by expression
    separator sep);
```
案例	
```
SELECT 
    GROUP_CONCAT(DISTINCT name
        ORDER BY name ASC
        SEPARATOR ';')
FROM
    test;
```

结果`zhangsan;lisi;wangwu`