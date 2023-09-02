# Log知识点

## Slf4j+Log4j2
### 引入Jar
```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.32</version>
</dependency>

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.0</version>
</dependency>

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.0</version>
</dependency>

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.17.0</version>
</dependency>
```


### log4j2.xml配置
```
<?xml version="1.0" encoding="UTF-8"?>
<!-- 
    status：这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出。
    monitorInterval : Log4j2能够自动检测修改配置文件和重新配置本身，设置间隔秒数，单位是s,最小是5s. 
-->
<Configuration status="DEBUG" monitorInterval="30">
    <Appenders>
        <!-- Console节点 配置控制台日志输出： 
            name:指定Appender的名字.
            target: SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT.
            PatternLayout: 输出格式，不设置默认为:%m%n.
        -->
        <Console name="Console" target="SYSTEM_OUT">
            <!-- ThresholdFilter 过滤器： 
                控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch)
                日志的级别： ALL< Trace < DEBUG < INFO < WARN < ERROR < FATAL < OFF。
            -->
            <ThresholdFilter level="DEBUG" onMatch="ACCEPT"
                onMismatch="DENY" />
            <!-- PatternLayout 日志输出格式模板：
                %d{yyyy-MM-dd HH:mm:ss.SSS} 日志生成时间。
                %-5level 日志级别(级别从左显示5个字符宽度)。
                %logger{36} 表示logger名字最长36个字符,否则按照句点分割。
                %L 日志输出所在行数 日志输出所在行数 
                [%t] 输出产生该日志事件的线程名
                %msg 日志消息
                %n 是换行符
            -->
            <PatternLayout
                pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger{36}:%L [%t] - %msg%n" />
        </Console>
    </Appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <Loggers>
        <!--过滤掉spring的一些无用的DEBUG信息-->
        <logger name="org.springframework" level="INFO"></logger>
        <!-- 配置日志的根节点 -->
        <!-- level="all" 只能够输出level级别是all及all以上的-->
        <root level="all">
            <appender-ref ref="Console" />
        </root>
    </Loggers>
</Configuration>
```

## logback无法产生每日日志
```
 <FileNamePattern>${LOG_HOME}/log-%d{yyyy-MM-dd}-%i.log</FileNamePattern>
```

后来查资料发现需要加入`%i`，但是也不是直接加入`%i`就行的，不然也会解析失败，就是`class`也要改成`ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy`，下面就是修正并测试正常的配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="/data/log"/>
    <!-- 控制台输出 -->
    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] [%-5level] [%logger{50}] [%file:%line] - %msg%n</pattern>
        </encoder>
    </appender>
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/log-%d{yyyy-MM-dd}-%i.log</FileNamePattern>
            <maxFileSize>30MB</maxFileSize>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>

        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%-5level] [%logger{50}] [%file:%line] - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>

</configuration>
```
