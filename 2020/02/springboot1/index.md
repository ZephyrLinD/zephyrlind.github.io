# SpringBoot学习笔记（一） —— 一切的开始


## 起源：疫情

疫情是不得不说的一个原因。这次武汉新型冠状病毒导致整个社会的几乎停摆。在此首先向一直在一线奋斗的医疗工作人员们致以最高的敬意；也对造谣的某些媒体、个人，还有至今仍在走形式主义官僚主义的当官的，发国难财吃人血馒头的人，还有 Youtube、Twitter、Ins 等国外开放媒体上意图抹黑分裂祖国的、假意爱国实则造谣的意图抹黑分裂祖国的某些个人、媒体也好动物也好说一句：NMSL。

这毕竟是我的个人博客。虽然博客主要写技术。但是偶尔也要夹杂一些随笔在里面。因为这些年光瞎玩了就是不写字。文笔太差所以只能夹杂在里面了请大家见谅（我相信没人会看哈哈哈哈）。

**言归正传**。这次疫情导致我也没法去学校上课。学校要求的一些课只能通过网络授课来完成。其中就包括 SpringBoot 这个我想学的后端框架。没法面授，未知原因不让直播，未知原因不让看录播课。所以只能用一些开放平台的课程来代替。所以我决定，自己找课学一下 SpringBoot。所以 SpringBoot 的学习笔记会根据 [B站Up主 `狂神说Java` 的 SpringBoot 课程](https://www.bilibili.com/video/av75233634) 进行记录。

至于之前断更的 SSM 框架的笔记我会逐渐补全。

## SpringBoot 的特点

Spring Boot 基于 Spring 开发，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于 Spring 框架的应用程序。也就是说，它并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。Spring Boot 以约定大于配置的核心思想，默认帮我们进行了很多设置，多数 Spring Boot 应用只需要很少的 Spring 配置。同时它集成了大量常用的第三方库配置（例如 Redis、MongoDB、Jpa、RabbitMQ、Quartz 等等），Spring Boot 应用中这些第三方库几乎可以零配置的开箱即用

## 项目主结构生成

SpringBoot 的 Demo 可以从 [Spring 官网](https://spring.io) 的 [Spring Initializr](https://start.spring.io/) 找到

[![1_Spring-Initializr.md.png](https://img.zephyrl.co/images/2020/02/18/1_Spring-Initializr.md.png)](https://img.zephyrl.co/image/No78)

如图所示，按照图示选择对应的版本、语言后就可以把整个工程文件下载下来。然后在 IDEA 中 Import Project 就可以了

但是其实，IDEA 自带这个 `Spring Initializr`，我们不需要从网上导入这个项目。在 IDEA 中新建项目，点击下图中的 Spring Initializr 即可和上面一样

[![2_Spring-Initializr-IDEA.md.png](https://img.zephyrl.co/images/2020/02/18/2_Spring-Initializr-IDEA.md.png)](https://img.zephyrl.co/image/NjyO)

[![3_.md.png](https://img.zephyrl.co/images/2020/02/18/3_.md.png)](https://img.zephyrl.co/image/NBK2)

[![4_chooseversion.md.png](https://img.zephyrl.co/images/2020/02/18/4_chooseversion.md.png)](https://img.zephyrl.co/image/N7YB)

之后在选下一步即可完成创建。完成创建后的样子如下图所示，自动帮我们导入很多的包

[![5_success.md.png](https://img.zephyrl.co/images/2020/02/18/5_success.md.png)](https://img.zephyrl.co/image/NTGK)

然后我们点开项目目录下的 java 文件夹，打开那些包，点击里面的 `demoApplication.java`，**这就是我们程序的入口**

运行代码。运行结果如图所示

[![6_starthelloworld.md.png](https://img.zephyrl.co/images/2020/02/18/6_starthelloworld.md.png)](https://img.zephyrl.co/image/N8sG)

此时我们就可以打开 `http://localhost:8080` 和 `http://localhost:8080/error` 查看第一个 Demo 了

[![7_8080.md.png](https://img.zephyrl.co/images/2020/02/18/7_8080.md.png)](https://img.zephyrl.co/image/NM3E)

之后，我们要在和**程序入口的同级创建包**，在这里面开发。比如我创建一个 `controller` 包。应该在下图位置。

![7_package.png](https://img.zephyrl.co/images/2020/02/18/7_package.png)

## SpringBoot 项目结构分析

通过上面步骤完成了基础项目的创建。就会自动生成以下文件。

- 程序的主程序类
- 一个 application.properties 配置文件
- 一个测试类

生成的 `DemoApplication` 和测试包下的 `DemoApplicationTests` 类都可以直接运行来启动当前创建的项目，由于目前该项目未配合任何数据访问或 Web 模块，程序会在加载完 Spring 之后结束运行。

## 编写 Hello, World HTTP 接口

首先，我们创建上图的 `Controller` 包

然后，在 `Controller` 包内我们创建一个 `HelloController` 类

```java
// helloWorld.java
@RestController
public class helloWorld {
    // 接口： http://localhost:8080/hello
    @RequestMapping("/hello")
    public String sayHello() {
        // 调用业务，接受前端的参数
        return "Hello, World!";
    }
}
```

编写完毕后，从主程序启动项目，浏览器发起请求，访问 hello 请求，字符串成功返回！

最后，如下图，打成 jar 包

[![8_jar.md.png](https://img.zephyrl.co/images/2020/02/19/8_jar.md.png)](https://img.zephyrl.co/image/Neai)

## 修改端口号

在 `src/resources/application.properties` 中可以修改服务器端口号：

```properties
# 修改端口号
server.port=8081
```

## 彩蛋

`src/resources/` 文件夹内可以新建一个叫 `banner.txt` 的文件，存放一个像素画，这样可以替代 SpringBoot 启动服务器的时候的 Banner

[![9_draw.md.png](https://img.zephyrl.co/images/2020/02/19/9_draw.md.png)](https://img.zephyrl.co/image/NgDR)

比如我改成了这个巨大的像素画
