# SSM框架学习笔记（六） 事务


## 事务的回顾

### 什么是事务

- 事务：逻辑上的一组操作，组成这组操作的各个单元，要么全都成功，要么全都失败。

### 事务的特性

- 原子性：事务不可分割
- 一致性：事务执行前后数据完整性保持一致
- 隔离性：一个事务的执行不应该受到其他事务的干扰
- 持久性：一旦事务结束，数据就持久化到数据库

### 如果不考虑隔离性引发安全性问题

1. 读问题

     脏读 ：一个事务读到另一个事务未提交的数据

     不可重复读 ：一个事务读到另一个事务已经提交的update的数据，导致一个事务中多次查询结果不一致

     虚读、幻读 ：一个事务读到另一个事务已经提交的insert的数据，导致一个事务中多次查询结果不一致。

2. 写问题

    丢失更新

### 解决读问题 —— 设置事务的隔离级别

- Read uncommitted ：未提交读，任何读问题解决不了。
- Read committed ：已提交读，解决脏读，但是不可重复读和虚读有可能发生。
- Repeatable read ：重复读，解决脏读和不可重复读，但是虚读有可能发生。
- Serializable ：解决所有读问题（会降低数据库查询效率）。

### 解决写问题

- 悲观锁 —— 认为**每次都失败**，所以在 SQL 语句的最后加上关键字 `for update`，也就是每次存数据的时候更新一下数据库，保证都能存进去。

- 乐观锁 —— 当可能丢失的时候再加锁。

这两者根本的不同就是悲观锁是数据库提供的工具，乐观锁是程序员自己做的的版本控制。

给乐观锁举个例子，每次上传的版本号都会更新，如果发现版本号异常，就不进行数据更新，即可形成一个乐观锁。

## Spring 的事务管理的 API

### PlatformTransactionManager：平台事务管理器

平台事务管理器：接口，是Spring用于管理事务的真正的对象。

DataSourceTransactionManager ：底层使用JDBC管理事务

HibernateTransactionManager ：底层使用Hibernate管理事务

### TransactionDefinition ：事务定义信息

事务定义：用于定义事务的相关的信息，隔离级别、超时信息、**传播行为**、是否只读

### TransactionStatus：事务的状态

事务状态：用于记录在事务管理过程中，事务的状态的对象。

## 事务管理的API的关系

Spring 进行事务管理的时候，首先平台事务管理器根据事务定义信息进行事务的管理，在事务管理过程中，产生各种状态，将这些状态的信息记录到事务状态的对象中。

## Spring 的事务的传播行为

Spring中提供了**七种**事务的传播行为，其中还分为三类：

1. 保证多个操作在同一个事务中

    **PROPAGATION_REQUIRED ：默认值，如果A中有事务，使用A中的事务，如果A没有，创建一个新的事务，将操作包含进来**

    PROPAGATION_SUPPORTS ：支持事务，如果A中有事务，使用A中的事务。如果A没有事务，不使用事务。

    PROPAGATION_MANDATORY ：如果A中有事务，使用A中的事务。如果A没有事务，抛出异常。

2. 保证多个操作不在同一个事务中

    PROPAGATION_REQUIRES_NEW ：如果A中有事务，将A的事务挂起（暂停），创建新事务，只包含自身操作。如果A中没有事务，创建一个新事务，包含自身操作。

    PROPAGATION_NOT_SUPPORTED ：如果A中有事务，将A的事务挂起。不使用事务管理。

    PROPAGATION_NEVER ：如果A中有事务，报异常。

3. 嵌套式事务

    PROPAGATION_NESTED ：嵌套事务，如果A中有事务，按照A的事务执行，执行完成后，设置一个保存点，执行B中的操作，如果没有异常，执行通过，如果有异常，可以选择回滚到最初始位置，也可以回滚到保存点。

## Spring 的事务管理（银行转账小案例）

场景：A给B转账，需要A的账户，B的账户已经转账金额，当A账户扣钱成功且B账户加钱成功，说明转账成功，其余情况转账失败。只有成功的情况下才会将数据写入数据库。

我们首先写一个正确的业务：

