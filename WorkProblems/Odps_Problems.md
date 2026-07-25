# 问题列表_Odps

## 1、Odps小文件过多导致查询缓慢

Odps里面有一张表一直用的是insert into插入数据，差不多一年左右，这张表查询起来就非常慢，一查原因就是因为小文件太多了，导致查询速度降低，所以需要重新调整表，减少小文件

合并方式

```shell
set odps.stage.reducer.num = 10;
alter table aaa merge smallfiles；
```

使用覆盖方式

```shell
INSERT OVERWRITE TABLE aaa
SELECT * FROM aaa;
```
