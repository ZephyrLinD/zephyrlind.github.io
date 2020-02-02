# SSM框架学习笔记（一） 相关环境的搭建


## 安装JDK

只需安装JDK8即可。不做赘述。

## 安装TomCat

安装TomCat8，装在想要解压的位置即可，不做赘述。

## 安装Maven

从Apache网站上下载Maven，解压。

打开`conf`文件夹中的`settings.xml`

第一步是配置本地仓库：

找到注释中的如下代码段：

```xml
<!-- localRepository
 | The path to the local repository maven will use to store artifacts.
 |
 | Default: ${user.home}/.m2/repository
<localRepository>/path/to/local/repo</localRepository>
-->
```

中的注释行最后一行挪出来，换到别的地方

```xml
<localRepository>c:\develop\.m2\repository</localRepository>
```

我只是以自己为例，中间是路径。

第二步是添加镜像服务器仓库，

在`<mirrors>`标签下，添加代码如下：

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

## 安装Idea、MySQL过程不再赘述，官网下载安装即可

## IDEA Maven配置

IDEA的项目和eclipse不同，idea的项目内可以包含模块，而模块相当于eclipse的项目。

所以我们一般新建一个空项目来导入模块。新建空项目，然后点击加号新建一个Maven模块。然后根据下图选择：

[![1.png](https://img.zephyrl.co/images/2020/02/02/1.png)](https://img.zephyrl.co/image/Uzu)

点击下一步，写出自己的组名，再次点击下一步:

[![2.png](https://img.zephyrl.co/images/2020/02/02/2.png)](https://img.zephyrl.co/image/3Np)

之后是让选择自己的Maven二进制文件、配置文件的位置和包存储位置，填写上面设置的即可：

[![3.png](https://img.zephyrl.co/images/2020/02/02/3.png)](https://img.zephyrl.co/image/HX1)

[![4.png](https://img.zephyrl.co/images/2020/02/02/4.png)](https://img.zephyrl.co/image/tFa)

在之后就进入了漫长的读条时间，第一次编译会很慢，后几次就不会这么慢了，出现`BUILD SUCCEED`即可说明配置完成，点击应用即可。

[![5.png](https://img.zephyrl.co/images/2020/02/02/5.png)](https://img.zephyrl.co/image/brq)

打开Maven生成的`pom.xml`文件，找到如下代码段，将JDK版本改成自己版本（我是1.8）,然后在`<dependency>`模块内的`junit`版本换成4.12

[![6.png](https://img.zephyrl.co/images/2020/02/02/6.png)](https://img.zephyrl.co/image/FSW)

之后在左边的目录下创建四个文件夹，文件夹名和结构如下图：

[![7.png](https://img.zephyrl.co/images/2020/02/02/7.png)](https://img.zephyrl.co/image/Gsy)

之后要把文件夹的标签换成相应标签，有两种办法，第一种办法是右键文件夹，选择`Mark Directory as`然后改成相对文件夹：

[![8.png](https://img.zephyrl.co/images/2020/02/02/8.png)](https://img.zephyrl.co/image/N2Vh)

第二种办法是打开工程设置，然后点击文件夹和想要标记的文件夹类型一一对应即可：

[![9.png](https://img.zephyrl.co/images/2020/02/02/9.png)](https://img.zephyrl.co/image/lAX)

之后，删掉`index.jsp`，新建一个JSP文件，取名为`index.jsp`

此时可以发现代码头部已经标记了如下代码段：

```jsp
<% page contentType="text/html;charset=UTF-8" language="java" %>
```

此时相关配置就完成啦！

## 总结

本节课已经将我们需要的实验环境搭建完成啦！接下来可以通过这个环境进行学习和实验！
