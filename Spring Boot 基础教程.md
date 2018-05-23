# 工程配置

## chapter2-1-1：配置文件详解

如果不是特殊应用场景，就只需要在`application.properties`中完成一些属性配置就能开启各模块的应用。

### 自定义属性与加载

我们在使用Spring Boot的时候，通常也需要定义一些自己使用的属性，我们可以如下方式直接定义：

```properties
com.didispace.blog.name=程序猿DD
com.didispace.blog.title=Spring Boot教程
```

然后通过`@Value("${属性名}")`注解来加载对应的配置属性，具体如下：

```java
@Component
public class BlogProperties {

    @Value("${com.didispace.blog.name}")
    private String name;
    @Value("${com.didispace.blog.title}")
    private String title;

    // 省略getter和setter
}
```

按照惯例，通过单元测试来验证BlogProperties中的属性是否已经根据配置文件加载了。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ApplicationTests {

	@Autowired
	private BlogProperties blogProperties;


	@Test
	public void getHello() throws Exception {
		Assert.assertEquals(blogProperties.getName(), "程序猿DD");
		Assert.assertEquals(blogProperties.getTitle(), "Spring Boot教程");
	}

}
```

### 参数间的引用

在`application.properties`中的各个参数之间也可以直接引用来使用，就像下面的设置：

```properties
com.didispace.blog.name=程序猿DD
com.didispace.blog.title=Spring Boot教程
com.didispace.blog.desc=${com.didispace.blog.name}正在努力写《${com.didispace.blog.title}》
```

`com.didispace.blog.desc`参数引用了上文中定义的`name`和`title`属性，最后该属性的值就是`程序猿DD正在努力写《Spring Boot教程》`。

### 使用随机数

在一些情况下，有些参数我们需要希望它不是一个固定的值，比如密钥、服务端口等。Spring Boot的属性配置文件中可以通过`${random}`来产生int值、long值或者string字符串，来支持属性的随机值。

```properties
# 随机字符串
com.didispace.blog.value=${random.value}
# 随机int
com.didispace.blog.number=${random.int}
# 随机long
com.didispace.blog.bignumber=${random.long}
# 10以内的随机数
com.didispace.blog.test1=${random.int(10)}
# 10-20的随机数
com.didispace.blog.test2=${random.int[10,20]}
```

### 通过命令行设置属性值

`java -jar xxx.jar --server.port=8888`，通过使用–server.port属性来设置xxx.jar应用的端口为8888。

在命令行运行时，连续的两个减号`--`就是对`application.properties`中的属性值进行赋值的标识。所以，`java -jar xxx.jar --server.port=8888`命令，等价于我们在`application.properties`中添加属性`server.port=8888`。

Spring Boot也提供了屏蔽命令行访问属性的设置，只需要这句设置就能屏蔽：`SpringApplication.setAddCommandLineProperties(false)`。

### 多环境配置

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

- `application-dev.properties`：开发环境
- `application-test.properties`：测试环境
- `application-prod.properties`：生产环境

至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。

如：`spring.profiles.active=test`就会加载`application-test.properties`配置文件内容。

下面，以不同环境配置不同的服务端口为例，进行样例实验。

- 针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`
- 在这三个文件均都设置不同的`server.port`属性，如：dev环境设置为1111，test环境设置为2222，prod环境设置为3333
- application.properties中设置`spring.profiles.active=dev`，就是说默认以dev环境设置
- 测试不同配置的加载
  - 执行`java -jar xxx.jar`，可以观察到服务端口被设置为`1111`，也就是默认的开发环境（dev）
  - 执行`java -jar xxx.jar --spring.profiles.active=test`，可以观察到服务端口被设置为`2222`，也就是测试环境的配置（test）
  - 执行`java -jar xxx.jar --spring.profiles.active=prod`，可以观察到服务端口被设置为`3333`，也就是生产环境的配置（prod）

按照上面的实验，可以如下总结多环境的配置思路：

