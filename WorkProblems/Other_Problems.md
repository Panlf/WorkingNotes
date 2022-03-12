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
