### Java执行Linux命令

今天开发一个简单程序，需要用Java程序调用Linux命令来执行datax任务，但是`.../datax.py ..../job.json > ...log`，该命令有管道重定向的语句，一直不能执行成功，但是也不抛出异常。后来改变一下写法即可。

存在问题
```
public static boolean executeLinuxCmd(String cmd){
        Runtime run = Runtime.getRuntime();
        boolean result = true;
        try {
            Process process = run.exec(cmd);
            //执行结果 0 表示正常退出
            int exeResult=process.waitFor();
            if(exeResult==0){
                log.info("Execute cmd success : {}",cmd);
            }else{
                result = false;
                log.error("Execute cmd failed : {}",cmd);
            }
        }
        catch (Exception e) {
            log.error("LinuxCmdUtils.executeLinuxCmd error message:{}",e.getMessage());
            return false;
        }
        return result;
    }
```

修改之后
```
public static boolean executeLinuxCmd(String cmd){
      Runtime run = Runtime.getRuntime();
      boolean result = true;
      try {
          String[] cmdArr = new String[]{"sh","-c",cmd};
          Process process = run.exec(cmdArr);
          //执行结果 0 表示正常退出
          int exeResult=process.waitFor();
          if(exeResult==0){
              log.info("Execute cmd success : {}",cmd);
          }else{
              result = false;
              log.error("Execute cmd failed : {}",cmd);
          }
      }
      catch (Exception e) {
          log.error("LinuxCmdUtils.executeLinuxCmd error message:{}",e.getMessage());
          return false;
      }
      return result;
  }
```
### SpringBoot项目部署到Tomcat
#### 版本
- SpringBoot 2.3.12.RELEASE
- JDK 11
- Tomcat 9.0.67

#### 配置

##### 依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.plf</groupId>
    <artifactId>deployment-tutorial</artifactId>
    <version>0.0.1</version>
    <name>deployment-tutorial</name>
    <description>Deployment project for Spring Boot</description>
    <packaging>war</packaging>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>deployment-tutorial</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <profileActive>dev</profileActive>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profileActive>test</profileActive>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profileActive>prod</profileActive>
            </properties>
        </profile>
    </profiles>
</project>
```

注意使用`war`打包方式。

##### 启动类
```
package com.plf.deployment;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class DeploymentTutorialApplication  extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(DeploymentTutorialApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(DeploymentTutorialApplication.class, args);
    }

}
```

注意需要继承`SpringBootServletInitializer`。

##### 配置文件
application.yml
```
spring:
  profiles:
    active: @profileActive@
```

application-dev.yml
```
test:
  info: dev
```


application-prod.yml
```
test:
  info: prod
```

主要是为了区分测试环境和线上环境的测试

##### 接口

```
package com.plf.deployment.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author panlf
 * @date 2022/9/29
 */
@RestController
@RequestMapping("test")
public class TestController {

    @Value("${test.info}")
    private String info;

    @GetMapping("getInfo")
    public String test(){
        return info+"测试成功";
    }
}
```

##### 部署

打包
```
mvn clean -Pprod -Dmaven.test.skip=true
```

打包出来的`war包`部署到`Tomcat`的`webapps`中，启动`bin`目录下的`startup.bat`，访问`http://localhost:8080/deployment-tutorial/test/getInfo`即可看到结果。