- `application.properties`中配置通用内容，并设置`spring.profiles.active=dev`，以开发环境为默认配置
- `application-{profile}.properties`中配置各个环境不同的内容
- 通过命令行方式去激活不同环境的配置

### Spring Boot构建RESTful API与单元测试

User实体类：

```java
public class User {
    private Long id;
    private String name;
    private Integer age;

    // 省略setter和getter 
}
```

实现对User对象的操作接口

```java
@RestController 
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下 
public class UserController { 
 
    // 创建线程安全的Map 
    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>()); 
 
    @RequestMapping(value="/", method=RequestMethod.GET) 
    public List<User> getUserList() { 
        // 处理"/users/"的GET请求，用来获取用户列表 
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递 
        List<User> r = new ArrayList<User>(users.values()); 
        return r; 
    } 
 
    @RequestMapping(value="/", method=RequestMethod.POST) 
    public String postUser(@ModelAttribute User user) { 
        // 处理"/users/"的POST请求，用来创建User 
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数 
        users.put(user.getId(), user); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.GET) 
    public User getUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息 
        // url中的id可通过@PathVariable绑定到函数的参数中 
        return users.get(id); 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.PUT) 
    public String putUser(@PathVariable Long id, @ModelAttribute User user) { 
        // 处理"/users/{id}"的PUT请求，用来更新User信息 
        User u = users.get(id); 
        u.setName(user.getName()); 
        u.setAge(user.getAge()); 
        users.put(id, u); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE) 
    public String deleteUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的DELETE请求，用来删除User 
        users.remove(id); 
        return "success"; 
    }  
}
```

下面针对该Controller编写测试用例验证正确性，具体如下。当然也可以通过浏览器插件等进行请求提交验证。

```java
import org.hamcrest.Matchers;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.RequestBuilder;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;


@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
public class UserControllerTest {

    private MockMvc mvc;

    @Before
    public void setUp() {
        mvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
    }

    @Test
    public void testUserController() throws Exception {
        RequestBuilder request = null;

        // 1、get查一下user列表，应该为空
        mvc.perform(MockMvcRequestBuilders.get("/users/"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string(Matchers.equalTo("[]")));

        // 2、post提交一个user
        request = MockMvcRequestBuilders.post("/users/")
                .param("id", "1")
                .param("name", "测试大师")
                .param("age", "20");
        mvc.perform(request)
                .andExpect(MockMvcResultMatchers.content().string(Matchers.equalTo("success")));

        // 3、get获取user列表，应该有刚才插入的数据
        request = MockMvcRequestBuilders.get("/users/");
        mvc.perform(request)
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string(Matchers.equalTo("[{\"id\":1,\"name\":\"测试大师\",\"age\":20}]")));
    }

}
```

# Web开发

## chapter3-1-6：统一异常处理

虽然，Spring Boot中实现了默认的error映射，但是在实际应用中，上面你的错误页面对用户来说并不够友好，我们通常需要去实现我们自己的异常提示。

- 创建全局异常处理类：通过使用`@ControllerAdvice`定义统一的异常处理类，而不是在每个Controller中逐个定义。`@ExceptionHandler`用来定义函数针对的异常类型，最后将Exception对象和请求URL映射到`error.html`中

  ```java
  @ControllerAdvice
  public class GlobalExceptionHandler {

      public static final String DEFAULT_ERROR_VIEW = "error";

      @ExceptionHandler(value = Exception.class)
      public String defaultErrorHandler(HttpServletRequest request,
                                              Exception e,
                                              Model model) throws Exception {
          ModelAndView mav = new ModelAndView();
          model.addAttribute("exception", e);
          model.addAttribute("url", request.getRequestURL());
          return DEFAULT_ERROR_VIEW;
      }
  }
  ```

