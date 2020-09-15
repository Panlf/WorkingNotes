## 问题列表_IDE

### 1、使用IDEA出现Lombok的编译错误

我已经在IDEA中安装了Lombok插件，在编写代码中也能够成功导入，但是运行代码会报编译错误，然后需要在IDEA中设置才能使用。
```
file - >  Setting - Compiler - Annotation Processors - Enable annotation processing 
选中即可，运行成功。
```

### 2、Eclipse中不需要源码包也可查看源码

今天我使用Eclipse时候，想要查看某个类的源码，但是提示我需要导入源码，这样非常麻烦，就需要一个反编译插件。

Eclipse的反编译插件可以使用 - Eclipse Class Decompiler

Eclipse操作
```
1、Help - > Eclipse Marketplace 
2、输入Decompiler，选择Eclipse Class Decompiler安装
3、安装完成重启Eclipse
4、选择Window -> Preferences -> Gerenal -> Editors -> File Associations 
5、选择File types的*.class without source 把Class Decompiler Viewer设置为默认的editors
6、测试使用
```

### 3、IDEA下SpringBoot项目的热部署

1、File -> Settings -> Default Settings -> Build -> Compiler 然后勾选 Build project automatically

2、同时按住 Ctrl + Shift + Alt + / 然后进入Registry ，勾选自动编译并调整延时参数
```
compiler.automake.allow.when.app.running -> 自动编译
compiler.document.save.trigger.delay -> 自动更新文件
```
3、开启IDEA的热部署策略，顶部Application- >Edit Configurations
```
On 'Update' action:Update classes and resources
On frame deactivation:Update classes and resources
```
4、在项目添加热部署插件
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

有时候会碰到访问接口报404，因为DevTools的检测时间和idea的编译所需时间存在差异。在idea还没完成编译工作前，DevTools就开始进行重启和加载，导致@RequestMapping没有被全部正常处理。然后选择牺牲一点时间，去加长devtools的轮询时间，增大等待时间。这个时候只需要在application.properties里添加两个参数。
```
spring.devtools.restart.poll-interval=2s
spring.devtools.restart.quiet-period=1s
```
第一个参数的时间要大于第二个参数的时间。
