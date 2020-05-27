# SpringBoot学习笔记（三）—— Web 开发：静态资源映射和首页的定制


## 静态资源映射规则

首先，创建一个SpringBoot项目。

如果是 webapp 我们的主文件夹下有一个 `webapp` 文件夹，我们以前都是将所有的页面放在这里面的。但是用 `Spring Initalizer` 创建的 Spring Boot 项目是 `pom` 包，静态资源存放位置和以前是不一样的。

先来看一看静态资源映射规则：

Spring Boot 中，SpringMVC 的 web 配置都在 `WebMvcAutoConfiguration` 这个配置里面，可以看到 `WebMvcAutoConfigurationAdapter` 中有很多配置方法，

*该包位于 `org.springframework.boot.autoconfigure.web.servlet`*

### webjars资源，方法：`addResourcesHandlers`

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/")
                .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
            .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```

举例，在源代码第十一行我们可以得出，所有的 `/webjars/**`，都需要去 `classpath:/META-INF/resources/webjars/` 找对应的资源

那么什么是 webjars 呢？他本质就是以 jar 包的方式引入我们的资源。

比如我们想导入 jQuery，只需导入 jQuery 对应版本的 pom 依赖即可：

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

导入后我们在 External Libraries 中找到它，包结构如下：

![jQueryJarStructs](https://img.zephyrl.co/images/2020/02/26/1_webjarjqstructs.png)

此时启动服务器，因为 SpringBoot 可以通过路径来查找静态资源。我是到这里访问：

http://localhost:8080/webjars/jquery/3.4.1/jquery.js

即可发现访问到了 `jquery.js`

![showjs](https://img.zephyrl.co/images/2020/02/26/2_getJS.png)

### 如何导入项目中的静态资源

我们去找 `staticPathPattern`，可以发现第二种映射规则：/**。访问当前项目的任意资源，他会找 `resourcesProperties` 这个类，我们点进去看一看

```java
// 代码已经手动格式化
@ConfigurationProperties (
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
        "classpath:/META-INF/resources/",
        "classpath:/resources/",
        "classpath:/static/",
        "classpath:/public/"
    };
    // 其他源码
}
```

`resourceProperties` 可以设置和我们静态资源有关的参数，`this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS;` 这里面指向了它会去寻找资源的文件夹；即上面数组的内容。再向下看一个方法；它还会在根路径下去寻找资源。

```java
private String[] appendSlashIfNecessary(String[] staticLocations) {
    String[] normalized = new String[staticLocations.length];
    for (int i = 0; i < staticLocations.length; i++) {
        String location = staticLocations[i];
        normalized[i] = location.endsWith("/") ? location : location + "/";
    }
    return normalized;
}
```

所以我们可以得出结论，会从一下目录中读取静态资源到 /**。在 `localhost:8080` 下所有的请求都会被接收

- "classpath:/META-INF/resources/",
- "classpath:/resources/",
- "classpath:/static/",
- "classpath:/public/",
- "/" ：当前项目的根目录

所以我们可以在 resources 根目录创建对应的文件夹，他们其实都可以存放我们的静态文件。

为了测试他们的优先级，我们分别创建几个文件放在这几个文件夹内，如图

![静态资源比较大小](https://img.zephyrl.co/images/2020/02/26/3_compare.png)

分别运行，可以比较出优先级。

### 总结

1. 在 springboot 中，我们可以使用以下方式处理静态资源

    - webjars ---> `localhost:8080/webjars/`
    - public static /** resources 文件夹 ---> `localhost:8080`
  
2. 优先级：resources > static > public，但是我们一般使用 static

## 首页如何定制

我们在 `WebMvcAutoConfiguration.java` 中继续寻找，可以找到如下代码段：

![indexCode](https://img.zephyrl.co/images/2020/02/26/4_getIndexSource.png)

可以发现 他注册了 Bean，也支持自定义。

同时，index 页面也在我们上面说的那几个目录里可以。

我们进入 static 文件夹，新建一个 index.html 试一试

![showIndex](https://img.zephyrl.co/images/2020/02/26/5_showIndex.png)

成功啦！
