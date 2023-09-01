# Kafka知识点

## 基础命令行命令
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