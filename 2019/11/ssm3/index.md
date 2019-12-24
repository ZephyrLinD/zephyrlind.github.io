# SSM框架学习笔记（三） Bean的详解


## 命名

`id`：给Bean起个名字。使用了约束中的唯一约束。里面不能出现特殊字符的。

`name`：也是给Bean起个名字。没有使用约束中的唯一约束（理论上可以出现重复的，但是实际开发不能出现的）。里面可以出现特殊字符。如果Bean没有`id`的话，也可当作id使用。

具体用在如下位置：

```xml
<bean id="people" name="fuck" class="co.zephyrl.people"></bean>
```

## 生命周期

init-method: Bean被初始化的时候执行的方法

destroy-method: Bean被销毁的时候执行的方法（Bean是单例创建，工厂关闭）

1. 在bean的配置上进行配置：

    init method="执行方法的名称"

    destroy-method="执行方法的名称"

    在配置文件`Spring.xml`中添加如下内容：

   ```xml
   <!-- spring.xml -->
   <bean id="people" class="co.zephyrl.people" init-method="onInit"   destroy-method="onDestroy">
   ```

   在Bean体内和单元测试中的写法如下：

   ```java
   // People.java
   public void onInit() {
       System.out.println("生命周期开始");
   }
   public void onDestroy() {
       System.out.println("生命周期结束");
   }

   // test.java
   @Test
   public void test1() {
       ApplicationContext ac = new ClassXmlApplicationContext("spring.xml");
       People people = ac.getBean("people", People.class);
       ((ClassPathXmlApplicationContext) ac).close();
   }
   ```

2. 类实现接口

   分别实现 `InitializingBean`、`DisposableBean` 的接口。

   `Spring.xml` 代码如下：

   ```xml
   <bean id="people" class="co.zephyrl.people"></bean>
   ```

   Bean改成如下代码：

   ```java
   public class People implements InitializingBean, DisposableBean {
       // 省略其他
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("实现接口的销毁方法");
       }

       @Override
       public void destroy() throws Exception {
           System.out.println("实现接口的初始化方法");
       }
   }
   ```

   再次执行测试类 `TestDemo1` 中的 `test1()` 方法，即可发现效果。

   **注意：如果以上两个方法同时执行，会先执行类实现接口。**

   下面这种方式，在类上进行注解时才可使用：

   ```java
   @PostConstruct
    public void onInit2() {
        System.out.println("初始化");
    }

   @PreDestroy
    public void onDestroy2() {
        System.out.println("销毁");
    }
   ```

   之后删除继承的`InitializingBean`、`DisposableBean`类，执行测试类 `TestDemo1` 中的 `test1()` 方法即可。

3. 全局默认初始化和方法的配置（方法可有可无）

   在 `spring.xml` 中的 `<Beans>` 标签中加入 `default-init-method=“”` 和`default-destroy-method=“”` 即可实现全局默认初始化方法和销毁方法的配置。

## 作用范围

在 `Spring.xml` 中的 `<Bean>` 标签里的 `scope` 实现，有以下几种：

- singleton：默认的，Spring会采用单例模式创建这个对象。
- prototype：多例模式。（Struts2和Spring整合一定会用到）
- request：应用在web项目中，Spring创建这个类以后，将这个类存入到request范围中。
- session：应用在web项目中，Spring创建这个类以后，将这个类存入到session范围中。
- globalsession：应用在web项目中，必须在porlet环境下使用。但是如果没有这种环境，相对于session。

尝试方法：可以创建两个同样的对象，直接输出对象的地址

```java
// TestDemo1
public void test2() {
    Applicationcontext ac = new ClassXmlApplicationContext("spring.xml");
    People p1 = ac.getBean("people", People.class);
    People p2 = ac.getBean("people", People.class);
    System.out.println(p1);
    System.out.println(p2);
    System.out.println(p1 == p2);
}
```

## 懒加载

当你需要的时候加载，不需要的时候不加载。

什么时候用，什么时候实例化。

- 好处：节省资源空间
- 可能导致某些操作响应时间增加

单个bean，`lazy-init=true`，在获取Bean的时候加载。

整个容器，`default-lazy-init=true`。

## Bean 的继承

1. 有些情况下，类与类之间因为继承关系，会继承父类的属性，这时候可以在配置bean时使用简便的方式注入：abstract表示这个bean不需要实例化
2. 没有继承时，两个类仍然有共有属性，可以直接设置一个bean

## Bean 的别名

仅举例，不做赘述：

- xml形式

   ```xml
   <bean id="people" name="pep, people2" class="co.zephyrl.people" />
   <alias name="people" alias="people1" />
   ```

- 注解形式

   ```java
   @Configuration
   public class BeanConfiguration {
       @Bean("Bean1", "Bean2")
       public Bean bean() {
           return Bean = new Bean();
       }
   }
   ```
