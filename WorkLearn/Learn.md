## PostgreSQL

### 添加自增序列

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

## Kafka

### 基础命令行命令
```shell
# 查看topic
bin/kafka-topics.sh --bootstrap-server [ip]:9092 --list

# 创建topic
bin/kafka-topics.sh --bootstrap-server [ip]:9092 --create --partitions 1 --replication-factor 1 --topic [topic名]

# 消费数据
bin/kafka-console-consumer.sh --bootstrap-server [ip]:9092 --from-beginning --topic [topic名]

# 各个分区的最大offset
kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list [ip]:9092 --topic test  --time -1

# 各个分区的最小offset
kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list [ip]:9092 --topic test  --time -2
```

 
## Hive

### Hive分区表的操作
```
# 查看某表的分区
show partitions [库名].[表名];

# 删除某一个分区
alter table [库名].[表名] drop partition ([分区字段]=[分区值]);

# 删除全部分区
drop table  [库名].[表名];
```
