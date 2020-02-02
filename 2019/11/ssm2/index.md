# SSM框架学习笔记（二） Spring综述


## 什么是Spring

Spring是一个开放源代码的设计层面框架，他解决的是业务逻辑层和其他各层的松耦合问题，因此它将面向接口的编程思想贯穿整个系统应用。Spring是于2003 年兴起的一个轻量级的Java 开发框架，由Rod Johnson创建。简单来说，Spring是一个分层的JavaSE/EE full-stack(一站式) 轻量级开源框架。

## Spring的优势

1. 方便解耦，简化开发
2. AOP编程的支持
3. 声明式事务的支持
4. 方便程序的测试
5. 方便集成各种优秀框架
6. 降低Java EE API的使用难度
7. Java 源码是经典学习范例

## 模块划分

Spring Framework由大约20个模块组成的功能组成。这些模块分为核心容器，数据访问/集成，Web，AOP（面向方面​​编程），仪器，消息传递和测试，如下图所示

![Spring模块](https://img.zephyrl.co/images/2020/02/02/1ba3eac8c098e0956.png)

1. Spring核心容器：核心容器是Spring框架的重要组成部分，也可以说是Spring框架的基础。他在整个框架中的作用是负责管理对象的创建，管理，配置等等的操作。其主要包含spring-core，spring-beans，spring-context，spring-expression，spring-context-support组件。

2. 面向切面编程：Spring框架还提供了面向切面编程的能力，利用面向切面编程，可以实现一些面向对象编程无法很好实现的操作。例如，将日志，事务与具体的业务逻辑解耦。其主要包含spring-aop，spring-aspects组件。

3. Instrumentation：该模块提供了为JVM添加代理的功能，该模块包含spring-instrument，spring-instrument-tomcat组件，使用较少，不必过分关注。

4. 数据访问与集成：Spring框架为了简化数据访问的操作，包装了很多关于数据访问的操作，提供了相应的模板。同时还提供了使用ORM框架的能力，可以与很多流行的ORM框架进行整合，如hibernate，mybatis等等的著名框架。还实现了数据事务的能力，能够支持事务。包含spring-jdbc，spring-tx，spring-orm，spring-oxm，spring-jms，spring-messaging组件。

5. Web和远程调用：Spring框架支持Web开发，以及与其他应用远程交互调用的方案。包含spring-web，spring-webmvc，spring-websocket，spring-webmvc-portlet组件。

6. Spring测试：Spring框架提供了测试的模块，可以实现单元测试，集成测试等等的测试流程，整合了JUnit或者TestNG测试框架。包含spring-test组件。

以上就是Spring框架的几个主要模块。而在实际的开发过程中，不必要引入所有的Jar包，只需要引入与自己项目相关的库就行了。例如，只需要Spring框架的容器功能，实现依赖注入，那么只需要引入核心框架的几个库就完成了，其他模块例如Aop模块的库是没有必要引入的。

## Spring快速上手

1. 导入Spring Context核心包

    前往Maven的[mvnrepository页面](https://mvnrepository.com/)来搜索Spring，在结果中寻找Spring Context即可。我们要求的是[4.3.19 RELEASE版本](https://mvnrepository.com/artifact/org.springframework/spring-context/4.3.19.RELEASE)。复制如图代码，粘贴到`pom.xml`中的`<dependencies>`代码块中，点击右下角弹出的`Import`，稍等片刻即可完成导包。

    ![导包](https://img.zephyrl.co/images/2020/02/02/2c62852644f1f8b54.png)

2. 导入Spring整合单元测试包

    我们使用的是`spring-text`的[4.3.9 RELEASE版本](https://mvnrepository.com/artifact/org.springframework/spring-test/4.3.9.RELEASE)，重复上面的操作，完成导包。

3. 创建配置文件

    在我们之前创建好的`resources`文件夹内，新建一个关于Spring的配置文件`spring.xml`，复制如下基础核心配置文件，粘贴到里面。

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd ">
    </beans>
    ```

4. 创建测试类

    容器：存放类的实现对象。

    首先，在`java`文件夹下，创建包结构（比如我的`co.zephyrl`就先建立一个`com`包再建立一个`zephyrl`包）

    之后，创建类`Cat`，两个私有变量为`cname`和`color`，同时创建他们的`Getter`和`Setter`方法，还有`toString`方法。（右键 -> Generate -> 选择两个变量即可）。

    然后再单元测试文件夹`test`中，新建类`TestDemo1`，输入如下代码。注意添加注解`@Test`后会自动补全上方代码，然后点击左边绿色的运行按钮，会输出容器内默认值。

    ![单元测试](https://img.zephyrl.co/images/2020/02/02/39a3f44d7f4528a0c.png)

## Spring核心：控制反转+依赖注入（IOC+DI）

### IOC（Inversion of Control，控制反转）

   控制：控制对象的创建和销毁（生命周期）

   反转：以前创建对象由程序员完成，现在由spring来完成，底层可以理解为“工厂设计模式+反射+配置文件”实现程序解耦合；

## DI（Dependency injection，依赖注入）

   以前我们可以通过对象的`Get`方法进行参数的赋值，现在有了DI，我们可以在Spring容器内进行参数的赋值。

   所以。前提要有IOC

   组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系。容器全权负责的组件的装配，它会把符合依赖关系的对象通过JavaBean属性或者构造函数传递给需要的对象。

   比如我们之前的“小猫”的例子，将`TestDemo1.java`内的代码更改如下：

   ```java
   public class TestDemo1 {
    @Test
    public void test1() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        Cat cat = ac.getBean("cat", Cat.class);
        System.out.println(cat);
    }
   }
   ```

   我们使用DI中的set方法注入，类中必须有属性的set方法，必须有无参构造函数。此时需要更改Spring的配置文件`spring.xml`，在`<beans>`代码块中添加如下：

   ```xml
   <bean id="cat" class="com.zephyrl.Cat">
        <property name="cname" value="小白"/>
        <property name="color" value="白色"/>
    </bean>
   ```

   除了Set方法注入意外，还有以下方法可以进行DI：

   P名称空间注入：

   ```xml
   <bean id="cat" class="com.zephyrl.cat" p:name="小白" p:age="白色">
   </bean>
   ```

   其他的注入方法可以参考[这里](https://note.youdao.com/ynoteshare1/index.html?id=cd4bba03d078a599a1ed0f69c865d0bb&type=note) 

   此时返回单元测试模块，点击运行，结果如下：

   ![set方法注入](https://img.zephyrl.co/images/2020/02/02/41c1260fa10e2898f.png)

   相当容器在运行中不只实例化了Bean，而且给它赋值了。

## 总结

本节课学习了Spring的定义、发源、还有通过IOC+DI为例子的简单上手。
