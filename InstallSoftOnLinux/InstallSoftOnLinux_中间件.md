## 在Linux下安装软件_中间件

### 1、Redis安装

环境

- Redis 6.0.8
- Ubuntu 18.04

[官网](https://redis.io/download)上提供编译安装
```
$ wget http://download.redis.io/releases/redis-6.0.8.tar.gz
$ tar xzf redis-6.0.8.tar.gz
$ cd redis-6.0.8
$ make
```

但是编译安装，前提是需要安装`make`和`gcc`命令，不然无法编译。

Ubuntu安装gcc
```
默认的Ubuntu存储库包含一个名为build-essential的元包，它包含GCC编译器以及编译软件所需的许多库和其他实用程序。

1、sudo apt update
2、sudo apt install build-essential
3、gcc --version
```

编译过程中也发现了一个错误
```
d src && make all
make[1]: Entering directory '/opt/redis/redis-6.0.8/src'
/bin/sh: 1: pkg-config: not found
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory
 #include <jemalloc/jemalloc.h>
          ^~~~~~~~~~~~~~~~~~~~~
compilation terminated.
Makefile:317: recipe for target 'adlist.o' failed
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory '/opt/redis/redis-6.0.8/src'
Makefile:6: recipe for target 'all' failed
make: *** [all] Error 2
```

原因是jemalloc重载了Linux下的ANSI C的malloc和free函数。解决办法：make时添加参数。
```
make MALLOC=libc
```

上述即可启动Redis。
