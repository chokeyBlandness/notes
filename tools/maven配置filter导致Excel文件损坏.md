# maven配置filter导致Excel文件损坏

### 问题描述：

maven项目中使用到了Excel模板，在pom的profile->build->resources标签中添加了这些资源文件的扫描。在进行compile编译之后，目标Excel文件就会受损。

### 问题原因：

maven的filter配置会对资源文件进行过滤操作。

```xml
<resourcce>
	<directory>src/main/resources</directory>
    <includes><include>**/**</include></includes>
    <filtering>true</filtering>
</resourcce>
```

filter会用环境变量、pom文件里定义的属性和指定配置文件里的属性替换属性(`*.properties`等)文件里的占位符，`src/main/resources/`目录中有一些用于其他目的的二进制文件, 比如Excel、Word, 这些文件会被filter扫描到并且改变编码格式。

### 解决方法：

在pom配置中将这些二进制文件exclude掉，以免启用过滤功能时损坏这些文件。

```xml
<resourcce>
	<directory>src/main/resources</directory>
    <includes><include>**/**</include></includes>
    <excludes><exclude>*.xlsx</exclude></excludes>
    <filtering>true</filtering>
</resourcce>
```

