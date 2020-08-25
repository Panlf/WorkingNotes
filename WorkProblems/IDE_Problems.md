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