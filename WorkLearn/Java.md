# Java的知识点

## BigDecimal
### float和double的精度

在使用BigDecimal先了解float和double的精度问题，float占32位(bit),double占64位，即float占4个字节，double占8个字节。

float和double的范围是由指数的位数来决定的。float的存储时候1个bit是符号位，8个bit是指数位，剩下的23个bit是有效数字位，double存储的时候1个bit是符号位，11个bit是位指数位，剩下的52个bit是尾数位。精度由尾数决定，float的精度为7~8位有效数字，double的精度为16~17位。

我们在使用float或者double会出现精度问题
```
float a = 0.05f;
float b = 0.01f;
System.out.println(a + b);
```
这个代码出现的结果不是0.06，而是0.060000002，同样double也会出现这样的问题
```
double c = 0.05d;
double d = 0.01d;
System.out.println(c + d);  
```
打印结果是0.060000000000000005。

基于上述会出现的精度丢失的问题，我们有必要使用BigDecimal进行精确的计算。

### 构造函数
```
BigDecimal(int)
创建一个具有参数所指定整数值的对象

BigDecimal(double)
创建一个具有参数所指定双精度值的对象

BigDecimal(long)
创建一个具有参数所指定长整数值的对象

BigDecimal(String)
创建一个具有参数所指定以字符串表示的数值的对象
```

虽然构造函数支持float、double，但是我们也千万要注意使用string，不然同样会出现精度问题
```
float a = 0.05f;
float b = 0.01f;
BigDecimal m = new BigDecimal(a);
BigDecimal n = new BigDecimal(b);
System.out.println(m.add(n));
```
结果是0.060000000000000005，因为你以为传入a的值是0.05，但是BigDecimal构造的时候不是0.005，而是其更精确的值。这样就还是会出现精度问题，所以要用string方式构造。
```
BigDecimal s = new BigDecimal(String.valueOf(a));
BigDecimal t = new BigDecimal(String.valueOf(b));
System.out.println(s.add(t));
```
结果就是0.06

### 常用方法
```
add(BigDecimal)
BigDecimal对象中的值相加，返回BigDecimal对象

subtract(BigDecimal)
BigDecimal对象中的值相减，返回BigDecimal对象

multiply(BigDecimal)
BigDecimal对象中的值相乘，返回BigDecimal对象

divide(BigDecimal)
BigDecimal对象中的值相除，返回BigDecimal对象

toString()
将BigDecimal对象中的值转换成字符串

doubleValue()
将BigDecimal对象中的值转换成双精度数

floatValue()
将BigDecimal对象中的值转换成单精度数

longValue()
将BigDecimal对象中的值转换成长整数

intValue()
将BigDecimal对象中的值转换成整数
```