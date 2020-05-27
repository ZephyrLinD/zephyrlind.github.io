# SSM框架学习笔记（七） SpringMVC之MVC回顾


## 利用 Maven 依赖创建项目

### 创建父项目

在 IDEA 中，创建一个 Maven 空项目，点击 `pom.xml` 导入以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```

之后，删除 src 文件夹，创建子项目

### 创建子项目（所有子项目以此类推）

点击模块按钮，依旧是新建一个 Maven 空模块

我们可以发现 `src` 目录下可以识别文件夹且

然后右键模块，选择 Add Framework Support

![addFrameworkSupport](https://img.zephyrl.co/images/2020/03/04/1_addFrameworkSupport.png)

在之后点击 Web Application ---> OK 即可将他变成一个 web 项目

![web](https://img.zephyrl.co/images/2020/03/04/2_webProject.png)

可以发现，目录下多了 web 文件夹，里面也有我们的 `web.xml`，即 jsp 配置 servlet 的文件，同时 web 文件夹上有个小蓝点

[![3_successPoint.png](https://img.zephyrl.co/images/2020/03/04/3_successPoint.png)](https://img.zephyrl.co/image/NOK6)

然后进入**模块的** `pom.xml` 导入以下包：

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
```

## 编写 Servlet 回顾

首先我们编写一个 Servlet 来实现处理用户请求

```java
package co.zephyrl.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 1. 获取前端参数
        String method = req.getParameter("method");
        if(method.equals("add")) {
            req.getSession().setAttribute("msg", "执行了add方法");
        }
        if(method.equals("delete")) {
            req.getSession().setAttribute("msg", "执行了delete方法");
        }
        // 2. 调用业务层
        // 3. 重定向或视图转发
        req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

编写 Hello.jsp，在 WEB-INF 目录下新建一个 jsp 的文件夹，新建 `hello.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Kuangshen</title>
</head>
<body>
    ${msg}
</body>
</html>
```

在 `web.xml` 注册 Servlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.kuang.servlet.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/user</url-pattern>
    </servlet-mapping>

</web-app>
```

之后启动 Tomcat，启动测试

- localhost:8080/user?method=add
- localhost:8080/user?method=delete

## MVC 框架需要做哪些事情

1、将url映射到java类或java类的方法

2、封装用户提交的数据

3、处理请求--调用相关的业务处理--封装响应数据

4、将响应的数据进行渲染 jsp / html 等表示层数据
