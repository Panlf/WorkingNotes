## 问题列表_Windows

### 1、Windows查找端口是否被占用

今天使用`Hexo`的时候，本地`Hexo s`，然后访问`http:\\localhost:4000`，发现不能访问，猜想4000端口被占用。

Widows下查找占用端口：
```
//查找占用端口4000
netstat -ano | findstr "4000"  

//返回
TCP   127.0.0.1:4000  0.0.0.0:0    LISTENING       2084
```

查看进程：
```
tasklist | findstr "2084" 

//返回
FoxitProtect.exe   2084 Services    0      6,412 K
```

杀掉进程：
```
taskkill /im /t /f FoxitProtect.exe 

//也可以使用pid杀
taskkill /pid pid /t /f
```



