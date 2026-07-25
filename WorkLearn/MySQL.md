# MySQL知识点

## MYSQL多个字段拼接

### concat连接无NULL

```shell
select concat(1,2,3)
```

结果`123`

### concat连接带NULL

```shell
select concat(1,2,3,null)
```

结果`null`

### concat_ws连接无NULL

```shell
concat_ws(separator,str1,str2...)

separator表示分隔符
```

案例

```shell
select concat_ws(':',1,2,3)
```

结果`1:2:3`

### concat_ws连接带NULL

```shell
select concat_ws(':',1,2,3,null)
```

结果`1:2:3`

### group_concat

```shell
group_concat(distinct expression
    order by expression
    separator sep);
```

案例

```shell
SELECT 
    GROUP_CONCAT(DISTINCT name
        ORDER BY name ASC
        SEPARATOR ';')
FROM
    test;
```

结果`zhangsan;lisi;wangwu`

## delete、truncate、drop的区别

MySQL有三种删除数据的方式有delete、truncate、drop 。

### 执行速度

drop > truncate >> DELETE

### 区别

#### DELETE

- DELETE属于数据库DML操作语言，只删除数据不删除表的结构，会走事务，执行时会触发trigger。
- 在 InnoDB 中，DELETE其实并不会真的把数据删除，mysql 实际上只是给删除的数据打了个标记为已删除，因此 delete 删除表中的数据时，表文件在磁盘上所占空间不会变小，存储空间不会被释放，只是把删除的数据行设置为不可见。虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以重用这部分空间(覆盖)。
- DELETE执行时，会先将所删除数据缓存到rollback segement中，事务commit之后生效。
- delete from table_name删除表的全部数据,对于MyISAM会立刻释放磁盘空间，InnoDB 不会释放磁盘空间。
- 对于delete from table_name where xxx 带条件的删除, 不管是InnoDB还是MyISAM都不会释放磁盘空间。
- delete操作以后使用 optimize table table_name 会立刻释放磁盘空间。不管是InnoDB还是MyISAM 。所以要想达到释放磁盘空间的目的，delete以后执行optimize table操作。
- delete 操作是一行一行执行删除的，并且同时将该行的的删除操作日志记录在redo和undo表空间中以便进行回滚（rollback）和重做操作，生成的大量日志也会占用磁盘空间。

#### truncate

- truncate：属于数据库DDL定义语言，不走事务，原数据不放到 rollback segment中，操作不触发trigger。
- truncate table table_name 立刻释放磁盘空间 ，不管是 InnoDB和MyISAM。
- truncate能够快速清空一个表。并且重置auto_increment的值。

```text
对于MyISAM，truncate会重置auto_increment（自增序列）的值为1。而delete后表仍然保持auto_increment。

对于InnoDB，truncate会重置auto_increment的值为1。delete后表仍然保持auto_increment。但是在做delete整个表之后重启MySQL的话，则重启后的auto_increment会被置为1。
```

也就是说，InnoDB的表本身是无法持久保存auto_increment。delete表之后auto_increment仍然保存在内存，但是重启后就丢失了，只能从1开始。实质上重启后的auto_increment会从 SELECT 1+MAX(ai_col) FROM t 开始。

#### drop

- drop：属于数据库DDL定义语言，同Truncate。
- drop table table_name 立刻释放磁盘空间，不管是 InnoDB 和 MyISAM;drop语句将删除表的结构被依赖的约束(constrain)、触发器(trigger)、索引(index);依赖于该表的存储过程/函数将保留,但是变为 invalid 状态。

## MySQL查询过程中更新数据表

业务场景：有一张表的主键放在本表里，然后有几个字段的信息放在辅表里。现在需要根据主键的主键去辅表去查信息

```shell
# 主表字段 t_user : id,user_name,area_code,area_name
# 辅表字段 t_area_code : id,code,name

# 操作
update t_user tu,t_area_code tac 
set tu.area_name = tac.name
where tu.area_code = tac.code

```

## MySQL JDBC批量执行参数

MySQL Jdbc驱动在默认情况下会无视executeBatch()语句，把我们期望批量执行的一组sql语句拆散，一条一条地发给MySQL数据库，直接造成较低的性能。

