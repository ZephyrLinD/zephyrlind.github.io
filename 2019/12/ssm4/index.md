# SSM框架学习笔记（四） 整合JDBC


## Spring 的 JDBC 模板

Spring 是 EE 开发的一站式框架，有 EE 开发的每层解决方案。Spring 对持久层也提供了解决方案：ORM 模板和 JDBC 的模板。

Spring 提供了很多的模板用以简化开发：

|    ORM持久化技术    |                        模板类                        |
| :---------------: | :--------------------------------------------------: |
|       JDBC        |      org.springfranework.jdbc.core.JdbcTemplate      |
|   Hibernate3.0    | org.springfranework.orm.hibernate3.HibernateTemplate |
| IBatics(MyBatics) | org.springfranework.orm.ibatis.SqlMapClientTemplate  |
|        JPA        |       org.springfranework.orm.jpa.JpaTemplate        |

## JDBC 模板使用入门

创建项目，引入 Jar 包 -> 引入基本开发包 -> 数据库驱动

Spring 的 jdbc 模板的 jar 包

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.19.RELEASE</version>
</dependency>
```

MySQL 数据库链接的 jar 包

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<!-- 视MySQL版本而定 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.18</version>
</dependency>
```

## 创建数据库和表

## 使用 JDBC 的模板：数据增删改查

```java
@Test
public void test1() {
    // 创建一个连接池
    DriverManagerDataSource dmd = new DriverManagerDataSource();
    dmd.setDriverClassName("com.mysql.jdbc.Driver");
    dmd.setUrl("jdbc:mysql://localhost:3306/LZA_SSM");
    dmd.setUsername("root");
    dmd.setPassword("123456");

    // 创建JDBC模板，这里需要一个连接池
    JdbcTemplate jdt = new JdbcTemplate(dmd);

    // 测试
    String sql = "select * from users";
    List<Map<String, Object>> li = jdt.queryForList(sql);
    for (Map<String, Object> map : li) {
        for (String s : map.keySet()) {
            System.out.println("列名: " + s + map.get(s));
        }
    }
}
```

## 将连接池和模板交给 Spring 来管理

### 引入 Spring 的配置文件

```xml
<!-- spring.xml 省略其他约束文件-->

<!-- 数据库连接池 -->
<bean id="managerDataSource"class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://LZA_SSM" />
    <property name="username" value="root" />
    <property name="password" value="123456" />
 </bean>

<!-- JDBC模板 -->
<bean id="jdbcTemplate"class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="managerDataSource" />
</bean>
```

### 使用 JDBC 模板

```java
@Test
public void test2() {
    // 给Spring处理
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdt = ac.getBean("jdbcTemplate", JdbcTemplate.class);

    // 测试同上，不做赘述
}
```

## 抽取配置到属性文件

### 首先在 `spring.xml` 的 `beans` 标签中添加两条约束

```xml
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-contest-4.2.xsd
```

### 然后在 `resources` 文件夹中创建文件 `db.properties`

内容如下：

```xml
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/LZA_SSM
jdbc.userName=root
jdbc.passWord=123456
```

### 之后，修改 `spring.xml`

```xml
<bean id="managerDataSource"class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${jdbc.dirverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.userName}" />
    <property name="password" value="${jdbc.passWord}" />
 </bean>
 ```

即可完成，和上面的意思其实一样，只不过是修改属性来修改配置文件。

## 使用 JDBC 的模板完成 CRUD 的操作

首先，表结构大概如下：

```shell
mysql> select * from users;
+----+------+-----------+-----+---------------------+------+
| id | ages | uname     | psd | cdata               | msg  |
+----+------+-----------+-----+---------------------+------+
|  1 |   10 | zhangxin1 | 123 | 2019-12-19 00:00:00 | NULL |
|  2 |   10 | zhangxin2 | 123 | 2019-12-18 00:00:00 | NULL |
|  3 |   10 | zhangxin3 | 123 | 2019-12-19 00:00:00 | NULL |
|  4 |   10 | zhangxin4 | 123 | 2019-12-19 14:13:51 | NULL |
|  5 |   10 | zx2       | 123 | 2019-12-19 14:14:15 | cs   |
|  6 |   10 | zx0       | 123 | 2019-12-19 14:14:28 | cs   |
|  7 |   10 | zx1       | 123 | 2019-12-19 14:14:52 | cs   |
|  8 |   10 | zx2       | 123 | 2019-12-19 14:15:08 | cs   |
+----+------+-----------+-----+---------------------+------+
```

### 插入、删除操作

1. 插入操作如下：

    ```java
    @Test
    public void saveTest() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date d = new Date();
        String nowDate = sdf.format(d);
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        jdt.update("Insert into users values(null, ?, ?, ?, ?, ?)", "9", "20",    "lin", "123", nowDate, "se");
    }
    ```

2. 删除操作如下：

    ```java
    public void deleteTest() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        jdt.update("Delete from users where id = ?", 12);
    }
    ```

### 查询相关操作

1. 查询某个属性

    ```java
    @Test
    public void queryTest1() {
        // 查询操作
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        String name = jdt.queryForObject("select uname from users where id = ?", String.class, "9");
        System.out.println("Name from query is: " + name);

        // 统计查询
        int count = jdt.queryForObject("Select count(*) from users", int.class);
        System.out.println("The count of query is: " + count);
    }
    ```

2. 查询返回对象或集合

    ```java
    // MyRowmapper.Java
    public class MyRowMapper implements RowMapper<Users> {
        @Override
        public Users mapRow(ResultSet resultSet, int rowNumber) throws SQLException {
            Users us = new Users();
            us.setId(resultSet.getInt("id"));
            us.setAges(resultSet.getInt("ages"));
            us.setCdata(resultSet.getString("cdata"));
            us.setUname(resultSet.getString("uname"));
            us.setMsg(resultSet.getString("msg"));
            us.setPsd(resultSet.getInt("psd"));
            return us;
        }
    }

    // TestDemo.java
    @Test
    public void  queryTest2_1() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        // 封装在一个对象中
        Users usr = jdt.queryForObject("Select * from users where id = ?", new MyRowMapper(), "10");
        System.out.println(usr);
    }

    @Test
    public void queryTest2_2() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdt = ac.getBean("jdbcTemplate", JdbcTemplate.class);
        // 查询多条记录
        List<Users> list = jdt.query("Select * from users", new MyRowMapper());
        for(Users u : list) {
            System.out.println(u);
        }
    }
    ```
