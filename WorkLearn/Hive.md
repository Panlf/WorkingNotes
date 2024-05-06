# Hive的知识点

## 分区表的操作
```
# 查看某表的分区
show partitions [库名].[表名];

# 删除某一个分区
alter table [库名].[表名] drop partition ([分区字段]=[分区值]);

# 删除全部分区
drop table  [库名].[表名];
```

## 时间操作

### 获取当月的天数
```
select dayofmonth(current_date);
```

### 时间转换
```
select from_unixtime(unix_timestamp('20230328','yyyyMMdd'),'yyyyMM');
```

## row_number()排序函数 
统计每个部门薪资最高的员工信息（同一个部门的员工按照薪资进行降序排序）
``` 
//如果不写asc/desc的话，默认为asc 
第一种写法：row_number() over(partition by 一个或多个分组列 order by 一个或多个排序列 asc/desc) as 别名 
 
第二种写法：row_number() over(distribute by 一个或多个分组列 sort by 一个或多个排序列 asc/desc) as 别名
```

在使用`row_number() over()`函数时候，`over()`里头的分组以及排序的执行晚于 `where` 、`group by`、  `order by`的执行。

```
select *,
row_number() over(distribute by deptid sort by salary desc) rn from employee;

//1.distribute by deptid sort by salary desc：按照部门deptid进行分组，每个分组内按照薪资即salary进行降序排序，即同一个部门的员工按照薪资进行降序排序

//2.分组条件：distribute by deptid        排序条件：sort by salary desc

//3.rn：为别名，代表每个分组中每行数据的所在序号ID，可用于根据rn序号ID直接获取出每个分组中的第一条数据，作用大。
```

## Hive使用count计算数据量为0
我是用阿里巴巴的数据传输工具`DataX`导入到Hive的，没有正常的操作录入的，所以Hive的元数据保存的状态的信息还是0，这样我`count`出来的数据量就是0。

我们可以选择让Hive的元数据进行修复
```
hive -e 'msck repair table [database].[tablename]'
```

也能使用修改参数方式
```
set hive.compute.query.using.stats=false;
```
这样查询语句就不会走元数据信息，会使用`mapreduce`查询数据。

使用Jdbc可以用以下方式
```java
public static long countData(Connection conn,String sql){
    Statement st = null;
    ResultSet res = null;
    long count = 0l;
    try{
        st = conn.createStatement();
        st.execute("set hive.compute.query.using.stats=false");

        res = st.executeQuery(sql);

        while (res.next()){
            count = res.getLong(1);
        }
        st.execute("set hive.compute.query.using.stats=true");

    }catch (Exception e){
        e.printStackTrace();
    }finally {
        HiveUtils.close(conn,st,null,res);
    }
    return count;
}
```

还可以使用以下投机的方式
```
select count(表中的字段) from 表名
```

## 自定义函数
```
# 将自定义jar包放到服务器上
/data/temp_data/low-str-1.0.0.jar
# 使用hdfs放到hadoop上
hdfs dfs -put /data/temp_data/low-str-1.0.0.jar /hivejar/hiveudf
# 添加到hive
add jar hdfs://172.12.3.9:8020/hivejar/hiveudf/low-str-1.0.0.jar;
# 临时函数 默认是default数据库
create temporary function low_str as 'com.hive.tutorial.udf.LowStr'; 
# 永久函数
create function sys.low_str as 'com.hive.tutorial.udf.LowStr' using jar 'hdfs://172.12.3.9:8020/hivejar/hiveudf/low-str-1.0.0.jar';
```
