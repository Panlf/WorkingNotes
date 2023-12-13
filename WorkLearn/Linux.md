# Linux的知识点

## shell脚本变量出现特殊字符的时候
例如我有一个密码是`D)K+W3=#@!B&2w`，直接赋值给shell脚本的变量，会报错，需要用单引号括一下
```
PASSWORD='D)K+W3=#@!B&2w'
```

## 查询应用的端口
### lsof

语法格式
```
lsof -i:端口
```

显示信息
```
COMMAND: 进程的名称
PID: 进程标识符
USER: 进程所有者
FD: 文件描述符，应用程序通过文件描述符识别该文件。
TYPE: 文件类型
DEVICE: 指定磁盘的名称
SIZE: 文件的大小
NODE: 索引节点（文件在磁盘上的标识）
NAME: 打开文件的确切名称
```

### netstat

语法格式
```
netstat -tunlp | grep 端口号
```

```
-t 显示tcp连接信息
-u 显示udp相关信息
-n 使用数字形式的IP、端口号、用户ID替代主机、协议、用户等名称信息
-l 仅显示正在监听的sockets接口信息
-p 显示进程名称及对应进程ID
```

## 文本处理
### grep

行过滤工具；用于查找文件里符合条件的字符串。

#### 语法

grep [选项] '关键字' 文件名


常用选项

- -i	不区分大小写
- -v	查找不包含指定内容的行，反向选择
- -w	按单词搜索
- -o	打印匹配关键字
- -c 	统计匹配的次数
- -n	显示行号
- -r	逐层遍历目录查找
- -A	显示匹配行及后面多少行
- -B	显示匹配行及前面多少行
- -C	显示匹配行及前后多少行
- -l	只列出匹配的文件名
- -L	列出不匹配的文件名
- -e	使用正则匹配
- -E	使用扩展正则匹配
- ^key	以关键字开头
- key$	以关键字结尾
- ^$	匹配空行
- --color=auto	可以将找到的关键词部分加上颜色的显示

#### 测试
```
> cat test.txt
Hello
World
Linux
Ubuntu

> grep -inw 'linux' test.txt
3:Linux

```

### cut工具

列截取工具；用于文本文件列的截取。

#### 语法

cut  选项  文件名

常用选项

- -c	以字符为单位进行分割，截取
- -d	自定义分割符，默认为制表符\t
- -f	与-d一起使用，指定截取哪个区域

#### 测试
```
> cat test.txt
Hello World
Linux Ubuntu

# 按空格分隔 截取第一列内容
> cut -d ' ' -f1 test.txt
Hello
Linux

# 截取文件中每行的1到5个字符
> cut -c1-5 text.txt 
Hello
Linux
```


### sort工具

sort工具用于排序，它将文件的每一行做为一个单位，从首字符向后，依次按ACSII 码值进行比较，最后将他们按升序输出。

#### 语法

sort [选项] 文件

常用选项

- -u	去除重复行
- -r	降序排列，默认是升序
- -o	将排序结果输出到文件中，类似重定向符号>
- -n	以数字排序，默认是按字符排序
- -t	分割符
- -k    第N列
- -b	忽略每行前面开始出的空格字符

#### 测试
```
> cat test.txt
DDD
AAA
CCC
BBB

> sort test.txt
AAA
BBB
CCC
DDD

> cat test.txt
DDD	2
AAA 4
CCC 5
BBB	1

#  以空格分割第2行升序排列
> sort -t ' ' -k2 test.txt
BBB	1
DDD	2
AAA 4
CCC 5
```

### uniq

uniq用于去除连续的重复行
	
#### 语法

uniq [选项] 文件

常用选项

- -u	只显示没有重复的纪录
- -c	统计重复行次数
- -d	只显示重复行

#### 测试

```
> cat test.txt
AAA
BBB
BBB
bbb
CCC
AAA

> uniq test.txt
AAA
BBB
bbb
CCC
AAA

> uniq -u test.txt
AAA
bbb
CCC
AAA
```

### tee工具

tee工具时从标准输入读取并写入到标准输出和文件。即：双向覆盖重定向（屏幕输出|文本输入）。

#### 语法

tee [参数] [文件]

常用选项

- -a 附加到既有文件的后面，而非覆盖它

#### 测试
```
> echo 'hello word' | tee test.txt
> cat test.txt
hello world
```

### diff

diff工具逐行比较文件的不同。

#### 语法

diff [选项]  文件1  文件2

常用选项

- -b	不检查空格
- -B	不检查空白行
- -i	不检查大小写
- -w	忽略所有的空格
- --normall	正常格式显示（默认）
- -c	上下文格式显示
- -u	合并格式显示


#### 测试
```
> cat file1.txt
AA
CC
BB
QQ

> cat file2.txt
CC
DD
AA

> diff file1.txt file2.txt
1d0  # 第一个文件要删除第一行才能与第二个文件的0行匹配
< AA 	# 后面的文件需要添加
3,4c2,3	# 第一个文件3,4行要改变才能与第二个文件的2,3行匹配
< BB  # 后面的文件需要添加
< QQ # 后面的文件需要添加
---
> DD # 前面的文件需要添加
> AA # 前面的文件需要添加 

> diff file1.txt file2.txt -y
AA  <
CC 	CC
BB 	| DD
QQ	| AA
```

*注意*

"|"表示前后2个文件内容有不同，"<"表示后面文件比前面文件少了1行内容，">"表示后面文件比前面文件多了1行内容。


### paste

paste工具用于合并文件行

#### 语法

paste [参数] [文件1] [文件2]

常用选项

- -d	自定义间隔符，默认是tab
- -s	串行处理，非并行 第一行第一个文件，第二行第二个文件

#### 测试

```
> cat file1.txt
AA
CC
BB
QQ

> cat file2.txt
CC
DD
AA

> paste file1.txt file2.txt
AA  CC
CC  DD
BB  AA
QQ
```

### tr

tr用于字符转换、替换和删除，主要用于删除文件中控制字符或进行字符转换。

#### 语法

tr [参数] [字符串1] [字符串2]

常用选项

- -d	删除字符串1中所有输入字符
- -s	删除所有重复出现字符序列，只保留第一个，即将重复出现字符串压缩为一个字符串
- a-z
- A-Z
- 0-9
- [:alnum:] 	所有字母字符与数字
- [:alpha:] 	所有字母字符
- [:blank:] 	所有水平空格
- [:cntrl:] 	所有控制字符
- [:digit:] 	所有数字
- [:graph:] 	所有可打印的字符(不包含空格符)
- [:lower:] 	所有小写字母
- [:print:] 	所有可打印的字符(包含空格符)
- [:punct:] 	所有标点字符
- [:space:] 	所有水平与垂直空格符
- [:upper:] 	所有大写字母


#### 测试

``` 
# 所有小写字母改为大写字母
> echo 'hello WORLD' | tr [:lower:] [:upper:]
HELLO WORLD
```
