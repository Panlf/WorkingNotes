# PostgreSQL

## 查询某表的属性
```sql
SELECT
c.relname 表名称,
    A.attname AS 字段名称,
    col_description(A.attrelid, A.attnum) AS 注释,
    format_type(A.atttypid, A.atttypmod) AS 类型,
    CASE WHEN A.attnotnull = 'f'
THEN '否'
ELSE '是'
END AS 是否必填,
a.attnum 序号
FROM
pg_class AS c,
pg_attribute AS a
WHERE
A.attrelid = C.oid
AND A.attnum > 0
ORDER BY c.relname, a.attnum;
```

## 修改字段

```sql
alter table test ALTER COLUMN update_time_ set DEFAULT CURRENT_TIMESTAMP;  
alter table test ALTER COLUMN tsms_ set DEFAULT floor(date_part('epoch'::text, CURRENT_TIMESTAMP));  
alter table test ALTER COLUMN op_ set DEFAULT 'C'::character varying;  
```

## 添加自增序列

```sql
# 创建序列
CREATE SEQUENCE [[模式].[表名]_id_seq](自定义) START 1;

# 序列赋值
ALTER TABLE [表名] ALTER COLUMN [自增字段] SET DEFAULT nextval('序列名');

# 查询序列值
SELECT nextval('序列名');

# 序列重置
ALTER sequence [序列名] restart with 1
```

## 死锁和解锁
```
查询是否锁表了
    select oid from pg_class where relname=tablename
    select pid from pg_locks where relation=oid
如果查询到了结果，表示该表被锁 则需要释放锁定
    select pg_cancel_backend(oid)
```

通过上述的方法顺利解决我的锁表问题。不过通过百度，还看到别人有另一种的解决方式。
```
检索出死锁进程的ID
    select * from pg_stat_activity where wait_event_type = 'Lock';
如果查询到了pid，表示有死锁进程，则需要杀掉解锁进程
    select pg_terminate_backend('pid')
```
- *9.6 版本之后 `pg_stat_activity`视图的 `waiting`字段被 `wait_event_type`和 `wait_event`字段取代*
- *如果 `pg_stat_activity` 没有记录，可以查询 `pg_locks` 这个表中是否有锁定的记录。*
- *`pg_terminate_backend()` 也可以用 `pg_cancle_backend()` 代替。*