- 实现`error.html`页面展示：在`templates`目录下创建`error.html`，将请求的URL和Exception对象的message输出。

  ```html
  <!DOCTYPE html>
  <html>
  <head lang="en">
      <meta charset="UTF-8" />
      <title>统一异常处理</title>
  </head>
  <body>
  <h1>Error Handler</h1>
  <div>${url}</div>
  </body>
  </html>
  ```

启动该应用，访问：`http://localhost:8080/hello`，可以看到如下错误提示页面。

![image.png-26.3kB][1]

*通过实现上述内容之后，我们只需要在Controller中抛出Exception，当然我们可能会有多种不同的Exception。然后在@ControllerAdvice类中，根据抛出的具体Exception类型匹配@ExceptionHandler中配置的异常类型来匹配错误映射和处理。*

# 数据访问

## chapter3-2-3：多数据源配置

在单数据源的情况下，Spring Boot的配置非常简单，只需要在`application.properties`文件中配置连接参数即可。但是往往随着业务量发展，我们通常会进行数据库拆分或是引入其他数据库，从而我们需要配置多个数据源，下面基于之前的JdbcTemplate和Spring-data-jpa例子分别介绍两种多数据源的配置方式。

​

创建一个Spring配置类，定义两个DataSource用来读取`application.properties`中的不同配置。如下例子中，主数据源配置为`spring.datasource.primary`开头的配置，第二数据源配置为`spring.datasource.secondary`开头的配置。

```java
@Configuration
public class DataSourceConfig {

    @Bean(name = "primaryDataSource")
    @Qualifier("primaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondaryDataSource")
    @Qualifier("secondaryDataSource")
    @Primary
    @ConfigurationProperties(prefix="spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

对应的`application.properties`配置如下：

```properties
spring.datasource.primary.url=jdbc:mysql://localhost:3306/test1
spring.datasource.primary.username=root
spring.datasource.primary.password=root
spring.datasource.primary.driver-class-name=com.mysql.jdbc.Driver

