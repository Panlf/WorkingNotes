## 问题列表_2

### 1、使用IDEA出现Lombok的编译错误

我已经在IDEA中安装了Lombok插件，在编写代码中也能够成功导入，但是运行代码会报编译错误，然后需要在IDEA中设置才能使用。
```
file - >  Setting - Compiler - Annotation Processors - Enable annotation processing 
选中即可，运行成功。
```

### 2、将已有的项目上传到Github上
```
1、在github上新建的Repository -- 跟需上传项目的项目名称一致
2、在需上传项目的根目录在打开git，并进行以下命令
3、git init
4、git add .
5、git remote add origin [Repository地址]
6、git commit -m "提交说明"
7、git pull --rebase origin master
8、git push origin master
```

### 3、Eclipse中不需要源码包也可查看源码

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
