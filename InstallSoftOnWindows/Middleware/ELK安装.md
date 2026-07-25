# ELK安装

ELK即Elasticsearch、Logstash、 Kibana，在windows下安装比较简单，下载解压即可，不过使用需要具有Java环境，目前官网是7版本需要1.8以上Jdk。

1、[官网下载](https://www.elastic.co/cn/downloads/)

2、下载Elasticsearch并解压，启动`\Elasticesarch\bin\elasticsearch.bat`，然后访问`localhost:9200`，观察启动成功

3、下载Logstash，并在`\logstash\bin`新建`logstash.conf`。这里我是监控Nginx日志，所以以下配置文件是关于监控Nginx日志

```xml
input {  
 file {
  path => "D:/Nginx/nginx-1.14.1-images/logs/*.log"
  start_position => "beginning"
        ignore_older => 0
  type => "nginx_log"
 }
}

filter{  
  
}

output {

 elasticsearch {     
  hosts => ["127.0.0.1:9200"]
 }
 stdout {codec => rubydebug}
}
```

配置完毕启动` logstash -f logstash.conf >> C:\Users\Panlf\Desktop\log.txt `， 因为有时候配置文件错误会导致启动不成功，所以我自己会在命令结束后添加信息到日志文件中。之后访问`localhost:9600`

4、下载Kibana，并启动`\kibana\bin\kibana.bat`，访问`localhost:5601`，并配置`index-patterns`即可观察到可视化的Nginx日志数据。
