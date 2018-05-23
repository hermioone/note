# Spring Boot文档

## Ⅳ. Spring Boot特性

### 23. SpringApplication

SpirngApplication类提供了一种快捷方式，用于从main()方法启动Spring应用，多数情况下， 你只需要将该任务委托给`SpringApplication.run`静态方法：

```java
public static void main(String[] args){
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

当应用启动时，你应该会看到类似下面的东西：

```
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v1.4.1.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatEmbeddedServletContainerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下会显示INFO级别的日志信息，包括一些相关的启动详情，比如启动应用的用户等。

#### 23.1 启动失败

如果应用启动失败，注册的`FailureAnalyzers`就有机会提供一个特定的错误信息，及具体的解决该问题的动作。例如，如果在`8080`端口启动一个web应用，而该端口已被占用，那你应该可以看到类似如下的内容：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

**注** Spring Boot提供很多的`FailureAnalyzer`实现，你[自己实现](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#howto-failure-analyzer)也很容易。

如果没有可用于处理该异常的失败分析器（failure analyzers），你需要展示完整的auto-configuration报告以便更好的查看出问题的地方，因此你需要启用`org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer`的[debug](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-external-config)属性，或开启[DEBUG日志级别](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-custom-log-levels)。

例如，使用`java -jar`运行应用时，你可以通过如下命令启用`debug`属性：

```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

#### 23.2 自定义Banner

通过在classpath下添加一个`banner.txt`或设置`banner.location`来指定相应的文件可以改变启动过程中打印的banner。如果这个文件有特殊的编码，你可以使用`banner.encoding`设置它（默认为UTF-8）。除了文本文件，你也可以添加一个`banner.gif`，`banner.jpg`或`banner.png`图片，或设置`banner.image.location`属性。图片会转换为字符画（ASCII art）形式，并在所有文本banner上方显示。

在banner.txt中可以使用如下占位符：

| 变量                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ${application.version}                                       | MANIFEST.MF中声明的应用版本号，例如`Implementation-Version: 1.0`会打印`1.0` |
| ${application.formatted-version}                             | MANIFEST.MF中声明的被格式化后的应用版本号（被括号包裹且以v作为前缀），用于显示，例如(`v1.0`) |
| ${spring-boot.version}                                       | 当前Spring Boot的版本号，例如`1.4.1.RELEASE`                 |
| ${spring-boot.formatted-version}                             | 当前Spring Boot被格式化后的版本号（被括号包裹且以v作为前缀）, 用于显示，例如(`v1.4.1.RELEASE`) |
| ${Ansi.NAME}（或${AnsiColor.NAME}，${AnsiBackground.NAME}, ${AnsiStyle.NAME}） | NAME代表一种ANSI编码，具体详情查看[AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java) |
| ${application.title}                                         | `MANIFEST.MF`中声明的应用title，例如`Implementation-Title: MyApp`会打印`MyApp` |

**注** 如果想以编程的方式产生一个banner，可以使用`SpringBootApplication.setBanner(…)`方法，并实现`org.springframework.boot.Banner`接口的`printBanner()`方法。

你也可以使用`spring.main.banner-mode`属性决定将banner打印到何处，`System.out`（`console`），配置的logger（`log`）或都不输出（`off`)。

打印的banner将注册成一个名为`springBootBanner`的单例bean。

**注** YAML会将`off`映射为`false`，如果想在应用中禁用banner，你需要确保`off`添加了括号：

```
spring:
    main:
        banner-mode: "off"
```

#### 23.3 自定义SpringApplication

如果默认的`SpringApplication`不符合你的口味，你可以创建一个本地实例并对它进行自定义。例如，想要关闭banner你可以这样写：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

**注**：传递给`SpringApplication`的构造器参数将作为spring beans的配置源，多数情况下，它们是一些`@Configuration`类的引用，但也可能是XML配置或要扫描包的引用。

你也可以使用`application.properties`文件来配置`SpringApplication`，具体参考24. Externalized配置，访问SpringApplication Javadoc可获取完整的配置选项列表。

#### 23.4 流式构建API

如果需要创建一个分层的`ApplicationContext`（多个具有父子关系的上下文），或只是喜欢使用流式（fluent）构建API，那你可以使用SpringApplicationBuilder。 SpringApplicationBuilder允许你以链式方式调用多个方法，包括parent和child方法，这样就可以创建多层次结构，例如：

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

**注**：创建ApplicationContext层次时有些限制，比如，Web组件必须包含在子上下文中，并且父上下文和子上下文使用相同的Environment，具体参考SpringApplicationBuilder javadoc。

#### 23.5 Application事件和监听器

除了常见的Spring框架事件，比如ContextRefresheedEvent，`SpringApplication`也会发送其他的application事件。

**注** 有些事件实际上是在`ApplicationContext`创建前触发的，所以你不能在那些事件（处理类）中通过`@Bean`注册监听器，只能通过`SpringApplication.addListeners(…)`或`SpringApplicationBuilder.listeners(…)`方法注册。如果想让监听器自动注册，而不关心应用的创建方式，你可以在工程中添加一个`META-INF/spring.factories`文件，并使用`org.springframework.context.ApplicationListener`作为key指向那些监听器，如下：

```
org.springframework.context.ApplicationListener=com.example.project.MyListener
```

应用运行时，事件会以下面的次序发送：

1. 在运行开始，但除了监听器注册和初始化以外的任何处理之前，会发送一个`ApplicationStartedEvent`。
2. 在Environment将被用于已知的上下文，但在上下文被创建前，会发送一个`ApplicationEnvironmentPreparedEvent`。
3. 在refresh开始前，但在bean定义已被加载后，会发送一个`ApplicationPreparedEvent`。
4. 在refresh之后，相关的回调处理完，会发送一个`ApplicationReadyEvent`，表示应用准备好接收请求了。
5. 启动过程中如果出现异常，会发送一个`ApplicationFailedEvent`。

**注** 通常不需要使用application事件，但知道它们的存在是有用的（在某些场合可能会使用到），比如，在Spring Boot内部会使用事件处理各种任务。

#### 23.6 Web环境

`SpringApplication`将尝试为你创建正确类型的`ApplicationContext`，默认情况下，根据你开发的是否为web应用决定使用`AnnotationConfigApplicationContext`或`AnnotationConfigEmbeddedWebApplicationContext`。

用于确定是否为web环境的算法相当简单（判断是否存在某些类），你可以使用`setWebEnvironment(boolean webEnvironment)`覆盖默认行为。

通过调用`setApplicationContextClass(…)`，你可以完全控制`ApplicationContext`的类型。

**注** 在Junit测试中使用`SpringApplication`，调用`setWebEnvironment(false)`是很有意义的。

#### 23.7 访问应用参数

如果需要获取传递给`SpringApplication.run(…)`的应用参数，你可以注入一个`org.springframework.boot.ApplicationArguments`类型的bean。`ApplicationArguments`接口即提供对原始`String[]`参数的访问，也提供对解析成`option`和`non-option`参数的访问：

```java
import org.springframework.boot.*
import org.springframework.beans.factory.annotation.*
import org.springframework.stereotype.*

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }
}
```

**注** Spring Boot也会注册一个包含Spring `Environment`属性的`CommandLinePropertySource`，这就允许你使用`@Value`注解注入单个的应用参数。

#### 23.8 使用ApplicationRunner或CommandLineRunner

如果需要在`SpringApplication`启动后执行一些特殊的代码，你可以实现`ApplicationRunner`或`CommandLineRunner`接口，这两个接口工作方式相同，都只提供单一的`run`方法，该方法仅在`SpringApplication.run(…)`完成之前调用。

`CommandLineRunner`接口能够访问string数组类型的应用参数，而`ApplicationRunner`使用的是上面描述过的`ApplicationArguments`接口：

```java
import org.springframework.boot.*
import org.springframework.stereotype.*

@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }

}
```

如果某些定义的`CommandLineRunner`或`ApplicationRunner` beans需要以特定的顺序调用，你可以实现`org.springframework.core.Ordered`接口或使用`org.springframework.core.annotation.Order`注解。

#### 23.9 Application退出

为确保`ApplicationContext`在退出时被平静的（gracefully）关闭，每个`SpringApplication`都会注册一个JVM的shutdown钩子，所有标准的Spring生命周期回调（比如`DisposableBean`接口或`@PreDestroy`注解）都能使用。

此外，如果想在应用结束时返回特定的退出码（exit code），这些beans可以实现`org.springframework.boot.ExitCodeGenerator`接口。

### 25. Profiles

Spring Profiles提供了一种隔离应用程序配置的方式，并让这些配置只在特定的环境下生效。任何`@Component`或`@Configuration`都能注解`@Profile`，从而限制加载它的时机：

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

以正常的Spring方式，你可以使用`spring.profiles.active`的`Environment`属性来指定哪个配置生效。你可以使用通常的任何方式来指定该属性，例如，可以将它包含到`application.properties`中：

```shell
spring.profiles.active=dev,hsqldb
```

或使用命令行开关：

```shell
--spring.profiles.active=dev,hsqldb
```