只有把rewriteBatchedStatements参数置为true, 驱动才会帮你批量执行SQL (jdbc:mysql://ip:port/db?rewriteBatchedStatements=true)。

这个选项对INSERT/UPDATE/DELETE都有效，只不过对INSERT它为会预先重排一下SQL语句。

## MySQL utf8mb4

### 什么是utf8mb4

MySQL在5.5.3版本之后增加了这个utf8mb4的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。

### utf8与utf8mb4的联系

utf8mb4是utf8的超集(也就是说utf8mb4包含utf8)，理论上原来使用utf8，然后将字符集修改为utf8mb4，也不会对已有的utf8编码读取产生任何问题。当然，为了节省空间，一般情况下使用utf8也就够了。

### 为什么要用utf8mb4

低版本的MySQL支持的utf8编码，最大字符长度为 3 字节，如果遇到 4 字节的字符就会出现错误了。三个字节的 UTF-8 最大能编码的 Unicode 字符是 0xFFFF，也就是 Unicode 中的基本多文种平面（BMP）。也就是说，任何不在基本多文种平面的 Unicode字符，都无法使用MySQL原有的 utf8 字符集存储。
这些不在BMP中的字符包括哪些呢？最常见的就是Emoji 表情（Emoji 是一种特殊的 Unicode 编码，常见于 ios 和 android 手机上）和一些不常用的汉字以及任何新增的 Unicode 字符等等。

## MySQL导出每个表前几条数据

```cmd
mysqldump -u用户名 -p密码 -h IP地址 -P端口 --databases 数据库名 --where="true limit 100" > test.sql
```

## MySQL查询正在执行的事务

```shell
查询正在执行的事务:
SELECT * FROM information_schema.INNODB_TRX

杀掉线程:
kill 线程id(trx_mysql_thread_id)


查看正在锁的事务:
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS; 

查看等待锁的事务:
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```

## 创建索引时是否一定会锁表

### 什么是 DDL？

DDL（Data Definition Language，数据库定义语言）：用于定义和管理数据库的结构，它主要包括以下语句：

- CREATE：用于创建数据库、表、索引、视图等对象。
- ALTER：用于修改数据库、表、索引、视图等已存在的对象的结构。
- DROP：用于删除数据库、表、索引、视图等对象。
- TRUNCATE：用于删除表中的所有数据，但保留表的结构。
- RENAME：用于重命名数据库、表等对象。

### 什么是 DML？

DML (Data Manipulation Language，数据操作语言) ：用于查询和修改数据，它主要包括以下语句：

- INSERT：用于向表中插入新的数据行。
- UPDATE：用于更新表中已存在的数据行。
- DELETE：用于删除表中的数据行。
- SELECT：用于从表中检索数据。虽然 SELECT 主要用于查询，但某些包含数据修改的扩展 SQL 功能（如 LIMIT、ORDER BY、GROUP BY 等）也属于 DML 的范畴。

### 什么是 Online DDL？

Online DDL（Online Data Definition Language，在线数据定义语言）是指在数据库运行期间执行对表结构或其他数据库对象的更改操作，而不需要中断或阻塞其他正在进行的事务和查询。

在 MySQL 5.6 之前，创建索引时会锁表，但在 MySQL 5.6.7 之后，因为新增了 Online DDL 技术，所以此时在添加索引时，是可以和 DML 数据操作语言  INSERT、UPDATE、DELETE、SELECT 等命令一起执行的。

## MySQL5.7版本的安装和开启Binlog日志

### 安装

1、准备MySQL5.7，并解压到任一目录(个人目录:D:\Mysql\mysql-5.7.20-winx64)

2、配置文件

my.ini

```xml
[mysql]  
# 设置mysql客户端默认字符集  
default-character-set=utf8  
[mysqld]  
#设置3306端口  
port = 3306  
# 设置mysql的安装目录  
basedir=D:\Mysql\mysql-5.7.20-winx64
# 设置mysql数据库的数据的存放目录  
datadir=D:\Mysql\mysql-5.7.20-winx64\data  
# 允许最大连接数  
max_connections=200  
# 服务端使用的字符集默认为8比特编码的latin1字符集  
character-set-server=utf8  
# 创建新表时将使用的默认存储引擎  
default-storage-engine=INNODB  

#transaction_isolation=READ-COMMITTED
```

3、mysqld --initialize-insecure --user=mysql   初始化用户

4、mysqld -install  安装服务

5、net start mysql  启动MySQL

6、mysql -u root -p 按enter   无密码进入数据库

7、选择数据库 use mysql

8、重置密码

```shell
update user set authentication_string=password('123456') where user='root';
```

9.刷新

```shell
flush privileges;
```

10.退出quit;并重启服务。

### 开启Binlog

Binlog是一种二进制日志，主要是记录对数据发生或潜在发生更改的SQL语句，并以二进制的形式保存在磁盘中。可以帮助用户复制、恢复或审计。

#### 查看是否开启

```shell
show VARIABLES like 'log_bin'
```

#### 开启

1、net stop mysql  关闭服务

2、修改my.ini，添加如下配置

```xml
server-id=1
#设置日志格式
binlog_format=Row
#设置日志路径，注意路经需要mysql用户有权限写
log-bin=D:\Mysql\mysql-5.7.20-winx64\logs\mysql-bin
#设置binlog清理时间
expire_logs_days=7
#binlog每个日志文件大小
max_binlog_size=100m
#binlog缓存大小
binlog_cache_size=4m
#最大binlog缓存大小
max_binlog_cache_size=512m
```

3、net start mysql 开启服务

4、show VARIABLES like 'log_bin' 查看是否开启

## Explain详解

### 实例

```shell
explain select * from user；
```

执行上面的语句即可出现相关的属性信息`id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra`。

[官网](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_id)参考文档

|Column|SON Name|Meaning|
|--|--|--|
|id|select_id|The SELECT identifier|
|select_type|None|The SELECT type|
|table|table_name|The table for the output row|
|partitions|partitions|The matching partitions|
|type|access_type|The join type|
|possible_keys|possible_keys|The possible indexes to choose|
|key|key|The index actually chosen|
|key_len|key_length|The length of the chosen key|
|ref|ref|The columns compared to the index|
|rows|rows|Estimate of rows to be examined|
|filtered|filtered|Percentage of rows filtered by table condition|
|Extra|None|Additional information|

### 解释

#### id

select查询序列号，值为数字，表示查询中执行select子句或操作表的顺序。

- 1、id相同，可认为是一组，从上往下顺序执行。
- 2、id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行。
- 3、id相同不同同时存在。id如果相同，可以认为是一组，从上往下顺序执行，在所有组中，id值越大优先级越高，越先被执行。

#### select_type

查询的类型，主要是用于区别普通查询、联合查询、子查询等复杂查询

- 1、simple：简单查询。查询不包含子查询和union。
- 2、primary：查询中若包含任何复杂的子部分，最外层查询则被标记为primary。
- 3、subquery：在select或where中包含了子查询。
- 4、derived：在from中包含子查询。MySQL会递归执行这些子查询并将结果存放在一个临时表中，也称为派生表。
- 5、union：若第二个select出现在union之后，则被标记为union，若union包含在from子句的子查询中，外层select将被标记为derived。
- 6、union result：从 union 临时表检索结果的 select。

#### table

表示当前数据的所属的数据表

#### partitions

如果查询是基于分区表的话，会显示查询访问的分区，如果不是则返回null。

#### type

访问类型。从好到差为`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > all`。

- 1、system 表只有一行记录(等于系统表) 这是const类型的特例，平时不会出现，可以忽略不计。
- 2、const 表示通过索引一次就找到了，const 用于比较primary key或者unique索引。因为只匹配一行数据，所以很快如将主键置于where列表中，mysql就能将该查询转换为一个常量。
- 3、eq_ref 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常用于主键或者唯一索引扫描。
- 4、ref  非唯一性索引扫描，返回匹配某个单独值得所有行。
- 5、range 只检索给定范围的行，使用一个索引来选择行。
- 6、index  index 和all 的区别为index类型只遍历索引树。
- 7、all  将遍历全表以找到匹配的行

#### possible_keys

显示可能应用在这张表中的索引，一个或多个。显示涉及到的字段若存在索引，则该索引将被列出，但不一定被查询实际使用。

#### key

实际使用的索引。如果为null，则没有使用索引。

#### key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好，key_len 显示的值为索引字段的最大可能长度，并非实际使用长度。

#### ref

显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const、func、NULL、字段名。

#### rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。

#### filtered

百分比的值，rows * filtered/100 可以估算出将要和 explain 中前一个表进行连接的行数(前一个表指 explain 中的id值比当前表id值小的表)。

#### extra

不适合在其他列中显示但十分重要的额外信息。

- 1、using filesort  说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。此时MySQL会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行信息。MySQL中无法利用索引完成的排序操作称为"文件排序"。
- 2、using temporary MySQL需要创建一张临时表来处理查询。常见于排序order by和分组查询 group by。
- 3、using where 说明使用了where过滤
- 4、using index 这发生在对表的请求列都是同一索引的部分的时候，返回的列数据只使用了索引中的信息，而没有再去访问表中的行记录。是性能高的表现。
- 5、distinct 一旦MySQL找到了与行相联合匹配的行，就不再搜索了

## MySQL中使用JDBC流式查询

MySql中如果一次查询返回过多数据会发生OutOfMemoryError(OOM)错误，这时候可以调整JDBC的参数来规避这个错误。

```java
public void selectData(String sqlCmd,) throws SQLException {

    validate(sqlCmd);

    Connection conn = null;
    PreparedStatement stmt = null;
    ResultSet rs = null;   
    try {

        conn = petadataSource.getConnection();

        stmt = conn.prepareStatement(sqlCmd, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
        stmt.setFetchSize(Integer.MIN_VALUE);

        rs = stmt.executeQuery();       
        try {            
         while(rs.next()){                
          try {
                    System.out.println("one:" + rs.getString(1) + "two:" + rs.getString(2) +
                     "thrid:" + rs.getString(3));
                 } catch (SQLException e) {  
                     e.printStackTrace();
                 }
              }
         } catch (SQLException e) {
              e.printStackTrace();
         }
     } finally {
        close(stmt, rs, conn);
    }
}
```

之前完全不知道`prepareStatement`的方法里面还有两个参数可以写，然后自己去翻了API，发现有点东西。

> <https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html>

```java
PreparedStatement prepareStatement(String sql,
                                   int resultSetType,
                                   int resultSetConcurrency)
                            throws SQLException

Creates a PreparedStatement object that will generate ResultSet objects with the given type and concurrency. 
This method is the same as the prepareStatement method above, but it allows the default result set type and 
concurrency to be overridden. The holdability of the created result sets can be determined by calling getHoldability().

Parameters:
sql - a String object that is the SQL statement to be sent to the database; may contain one or more '?' IN parameters
resultSetType - a result set type; one of ResultSet.TYPE_FORWARD_ONLY, ResultSet.TYPE_SCROLL_INSENSITIVE, or ResultSet.TYPE_SCROLL_SENSITIVE
resultSetConcurrency - a concurrency type; one of ResultSet.CONCUR_READ_ONLY or ResultSet.CONCUR_UPDATABLE

Returns:
a new PreparedStatement object containing the pre-compiled SQL statement that will
 produce ResultSet objects with the given type and concurrency

Throws:
SQLException - if a database access error occurs, this method is called on a closed connection or
the given parameters are not ResultSet constants indicating type and concurrency
SQLFeatureNotSupportedException - if the JDBC driver does not support this method or
this method is not supported for the specified result set type and result set concurrency.

```

上述可知

- resultSetType -可选ResultSet.TYPE_FORWARD_ONLY, ResultSet.TYPE_SCROLL_INSENSITIVE,ResultSet.TYPE_SCROLL_SENSITIVE
- resultSetConcurrency -可选ResultSet.CONCUR_READ_ONLY，ResultSet.CONCUR_UPDATABLE。

其中可代表的意思是：

- `ResultSet.TYPE_FORWORD_ONLY` 结果集的游标只能向下滚动。
- `ResultSet.TYPE_SCROLL_INSENSITIVE` 结果集的游标可以上下移动，当数据库变化时，当前结果集不变。
- `ResultSet.TYPE_SCROLL_SENSITIVE` 返回可滚动的结果集，当数据库变化时，当前结果集同步改变。
- `ResultSet.CONCUR_READ_ONLY` 不能用结果集更新数据库中的表。
- `ResultSet.CONCUR_UPDATETABLE` 能用结果集更新数据库中的表。

上述的参数其实应用场景很多，我们在访问数据库的时候，在读取返回结果的时候，可能要前后移动指针，例如我们先计算有多少条信息，这是我们就需要把指针移到最后来计算，然后再把指针移到最前面，逐条读取，有时我们只需要逐条读取就可以了。还有就是有我们只需要读取数据，为了不破坏数据，我们可采用只读模式，有时我们需要望数据库里添加记录，这是我们就要采用可更新数据库的模式。不同场景设置不同的参数。对照上述解释即可理解原博客设置参数的意义。

## select......for update会锁表还是锁行

- 如果查询条件用了索引/主键，那么`select ..... for update`就会进行行锁。
- 如果是普通字段(没有索引/主键)，那么`select ..... for update`就会进行锁表。
