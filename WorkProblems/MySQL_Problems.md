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

### 6、delete、truncate、drop的区别
MySQL有三种删除数据的方式有delete、truncate、drop 。

#### 执行速度
drop > truncate >> DELETE

#### 区别
##### DELETE

- DELETE属于数据库DML操作语言，只删除数据不删除表的结构，会走事务，执行时会触发trigger。
- 在 InnoDB 中，DELETE其实并不会真的把数据删除，mysql 实际上只是给删除的数据打了个标记为已删除，因此 delete 删除表中的数据时，表文件在磁盘上所占空间不会变小，存储空间不会被释放，只是把删除的数据行设置为不可见。虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以重用这部分空间(覆盖)。
- DELETE执行时，会先将所删除数据缓存到rollback segement中，事务commit之后生效。
- delete from table_name删除表的全部数据,对于MyISAM会立刻释放磁盘空间，InnoDB 不会释放磁盘空间。
- 对于delete from table_name where xxx 带条件的删除, 不管是InnoDB还是MyISAM都不会释放磁盘空间。
- delete操作以后使用 optimize table table_name 会立刻释放磁盘空间。不管是InnoDB还是MyISAM 。所以要想达到释放磁盘空间的目的，delete以后执行optimize table操作。
- delete 操作是一行一行执行删除的，并且同时将该行的的删除操作日志记录在redo和undo表空间中以便进行回滚（rollback）和重做操作，生成的大量日志也会占用磁盘空间。

##### truncate

- truncate：属于数据库DDL定义语言，不走事务，原数据不放到 rollback segment中，操作不触发trigger。
- truncate table table_name 立刻释放磁盘空间 ，不管是 InnoDB和MyISAM。
- truncate能够快速清空一个表。并且重置auto_increment的值。

```
对于MyISAM，truncate会重置auto_increment（自增序列）的值为1。而delete后表仍然保持auto_increment。

对于InnoDB，truncate会重置auto_increment的值为1。delete后表仍然保持auto_increment。但是在做delete整个表之后重启MySQL的话，则重启后的auto_increment会被置为1。
```
也就是说，InnoDB的表本身是无法持久保存auto_increment。delete表之后auto_increment仍然保存在内存，但是重启后就丢失了，只能从1开始。实质上重启后的auto_increment会从 SELECT 1+MAX(ai_col) FROM t 开始。

##### drop
- drop：属于数据库DDL定义语言，同Truncate。
- drop table table_name 立刻释放磁盘空间，不管是 InnoDB 和 MyISAM;drop语句将删除表的结构被依赖的约束(constrain)、触发器(trigger)、索引(index);依赖于该表的存储过程/函数将保留,但是变为 invalid 状态。

### 7、MySQL查询过程中更新数据表

业务场景：有一张表的主键放在本表里，然后有几个字段的信息放在辅表里。现在需要根据主键的主键去辅表去查信息
```
# 主表字段 t_user : id,user_name,area_code,area_name
# 辅表字段 t_area_code : id,code,name

# 操作
update t_user tu,t_area_code tac 
set tu.area_name = tac.name
where tu.area_code = tac.code

```

### 8、MySQL JDBC批量执行参数
MySQL Jdbc驱动在默认情况下会无视executeBatch()语句，把我们期望批量执行的一组sql语句拆散，一条一条地发给MySQL数据库，直接造成较低的性能。

只有把rewriteBatchedStatements参数置为true, 驱动才会帮你批量执行SQL (jdbc:mysql://ip:port/db?rewriteBatchedStatements=true)。

这个选项对INSERT/UPDATE/DELETE都有效，只不过对INSERT它为会预先重排一下SQL语句。

### 9、MySQL utf8mb4
#### 什么是utf8mb4
MySQL在5.5.3版本之后增加了这个utf8mb4的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。

#### utf8与utf8mb4的联系
utf8mb4是utf8的超集(也就是说utf8mb4包含utf8)，理论上原来使用utf8，然后将字符集修改为utf8mb4，也不会对已有的utf8编码读取产生任何问题。当然，为了节省空间，一般情况下使用utf8也就够了。

#### 为什么要用utf8mb4
低版本的MySQL支持的utf8编码，最大字符长度为 3 字节，如果遇到 4 字节的字符就会出现错误了。三个字节的 UTF-8 最大能编码的 Unicode 字符是 0xFFFF，也就是 Unicode 中的基本多文种平面（BMP）。也就是说，任何不在基本多文种平面的 Unicode字符，都无法使用MySQL原有的 utf8 字符集存储。
这些不在BMP中的字符包括哪些呢？最常见的就是Emoji 表情（Emoji 是一种特殊的 Unicode 编码，常见于 ios 和 android 手机上）和一些不常用的汉字以及任何新增的 Unicode 字符等等。
