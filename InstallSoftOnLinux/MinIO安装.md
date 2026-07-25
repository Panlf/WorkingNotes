# MinIO分布式部署

部署脚本

```shell
RUNNING_USER=root
MINIO_HOME=/opt/minio
MINIO_HOST=192.168.22.10
#accesskey and secretkey
ACCESS_KEY=minio
SECRET_KEY=minio123

for i in {01..04}; do
    START_CMD="MINIO_ACCESS_KEY=${ACCESS_KEY} MINIO_SECRET_KEY=${SECRET_KEY} nohup ${MINIO_HOME}/minio  server --address "${MINIO_HOST}:90${i}" http://${MINIO_HOST}:9001/opt/min-data1 http://${MINIO_HOST}:9002/opt/min-data2 http://${MINIO_HOST}:9003/opt/min-data3 http://${MINIO_HOST}:9004/opt/min-data4 > ${MINIO_HOME}/minio-90${i}.log 2>&1 &"
    su - ${RUNNING_USER} -c "${START_CMD}"
done
```

- 所有运行分布式 MinIO 的节点需要具有相同的访问密钥和秘密密钥才能连接。建议在执行 MINIO 服务器命令之前，将访问密钥作为环境变量，MINIO access key 和 MINIO secret key 导出到所有节点上 。
- Minio 创建 4 到 16 个驱动器的擦除编码集。
- Minio 选择最大的 EC 集大小，该集大小除以给定的驱动器总数。例如，8 个驱动器将用作一个大小为 8 的 EC 集，而不是两个大小为 4 的 EC 集 。
- 建议所有运行分布式 MinIO 设置的节点都是同构的，即相同的操作系统、相同数量的磁盘和相同的网络互连 。
- 运行分布式 MinIO 实例的服务器时间差不应超过 15 分钟。

```xml
upstream http_minio {
    server 192.168.22.10:9001;
    server 192.168.22.10:9002;
    server 192.168.22.10:9003;
    server 192.168.22.10:9004;
}

server{
    listen       8888;
    server_name  192.168.22.10;

    ignore_invalid_headers off;
    client_max_body_size 0;
    proxy_buffering off;

    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-Host  $host:$server_port;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto  $http_x_forwarded_proto;

        proxy_connect_timeout 300;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_ignore_client_abort on;

        proxy_pass http://http_minio;
    }
}
```
