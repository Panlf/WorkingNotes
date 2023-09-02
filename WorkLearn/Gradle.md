# Gradle知识点
## Gradle的依赖范围
|名称|说明|
|---|---|
|compileOnly|gradle2.12之后版本新添加的,2.12版本时期曾短暂的叫provided,后续版本已经改成了compileOnly,由java插件提供,适用于编译期需要而不需要打包的情况|
|providedCompile|war插件提供的范围类型:与compile作用类似,但不会被添加到最终的war包中这是由于编译、测试阶段代码需要依赖此类jar包，而运行阶段容器已经提供了相应的支持，所以无需将这些文件打入到war包中了;例如Servlet API就是一个很明显的例子.|
|compile|	编译范围依赖在所有的classpath中可用，同时它们也会被打包。|
|providedRuntime|	同proiveCompile类似。|
|runtime|	runtime依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现。|
|testCompile|	测试期编译需要的附加依赖|
|testRuntime|	测试运行期需要|
|archives|	-|
|default	|配置默认依赖范围|

## 高版本中lombok失效

我将Gradle版本升为6.5，我的SpringBoot项目就启动报错，报错原因主要是lombok的注解报错。然后经过查Google，翻文档发现需要修改lombok相关引入。

修改后的配置
```
...
repositories {
	jcenter()
}

dependencies {
	implementation libs.springboot
	//必须如下引入，不然启动报错
	compileOnly libs.lombok
	annotationProcessor libs.lombok
}
...
```