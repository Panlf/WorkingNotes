# 问题列表_MySQL

## 1、MYSQL删除数据时发生错误

今天使用`delete from xxx where xxx`时候发生如下错误`The total number of locks exceeds the lock table size`，主要是因为数据表里数据太大了，缓存不够了，发生了如此错误。只要调大`innodb_buffer_pool_size`参数即可，该参数的默认值是8M。

命令：

```shell
# 3G  3*1024*1024*1024
SET GLOBAL innodb_buffer_pool_size=3221225472;
```

## 2、MYSQL出现连接错误次数过多

当连接错误次数过多时，mysql会禁止客户机连接。我们选择两种方式

- `mysqladmin  -u  root  -p  flush-hosts`清除缓存
- 修改mysql配置文件，在`[mysqld]`下面添加`max_connect_errors=1000`，然后重启MYSQL

## 3、MySQL解决字段调整自增问题

```shell
-- 1. 给所有重复为0的记录分配唯一ID
SET @seq = 0;
UPDATE `t_fs` SET `_F_ID_` = @seq:=@seq+1;

-- 2. 修改字段为主键+自增，移除默认值
ALTER TABLE `t_fs` 
MODIFY COLUMN `_F_ID_` bigint(20) unsigned NOT NULL PRIMARY KEY AUTO_INCREMENT;
```

## 4、tinyint(1)用java转化为int的坑

今天工作中有个需求:将数据库`tinyint`转换为`int`类型,在转换过程中发现该数字被转换为`Boolean`类型了

原因:在`MYSQL`官方的`JDBC文档`定义转换规则为:如果`tinyInt1isBit=true`(默认)，且`tinyInt`存储长度为1 ，则转为`java.lang.Boolean` 。

否则转为`java.lang.Integer`。

解决办法:在URL后面加上`:?tinyInt1isBit=false`