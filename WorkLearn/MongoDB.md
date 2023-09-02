# MongoDB知识点
## 数据备份、恢复和导入、导出
### 备份

```
mongodump -h dbhost -d dbname -o dbdirectory
```

- -h：MongoDB所在服务器地址
- -d：需要备份的数据库名称
- -o：备份的数据存放位置(该目录需要提前建立)

**案例**

```
mongodump -h 127.0.0.1:27017 -d test -o E:\temp
```
上述操作即可在`E:\temp`中备份`test库`，不过备份的文件格式是`BJON`

### 恢复
```
mongorestore -h dbhost -d dbname dbdirectory
```
- -h：MongoDB所在服务器地址
- -d：需要备份的数据库名称
- dbdirectory：备份的数据所在位置

**案例**

```
mongorestore -h 127.0.0.1:27017 -d test E:\temp\test
```

### 导出
```
mongoexport -h dbhost -d dbname -c collectionName -o output
```

- -h：MongoDB所在服务器地址
- -d：需要导出的数据库
- -c：需要导出的集合名称
- -o：确定导出的文件名

**案例**

```
mongoexport -h 127.0.0.1:27017 -d test -c person -o E:\text.txt
```

### 导入
```
mongoimport -h dbhost -d dbname -c collectionname output 
```


- -h：MongoDB所在服务器地址
- -d：需要导入的数据库
- -c：需要导入的集合名称

**案例**

```
mongoimport -h 127.0.0.1:27017 -d test -c person E:\text.txt 
```
