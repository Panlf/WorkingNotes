## 问题列表_Maven

### 1、Maven将第三方Jar打入Jar包

因为工作需要将一些代码打包发送到Linux服务器上进行运行，但是因为运行的代码依赖第三方库，所以需要将第三方库也打进运行的jar中。

#### 第三方依赖以Jar的形式存在

部署到Linux环境下需要将lib文件夹也带过去，不然提示无法到第三方Jar
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>package.MainClass</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/lib</outputDirectory>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
        </execution>
    </executions>
</plugin>
```


#### 第三方依赖以Class的形式打包到Jar中

Linux环境中只需要一个Jar包就可进行运行，因为第三方依赖都在Jar包中，所以文件大小也会大一点。
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>package.MainClass</mainClass>
            </manifest>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id> <!-- this is used for inheritance merges -->
            <phase>package</phase> <!-- 指定在打包节点执行jar包合并操作 -->
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 2、Maven打包SpringBoot项目

#### 命令行打包项目
```
cd 项目跟目录（和pom.xml同级）
mvn clean package
## 排除测试代码后进行打包
mvn clean package  -Dmaven.test.skip=true
```

#### Eclipse打包项目
```
选择项目右键Run As-->Maven build...
填入clean package  -Dmaven.test.skip=true
```

#### 运行项目
```
启动jar包命令
java -jar  target/xxx.jar
 
这种方式，只要控制台关闭，服务就不能访问了。所以需要在后台运行的方式来启动:(仅限linux环境)
nohup java -jar target/xxx.jar &

也可以在启动的时候选择读取不同的配置文件
java -jar xxx.jar --spring.profiles.active=dev

也可以在启动的时候设置jvm参数
java -Xms10m -Xmx80m -jar xxx.jar &
```

### 3、Maven命令package、install、deploy的联系与区别

#### 执行阶段
```
1、mvn clean package
    依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)等7个阶段。
2、mvn clean install
    依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install等8个阶段。
3、mvn clean deploy
    依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install、deploy等9个阶段。
```

#### 区别联系
```
1、package命令完成了项目编译、单元测试、打包功能，
    但没有把打好的可执行jar包(war包或其它形式的包)布署到本地maven仓库和远程maven私服仓库
2、install命令完成了项目编译、单元测试、打包功能，
    同时把打好的可执行jar包(war包或其它形式的包)布署到本地maven仓库，但没有布署到远程maven私服仓库
3、deploy命令完成了项目编译、单元测试、打包功能，
    同时把打好的可执行jar包(war包或其它形式的包)布署到本地maven仓库和远程maven私服仓库
```

### 4、Maven引入本地的Jar

```
<dependency>
    <groupId>com.plf.learn</groupId>
    <artifactId>Encryption</artifactId>
    <version>1.0.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/main/resources/lib/Encryption-1.0.0.jar</systemPath>
</dependency>
```
将Jar放到本项目下的src/main/resources/lib目录下即可。

### 5、将jar包安装到本地Maven仓库
```
mvn install:install-file -Dfile=tass-castle-2.0.1.jar -DgroupId=com.tass -DartifactId=tass-castle -Dversion=2.0.1 -Dpackaging=jar
```
