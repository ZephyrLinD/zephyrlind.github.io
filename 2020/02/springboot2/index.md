# SpringBoot学习笔记（二） —— 配置文件


## 配置文件

SpringBoot使用一个全局的配置文件，配置文件名称是固定的，可以是 `application.properties`：

```properties
# application.properties
# key = value
# 比如...
server.port=8080
```

或者 `application.yaml`

```yaml
# application.yaml
# key: value (key冒号空格value)
# 比如...
server:
    port: 8080
```

配置文件可以修改 SpringBoot 自动配置的默认值

## YAML

### 基础语法

YAML 用 `key:(space)value` 来表示一对键值对；用空格的缩进来表示层级关系，只要左边对齐的一列数据都是同一等级的。

注：`yaml` 大小写敏感

```yaml
server:
    port: 8081
    path: /hello
```

在此引入新的名词，字面量。字面量指的是数字、布尔值、字符串。

字面量在 yaml 中可以直接写在后面，不用加任何双引号或者单引号。

“” 双引号，不会转义字符串里面的特殊字符 ， 特殊字符会作为本身想表示的意思；

比如 ： name: "kuang \n shen"   输出 ： kuang  换行   shen

'' 单引号，会转义特殊字符 ， 特殊字符最终会变成和普通字符一样输出

比如 ： name: ‘kuang \n shen’   输出 ： kuang  \n   shen

（狂神牛逼）

### 对象、Map

```yaml
k: 
    v1:
    v2:
```

在下一行来写对象的属性和值得关系，注意缩进；比如：

```yaml
student:
    name: zephyr
    age: 20
```

行内写法：

```yaml
student: {name: zephyr, age: 20}
```

### 数组（list、set）

用 - 值表示数组中的一个元素,比如：

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```

## 注入配置文件

1、导入配置文件处理器

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

2、在 `resources` 中删除 `application.properties`，新建 `application.yaml`

```yaml
person:
    name: zephyr
    age: 20
    happy: false
    birth: 2000/01/01
    maps: {k1: v1,k2: v2}
    lists:
      - code
      - girl
      - music
    dog:
      name: 旺财
      age: 1
```

3、编写主类

```java
/*
@ConfigurationProperties作用：
将配置文件中配置的每一个属性的值，映射到这个组件中；
告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
参数 prefix = “person” : 将配置文件中的person下面的所有属性一一对应

只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能
*/

// Person.java

@Component //注册bean
@ConfigurationProperties(prefix = "person")
public class Person {
    @Component // 注册 Bean
    @ConfigurationProperties(prefix = "person") // 用application.yaml中的person注入
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;

    //getter,setter
    //toString

}
```

```java
public class Dog {
    private String name;
    private Integer age;

    //get、set方法
    //toString()方法  
}
```

4、编写测试类并运行检测

```java
@SpringBootTest
public class SpringbootDemo03ApplicationTests {

    @Autowired
    Person person = new Person();

    @Test
    public void contextLoads() {
        System.out.println(person);
    }

}
```

运行结果如下

![注入成功](https://img.zephyrl.co/images/2020/02/26/1_runsuccess.png)

## 其他注入方法

1、除了yml还有我们之前常用的 properties，properties 配置文件在中文的时候会有乱码，wine不需要去 IDEA 设置中设置编码格式为 UTF-8

![endocing](https://img.zephyrl.co/images/2020/02/26/2_encodings.png)

2、我们使用的是@configurationProperties的方式，还有一种方式是使用@value

```java
@Component //注册bean
public class Person {
    //直接使用@value
    @Value("${person.name}") //从配置文件中取值
    private String name;
    @Value("#{11*2}")  //#{SPEL} Spring表达式
    private Integer age;
    @Value("true")  // 字面量
    private Boolean happy;
    。。。。。。  
}
```

这个使用起来并不友好！我们需要为每个属性单独注解赋值，比较麻烦；我们来看个功能对比图

|                      | @ConfigurationProperties |   @Value   |
| :------------------: | :----------------------: | :--------: |
|         功能         | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） |           支持           |   不支持   |
|         SpEL         |          不支持          |    支持    |
|   JSR303 数据校验    |           支持           |   不支持   |
|     复杂类型封装     |           支持           |   不支持   |

- `@ConfigurationProperties` 只需要写一次即可，`@Value` 则需要每个字段都添加
- 松散绑定：这个什么意思呢? 比如我的 yml 中写的 last-name，这个和 lastName 是一样的， - 后面跟着的字母默认是大写的。这就是松散绑定
- JSR303数据校验，这个就是我们可以在字段是增加一层过滤器验证，可以保证数据的合法性
- 复杂类型封装，yml 中可以封装对象 ， 使用 `@Value`就不支持

## JSR303 数据校验

可以用 `@Validated`来校验数据，如果数据异常则会统一抛出异常，方便异常中心统一处理。我们这里来写个注解让我们的 name 只能支持 Email 格式

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @Email()
    private String name;
    private Integer age;
    private Boolean happy;
    private Date Birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Cat cat;
    // Constructor
    // Getter and Setter
}
```

此时会报错：

![error](https://img.zephyrl.co/images/2020/02/26/3_jsr303errord20dc6270a79e4dd.png)

## 写在最后

配置文件还有很多其他的小知识点，到参考文章可以看一看

参考文章：https://www.cnblogs.com/hellokuangshen/p/11259029.html
