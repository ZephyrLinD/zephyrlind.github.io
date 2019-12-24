# SSM框架学习笔记（五） 面向切面开发（AOP）


## AOP 中的术语

- Joinpoint(连接点):所谓连接点是指那些被拦截到的点。在 spring 中,这些点指的是方法,因为 spring 只支持方法类型的连接点.
- Pointcut(切入点):所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义.
- Advice(通知/增强):所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知.通知分为前置通知,后置通知,异常通知,最终通知,环绕通知(切面要完成的功能)
- Introduction(引介):引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类
- 动态地添加一些方法或 Field.
- Target(目标对象):代理的目标对象
- Weaving(织入):是指把增强应用到目标对象来创建新的代理对象的过程.
- spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装在期织入
- Proxy（代理） :一个类被 AOP 织入增强后，就产生一个结果代理类
- Aspect(切面): 是切入点和通知（引介）的结合

## 需要导的 Jar 包

```xml
<!-- spring 的传统 AOP 的开发的包 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aop</artifactId>
  <version>4.3.19.RELEASE</version>
</dependency>
<!-- aspectJ 的开发包: -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>4.3.19.RELEASE</version>
</dependency>
```

## 引入 Spring 的配置文件，引入 AOP 约束

```xml
<!-- spring.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:aop="http://www.springframework.org/schema/aop"
xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```

## 编写目标类

### 编写接口

```java
package co.zephyrl.dao;

public interface UserDao {
    void find();
    void save();
    void update();
    int delete();
}
```

### 编写目标类对象

```java
package co.zephyrl.dao;

public class UserDaoImpl implements UserDao{
    @Override
    public void find() {
        System.out.println("查找用户");
    }

    @Override
    public void save() {
        System.out.println("保存");
    }

    @Override
    public void update() {
        System.out.println("更新");
    }

    @Override
    public int delete() {
        System.out.println("删除");
        return 1;
    }
}
```

## 目标类的配置

```xml
<!-- spring.xml -->
<!-- 这个bean填写目标类的信息 -->
<bean id="userDao" class="co.zephyrl.dao.UserDaoImpl"></bean>
```

## 通知类型

- 前置通知 ：在目标方法执行之前执行.
- 后置通知 ：在目标方法执行之后执行
- 环绕通知 ：在目标方法执行前和执行后执行
- 异常抛出通知：在目标方法执行出现异常的时候执行
- 最终通知 ：无论目标方法是否出现异常最终通知都会执行

## 切入点表达式 

`execution` 主要用在 spring.xml 配置文件中。

### `execution([expressions])`

- *：匹配任何数量字符；
- ..：匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。
- +：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。

### 表达式格式

|              表达式              |                                    意义                                     |
| :------------------------------: | :-------------------------------------------------------------------------: |
| public \* com.itdr.dao.\*.\*(..) |             任意返回值类型，dao包下的任意类的任意方法都会被增强             |
|    \* com.itdr.dao.\*.\*(..)     |       任意修饰符，任意返回值类型，dao包下的任意类的任意方法都会被增强       |
| \* com.itdr.dao.UserDao+.\*(..)  |                  表示当前类和它的子类的所有方法都会被增强                   |
|    \* com.itdr.dao..\*.\*(..)    | 任意修饰符，任意返回值类型，dao包及其子包下的下的任意类的任意方法都会被增强 |


## 编写一个切面类

```xml
<!-- 配置切面类 -->
<bean id="aopDemo" class="co.zephyrl.dao.AopDemo"></bean>

<!-- 进行AOP设置 -->
<aop:config>
    <aop:pointcut id="p1" expression="execution(* co.zephyrl.dao.UserDao.save(..))"  />
    <aop:pointcut id="p2" expression="execution(* co.zephyrl.dao.UserDao.delete(..))"/>
    <aop:pointcut id="p3" expression="execution(* co.zephyrl.dao.UserDao.update(..))"/>
    <!--配置切面-->
    <aop:aspect ref="aopDemo">
        <!-- 配置前置通知指向切面中的方法，切入点指向p1 -->
        <aop:before method="before" pointcut-ref="p1" />
        <!-- 配置后置通知指向切面中的删除方法，切入点指向p2 -->
        <aop:after-returning method="af" pointcut-ref="p2" returning="num" />
        <!-- 配置环绕通知指向切面中的修改方法，切入点指向p3 -->
        <aop:around method="ar" pointcut-ref="p3" />
        <!-- 配置异常通知指向切面中的修改方法，切入点指向p3 -->
        <aop:after-throwing method="th" pointcut-ref="p3" throwing="thr" />
        <!-- 配置最终通知指向切面中的修改方法，切入点指向p3 -->
        <aop:after method="zz" pointcut-ref="p3" />
    </aop:aspect>
</aop:config>
```

## 测试

```java
public class TestDemo {
    @Test
    public void demo1() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        UserDao userDao = ac.getBean("userDao", UserDao.class);
        userDao.find();
        userDao.save();
        userDao.delete();
        userDao.update();
    }
}
```

## AOP 底层原理（了解）

AOP的底层原理是**动态代理**技术，而动态代理技术的底层原理是是**反射**

下面我们来使用 JDK 动态代理增强一个类中方法：

```java
public class MyJDKProxy implements InvocationHandler {
    private UserDao userDao;
    public MyJDKProxy(UserDao userDao) {
        this.userDao = userDao;
    }
    // 编写工具方法：生成代理
    public UserDao createProxy(){
        UserDao userDaoProxy = (UserDao)
        // 获取字节码
        Proxy.newProxyInstance(userDao.getClass().getClassLoader(),
        userDao.getClass().getInterfaces(), this);
        return userDaoProxy;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if("save".equals(method.getName())){
            System.out.println("权限校验================");
        }
        return method.invoke(userDao, args);
    }
}
```

进行单元测试：

```java
@Test
public void test() {
    UserDao ud = new JDBCUserDaoImpl();
    ud.save();
    // 代理
    UserDao udProxy = new JDKProxy(ud).createProxy;
    udProxy.save();
}
```

除此之外我们还可以用 Cglib 动态代理增强一个类中的方法:

```java
public class MyCglibProxy implements MethodInterceptor{
    private CustomerDao customerDao;
    public MyCglibProxy(CustomerDao customerDao){
        this.customerDao = customerDao;
    }
    // 生成代理的方法:
    public CustomerDao createProxy(){
        // 创建 Cglib 的核心类:
        Enhancer enhancer = new Enhancer();
        // 设置父类:
        enhancer.setSuperclass(CustomerDao.class);
        // 设置回调:
        enhancer.setCallback(this);
        // 生成代理：
        CustomerDao customerDaoProxy = (CustomerDao) enhancer.create();
        return customerDaoProxy;
    }
    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        if("delete".equals(method.getName())){
            Object obj = methodProxy.invokeSuper(proxy, args);
            System.out.println("日志记录================");
            return obj;
        }
        return methodProxy.invokeSuper(proxy, args);
    }
}
```
