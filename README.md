# Rate Limiter Spring Boot Starter
This is one Spring Boot Starter for Guava Rate Limter.

## It's easy to use this starter

### 1.add dependencies into pom.xml
```
      <dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- tag::actuator[] -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- end::actuator[] -->
		<!-- tag::tests[] -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- end::tests[] -->
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>19.0</version>
		</dependency>
		<dependency>
			<groupId>org.reflections</groupId>
			<artifactId>reflections</artifactId>
			<version>0.9.10</version>
		</dependency>
		<dependency>
			<groupId>osswangxining.github.io</groupId>
			<artifactId>ratelimiter-spring-boot-starter</artifactId>
			<version>1.0-SNAPSHOT</version>
		</dependency>
	</dependencies>
  ```
  
### 2. addInterceptors for specified path patterns 

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

import osswangxining.github.io.ratelimiter.LimiterAnnotationBean;
import osswangxining.github.io.ratelimiter.RateLimiterInterceptor;
@Configuration
public class WebAppConfig extends WebMvcConfigurerAdapter {
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(new RateLimiterInterceptor()).addPathPatterns("/");
	}
	
	@Bean
	public LimiterAnnotationBean limiterAnnotationBean() {
		return new LimiterAnnotationBean("hello");
	}
}
```

### 3. Add Limiter annotation into Rest Controller
```
@RestController
public class HelloController {
    
    @RequestMapping("/")
    @Limiter(0.1)
    public String index() {
        return "Greetings from Spring Boot!";
    }
    
}
```

## The underlying technologies

### Spring Boot：定制拦截器
除了使用过滤器包装web请求，Spring MVC还提供HandlerInterceptor（拦截器）工具。根据文档，HandlerInterceptor的功能跟过滤器类似，但拦截器提供更精细的控制能力：在request被响应之前、request被响应之后、视图渲染之前以及request全部结束之后。我们不能通过拦截器修改request内容，但是可以通过抛出异常（或者返回false）来暂停request的执行。

Spring MVC中常用的拦截器有：LocaleChangeInterceptor（用于国际化配置）和ThemeChangeInterceptor,也可以增加自己定义的拦截器.

Spring Boot提供了基础类WebMvcConfigurerAdapter, 项目中的WebConfiguration类继承WebMvcConfigurerAdapter；覆盖并重写了addInterceptors(InterceptorRegistory registory)方法，这是典型的回调函数——利用该函数的参数registry来添加自定义的拦截器。

在Spring Boot的自动配置阶段，Spring Boot会扫描所有WebMvcConfigurer的实例，并顺序调用其中的回调函数，这表示：如果我们想对配置信息做逻辑上的隔离，可以在Spring Boot项目中定义多个WebMvcConfigurer的实例。

### Spring的装配bean
Spring容器负责创建应用中的bean，并通过DI维护这些bean之间的协作关系。作为开发人员，应该负责告诉Spring容器需要创建哪些bean以及如何将各个bean装配到一起。Spring提供三种装配bean的方式：
 - 基于XML文件的显式装配
 - 基于Java文件的显式装配
 - 隐式bean发现机制和自动装配

例如，@Component注解告诉Spring需要创建XX bean。XML配置中使用<context:component-scan>标签启动Component扫描功能，并可设置base-package属性。类似的，Java中@ComponentScan(basePackages = "。。。。")。

此外， 通过@Autowired注解可以完成自动装配。
