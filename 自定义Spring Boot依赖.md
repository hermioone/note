# 自定义Spring Boot依赖

以配置HttpClient来举例

## 1. 加载配置文件

```java
package co.imooc.house.autoconfig;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;

@ConfigurationProperties(prefix = "spring.httpclient")
public class HttpClientProperties {

    private Integer connectTimeout=1000;
    private Integer socketTimeOut = 10000;
    private String agent = "agent";
    private Integer maxConnPerRoute = 10;
    private Integer maxConnTotal = 50;

    public Integer getConnectTimeout() {
        return connectTimeout;
    }

    public void setConnectTimeout(Integer connectTimeout) {
        this.connectTimeout = connectTimeout;
    }

    public Integer getSocketTimeOut() {
        return socketTimeOut;
    }

    public void setSocketTimeOut(Integer socketTimeOut) {
        this.socketTimeOut = socketTimeOut;
    }

    public String getAgent() {
        return agent;
    }

    public void setAgent(String agent) {
        this.agent = agent;
    }

    public Integer getMaxConnPerRoute() {
        return maxConnPerRoute;
    }

    public void setMaxConnPerRoute(Integer maxConnPerRoute) {
        this.maxConnPerRoute = maxConnPerRoute;
    }

    public Integer getMaxConnTotal() {
        return maxConnTotal;
    }

    public void setMaxConnTotal(Integer maxConnTotal) {
        this.maxConnTotal = maxConnTotal;
    }
}
```

这个Java Config会加载```application.properties```中的以spring.httpclient开头的配置项：

```properties
spring.httpclient.connectTimeout=xx
spring.httpclient.socketTimeOut=xx
spring.httpclient.agent=xx
spring.httpclient.maxConnPerRoute=xx
spring.httpclient.maxConnTotal=xx
```

如果要读取自定义的配置文件，需要在HttpClientProperties上加上注解：

```java
@ConfigurationProperties(prefix = "spring.httpclient")
@PropertySource(value = "classpath:xx.properties")
public class HttpClientProperties {}
```

## 2. 编写自动配置项

```java
package co.imooc.house.autoconfig;

import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.impl.NoConnectionReuseStrategy;
import org.apache.http.impl.client.HttpClientBuilder;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConditionalOnClass({HttpClient.class})     //只有在HttpClient这个class位于类路径上，才会实例化一个Bean
@EnableConfigurationProperties(HttpClientProperties.class)      //将HttpClientProperties作为一个bean引入进来
/**
 * 注解@ConfigurationProperties主要用来把properties配置文件转化为bean来使用的，
 * 而@EnableConfigurationProperties注解的作用是@ConfigurationProperties注解生效。
 * 如果只配置@ConfigurationProperties注解，在IOC容器中是获取不到properties配置文件转化的bean的
 */
public class HttpClientAutoConfiguration {

    private final HttpClientProperties properties;

    public HttpClientAutoConfiguration(HttpClientProperties properties) {
        this.properties = properties;
    }

    @Bean
    @ConditionalOnMissingBean(HttpClient.class)     //仅仅在当前上下文中不存在HttpClient对象时，才会实例化一个Bean
    public HttpClient httpClient() {
        //构建requestConfig
        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(properties.getConnectTimeout())
                .setSocketTimeout(properties.getSocketTimeOut()).build();
        HttpClient client = HttpClientBuilder.create().setDefaultRequestConfig(requestConfig)
                .setUserAgent(properties.getAgent())
                .setMaxConnPerRoute(properties.getMaxConnPerRoute())
                .setConnectionReuseStrategy(new NoConnectionReuseStrategy())
                .build();
        return client;
    }
}
```

## 3. 编写注解

注解的作用是为了防止上面的代码和spring boot的启动项不在同一个包下，这样spring boot的启动项默认情况下不会去扫描这些包，所以也就不会自动把HttpClient注册进spring容器中。

```java
package co.imooc.house.autoconfig;

import org.springframework.context.annotation.Import;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(HttpClientAutoConfiguration.class)
public @interface EnableHttpClient {
}
```

## 4. 在启动项上加注解

```java
package com.imooc;			//启动项和配置不在同一个包下

import co.imooc.house.autoconfig.EnableHttpClient;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableHttpClient
public class HouseApplication {

	public static void main(String[] args) {
		SpringApplication.run(HouseApplication.class, args);
	}
}
```

此时就可以测试HttpClient是否已注册进spring容器中

```java
package com.imooc.house;

import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.util.EntityUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class HouseApplicationTests {

	@Autowired
	private HttpClient httpClient;

	@Test
	public void testHttpClient() throws Exception {
		System.out.println(EntityUtils.toString(httpClient.execute(new HttpGet("http://www.baidu.com")).getEntity()));
	}
}
```

