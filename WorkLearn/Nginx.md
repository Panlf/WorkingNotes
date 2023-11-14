# Nginx知识点

## Nginx配置
### Nginx批量载入多个配置文件
```
stream {
    include nginx.conf.d/*.stream;
}
```

### Nginx配置前端代码
```
 location /test-worktable {
    alias /opt/test/test-worktable;
    index  index.html index.htm;
    try_files $uri $uri/ /test-worktable/index.html;
}
```