spring.datasource.secondary.url=jdbc:mysql://localhost:3306/test2
spring.datasource.secondary.username=root
spring.datasource.secondary.password=root
spring.datasource.secondary.driver-class-name=com.mysql.jdbc.Driver
```

### JdbcTemplate

对JdbcTemplate的支持比较简单，只需要为其注入对应的datasource即可，如下例子，在创建JdbcTemplate的时候分别注入名为`primaryDataSource`和`secondaryDataSource`的数据源来区分不同的JdbcTemplate。

```java
@Bean(name = "primaryJdbcTemplate")
public JdbcTemplate primaryJdbcTemplate(
        @Qualifier("primaryDataSource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}

@Bean(name = "secondaryJdbcTemplate")
public JdbcTemplate secondaryJdbcTemplate(
        @Qualifier("secondaryDataSource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}
```

接下来通过测试用例来演示如何使用这两个针对不同数据源的JdbcTemplate。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ApplicationTests {

	@Autowired
	@Qualifier("primaryJdbcTemplate")
	protected JdbcTemplate jdbcTemplate1;

	@Autowired
	@Qualifier("secondaryJdbcTemplate")
	protected JdbcTemplate jdbcTemplate2;

	//...
}
```

## chapter3-2-7 整合Mybatis

* pom中引入依赖

  ```xml
  <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.1.1</version>
  </dependency>

  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.21</version>
  </dependency>
  ```

* 在`application.properties`中配置mysql的连接配置

  ```properties
  spring.datasource.url=jdbc:mysql://localhost:3306/test
  spring.datasource.username=root
  spring.datasource.password=123456
  spring.datasource.driver-class-name=com.mysql.jdbc.Driver
  ```

### 使用Mybatis

#### 传参方式

1. 使用@Param

   ```java
   @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
   int insert(@Param("name") String name, @Param("age") Integer age);
   ```

   这种方式很好理解，`@Param`中定义的`name`对应了SQL中的`#{name}`，`age`对应了SQL中的`#{age}`。

2. 使用Map

   如下代码，通过Map对象来作为传递参数的容器：

   ```java
   @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name,jdbcType=VARCHAR}, #{age,jdbcType=INTEGER})")
   int insertByMap(Map<String, Object> map);
   ```

   对于Insert语句中需要的参数，我们只需要在map中填入同名的内容即可，具体如下面代码所示：

   ```java
   Map<String, Object> map = new HashMap<>();
   map.put("name", "CCC");
   map.put("age", 40);
   userMapper.insertByMap(map);
   ```

3. 使用对象

   除了Map对象，我们也可直接使用普通的Java对象来作为查询条件的传参，比如我们可以直接使用User对象:

   ```java
   @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
   int insertByUser(User user);
   ```

   这样语句中的`#{name}`、`#{age}`就分别对应了User对象中的`name`和`age`属性。

#### 返回结果的绑定

对于增、删、改操作相对变化较小。而对于“查”操作，我们往往需要进行多表关联，汇总计算等操作，那么对于查询的结果往往就不再是简单的实体对象了，往往需要返回一个与数据库实体不同的包装类，那么对于这类情况，就可以通过`@Results`和`@Result`注解来进行绑定，具体如下：

```java
@Results({
    @Result(property = "name", column = "name"),
    @Result(property = "age", column = "age")
})
@Select("SELECT name, age FROM user")
List<User> findAll();
```

在上面代码中，@Result中的property属性对应User对象中的成员名，column对应SELECT出的字段名。在该配置中故意没有查出id属性，只对User对应中的name和age对象做了映射配置，这样可以通过下面的单元测试来验证查出的id为null，而其他属性不为null：

# 日志管理

## chapter4-2-1：默认日志的配置

### 格式化日志

默认的日志输出如下：

```shell
2016-04-13 08:23:50.120  INFO 37397 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {4.3.11.Final}
```

输出内容元素具体如下：

- 时间日期 — 精确到毫秒
- 日志级别 — ERROR, WARN, INFO, DEBUG or TRACE
- 进程ID
- 分隔符 — `---` 标识实际日志的开始
- 线程名 — 方括号括起来（可能会截断控制台输出）
- Logger名 — 通常使用源代码的类名
- 日志内容

### 控制台输出

在Spring Boot中默认配置了`ERROR`、`WARN`和`INFO`级别的日志输出到控制台。

我们可以通过两种方式切换至`DEBUG`级别：

- 在运行命令后加入`--debug`标志，如：`$ java -jar myapp.jar --debug`
- 在`application.properties`中配置`debug=true`，该属性置为true的时候，核心Logger（包含嵌入式容器、hibernate、spring）会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

### 文件输出

Spring Boot默认配置只会输出到控制台，并不会记录到文件中，但是我们通常生产环境使用时都需要以文件方式记录。

若要增加文件输出，需要在`application.properties`中配置`logging.file`或`logging.path`属性。

- logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：`logging.file=my.log`
- logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：`logging.path=/var/log`

**日志文件会在10Mb大小的时候被截断，产生新的日志文件，默认级别为：ERROR、WARN、INFO**

### 级别控制

在Spring Boot中只需要在`application.properties`中进行配置完成日志记录的级别控制。

配置格式：`logging.level.*=LEVEL`

- `logging.level`：日志级别控制前缀，`*`为包名或Logger名
- `LEVEL`：选项TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

举例：

- `logging.level.com.didispace=DEBUG`：`com.didispace`包下所有class以DEBUG级别输出
- `logging.level.root=WARN`：root日志以WARN级别输出

### 自定义日志配置

由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：

- Logback：`logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, `logback.groovy`
- Log4j：`log4j-spring.properties`, `log4j-spring.xml`, `log4j.properties`, `log4j.xml`
- Log4j2：`log4j2-spring.xml`, `log4j2.xml`
- JDK (Java Util Logging)：`logging.properties`

**Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）**











[1]: http://static.zybuluo.com/vermouth9/gz3c2wns3oxazx7akaob5sdz/image.png

  ​

  ​