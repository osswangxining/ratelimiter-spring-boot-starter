# ratelimiter-spring-boot-starter
Spring Boot Starter for Guava Rate Limter

## How to use this starter.

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