```java
// UserService.java 接口
public interface UserService {
    void aTob(String aname, String bname, int money);
}

// UserServiceImpl.java 接口实现类
public class UserServiceImpl implements UserService {
    // 基础银行转账案例正确代码
    @Override
    public void aTob(String aname, String bname, int money) {
        UserDao userDao = new UserDao();
        // 参数校验
        if (StringUtils.isEmpty(aname) || StringUtils.isEmpty(bname) || money <= 0) {
            System.out.println("参数状态非法");
        }
        // A扣钱
        int i = userDao.updateByUnameAndMoneySub(aname, money);
        // B加钱
        int i1 = userDao.updateByUnameAndMoneyAdd(bname, money);

        if (i > 0 && i1 > 0) {
            System.out.println("转账成功");
        } else {
            System.out.println("转账失败");
        }
    }
}

// UserDao.java
public class UserDao {
    // 根据用户名扣除相应金额
    public int updateByUnameAndMoneySub(String aname, int money) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdbcTemplate = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        String sql = "UPDATE bank SET cdata = now(), money = money - ? WHERE uname = ?";
        int linIsSuccessful = jdbcTemplate.update(sql, money, aname);
        return linIsSuccessful;
    }
    // 根据用户名增加金额
    public int updateByUnameAndMoneyAdd(String bname, int money) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdbcTemplate = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        String sql = "UPDATE bank SET cdata = now(), money = money + ? WHERE uname = ?";
        int linIsSuccessful = jdbcTemplate.update(sql, money, bname);
        return linIsSuccessful;
    }
}
// spring.xml省略，只注入了jdbc
```

接下来，我们在 `UserServiceImpl.java` 的如下位置添加代码，设置一个人为的零除数异常：

```java
// A扣钱
int i = userDao.updateByUnameAndMoneySub(aname, money);
// 人为异常
int a = 1 / 0;
// B加钱
int i1 = userDao.updateByUnameAndMoneyAdd(bname, money);
```

此时，A扣了钱但是B没有收到转账。此时我们就需要引入**事务管理**

## 事务管理

### 编程式事务管理

引入新的约束：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-contest-4.2.xsd
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">
```

主要是关于tx的引入。

第一步：配置平台事务管理器

```xml
<bean id="manager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="managerDataSource" />
</bean>
```

第二步：配置事务管理的模板类（可以简化事务管理器的使用）

```xml
<bean id="template" class="org.springframework.transaction.support.TransactionTemplate">
    <property name="transactionManager" ref="manager" />
</bean>
```

第三步：在业务层注入事务管理的模板

```xml
<!--  业务层  -->
<bean id="userService" class="co.zephyrl.service.UserServiceImpl">
    <property name="userDao" ref="userDao" />
    <property name="template" ref="template" />
</bean>
<!--  数据层  -->
<bean id="userDao" class="co.zephyrl.dao.UserDao">
    <property name="jdbcTemplate" ref="jdbcTemplate" />
    <property name="template" ref="template" />
</bean>
```

第四步：编写代码

```java
// UserServiceImpl.java

// 数据层注入
private UserDao userDao;
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}

// 事务管理模板注入
private TransactionTemplate template;
public void setTemplate(TransactionTemplate template) {
    this.template = template;
}

@Override
public void aTob2(String aName, String bName, int money) {
    template.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
            transfer(aName, bName, money);
        }
    });
}
```

第五步：编写测试类

```java
// TestDemo.java
@Test
public void test3() {
    // 准备测试数据
    String aname = "zhangxin1";
    String bname = "Lin";
    int money = 10;

    // 根据配置文件创建spring容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");

    // 创建业务层对象
    UserService userService = ac.getBean("userService", UserService.class);
    userService.aTob2(aname, bname, money);
}
```

此时，异常没有解决的情况下，事务会自动回滚防止数据脏读

### 声明式事务管理（通过配置实现）-- AOP

第一步：引入约束（context）

第二部：开启注解扫描

```xml
<!--  开启注解扫描   -->
<context:component-scan base-package="co.zephyrl"></context:component-scan>
```

第三步：配置事务管理器

```xml
<!-- Spring平台事务管理器 -->
<bean id="manager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="managerDataSource" />
</bean>
```

第四步：开启注解事务

```xml
<!--  开启注解事务  -->
<tx:annotation-driven transaction-manager="manager" ></tx:annotation-driven>
```

第五步：在各个层添加注解

![业务层注解](../../../blogimages/ssm6/q.png)
