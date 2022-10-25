### 1、正则获取两个特殊符号之间的字符串
```
表达式: A.*?B（"."表示任意字符，"?"表示匹配0个或多个） 
示例: Abaidu.comB 
结果: Awww.apizl.comB 

匹配两个字符串A与B中间的字符串包含A但是不包含B： 
表达式: A.*?(?=B) 
示例: Awww.apizl.comB 
结果: Awww.apizl.com 

匹配两个字符串A与B中间的字符串且不包含A与B： 
表达式: (?<=A).*?(?=B) 
示例: Awww.baidu.comB 
结果: www.baidu.com
```
### 2、正则获取括号里的内容
```
[\((][^\))]+[\))]$
```

### 3、Datax传输生僻字

MySql数据库的编码支持UFT8字符集。utf-8编码可能是2个字节、3个字节、4个字节的字符，MYSQL的utf-8编码，只支持3个字节的字符。汉字中很多生僻字都是4个字节的字符，日常生活中人的姓名就会有很多高位的生僻字。
如果直接使用datax同步数据到utf-8编码的数据库中，遇到高位字节的字符时，程序会抛异常。即便数据库中的表的字符集是设置为uft8mb4字符集。在datax异常日志中：
```
java.sql.BatchUpdateException:Incorrect string value:'xF0xA1x80x84' for column 'XXXX' at row 66.
```
如果在创建数据库实例的时候，就把实例创建成uft8mb4字符集，就不会出现这个问题。之前这个问题，datax是没有解决的，我都是通过重新创建数据库实例来实现。现在datax有方法可以解决这个问题，方法就是在jdbc配置中增加?com.mysql.jdbc.faultInjection.serverCharsetIndex=45。例如：
```
jdbc:mysql://ip:3306/testabc?com.mysql.jdbc.faultInjection.serverCharsetIndex=45
```

### 4、根据分隔符分割保留分隔符
```
(?<=(省|市|区))
```

主要是用来切割省市区，并保留省市区整个名称。
