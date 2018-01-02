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

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
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
	@ConditionalOnMissingBean
        @ConditionalOnProperty(prefix = "rate.limiter",value = "enabled",havingValue = "true")
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

### 1. Spring Boot：定制拦截器
除了使用过滤器包装web请求，Spring MVC还提供HandlerInterceptor（拦截器）工具。根据文档，HandlerInterceptor的功能跟过滤器类似，但拦截器提供更精细的控制能力：在request被响应之前、request被响应之后、视图渲染之前以及request全部结束之后。我们不能通过拦截器修改request内容，但是可以通过抛出异常（或者返回false）来暂停request的执行。

Spring MVC中常用的拦截器有：LocaleChangeInterceptor（用于国际化配置）和ThemeChangeInterceptor,也可以增加自己定义的拦截器.

Spring Boot提供了基础类WebMvcConfigurerAdapter, 项目中的WebConfiguration类继承WebMvcConfigurerAdapter；覆盖并重写了addInterceptors(InterceptorRegistory registory)方法，这是典型的回调函数——利用该函数的参数registry来添加自定义的拦截器。

在Spring Boot的自动配置阶段，Spring Boot会扫描所有WebMvcConfigurer的实例，并顺序调用其中的回调函数，这表示：如果我们想对配置信息做逻辑上的隔离，可以在Spring Boot项目中定义多个WebMvcConfigurer的实例。

### 2. Spring的装配bean
Spring容器负责创建应用中的bean，并通过DI维护这些bean之间的协作关系。作为开发人员，应该负责告诉Spring容器需要创建哪些bean以及如何将各个bean装配到一起。Spring提供三种装配bean的方式：
 - 基于XML文件的显式装配
 - 基于Java文件的显式装配
 - 隐式bean发现机制和自动装配

例如，@Component注解告诉Spring需要创建XX bean。XML配置中使用<context:component-scan>标签启动Component扫描功能，并可设置base-package属性。类似的，Java中@ComponentScan(basePackages = "。。。。")。

此外， 通过@Autowired注解可以完成自动装配。

### 3. Spring Bean注入、销毁时执行指定行为
Spring提供了2种方式在Bean全部属性设置成功后执行的特定行为: 
1. 使用init-method属性。 
2. 实现InitializingBean接口。 
如果某个Bean类实现了InitializingBean接口，同时指定了init-method属性，Spring容器会先调用接口的afterPropertiesSet()方法，然后调用init-method指定的方法。 
#### 在配置文件中使用init-method属性。
```
public class MyTestBean implements BeanNameAware {
    private String id;

    public void init() {
        System.out.println("正在执行初始化方法 init...");
    }

    @Override
    public void setBeanName(String name) {
        this.id = name;
    }

    public void info() {
        System.out.println(id);
    }
}
```
指定Bean全部属性设置完成后执行该对象的init方法:
```
<bean id="myTestBean" class="xxx.MyTestBean" init-method="init"/>
```

#### 实现InitializingBean

```
public class MyTestBean2 implements BeanNameAware, InitializingBean {
    private String id;

    public void init() {
        System.out.println("正在执行初始化方法 init...");
    }

    @Override
    public void setBeanName(String name) {
        this.id = name;
    }

    public void info() {
        System.out.println(id);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("实现 InitializingBean 接口");
    }
}
```

同样销毁Bean执行特定方法也有2种:
 - 使用destory-method属性。
 - 实现DisposableBean接口。
如果指定了destory-method属性，也实现了DisposableBean接口，Spring容器会先执行DisposableBean的destroy()方法，然后执行destory-method属性指定的方法。

### Spring 生命周期方法
当一个 bean 被实例化时，它可能需要执行一些初始化使它转换成可用状态。同样，当 bean 不再需要，并且从容器中移除时，可能需要做一些清除工作。

Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：
 - Bean自身的方法：这个包括了Bean本身被@PostConstruct和@PreDestroy注解的方法和通过配置文件中<bean>的init-method和destroy-method指定的方法
 - Bean生命周期回调接口方法：这个包括了BeanNameAware、BeanFactoryAware、ApplicationContextAware、InitializingBean和DiposableBean这些接口的方法
 - 容器级生命周期接口方法：这个包括了BeanPostProcessor 和BeanFactoryPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。
	
#### 执行顺序
假设一个bean使用了上边所有的方式，那么它们的执行顺序是这样的：

1. 如果bean实现了 BeanNameAware接口，则调用BeanNameAware.setBeanName()
2. 如果bean实现了BeanFactoryAware接口，则调用BeanFactoryAware.setBeanFactory()
3. 如果bean实现了ApplicationContextAware接口，则调用ApplicationContextAware.setApplicationContext()
4. @PostConstruct 注解指定的初始化方法
5. 如果bean实现了InitializingBean接口，则调用InitializingBean.afterPropertiesSet()
6. 调用<bean>的init-method属性指定的初始化方法
7. @PreDestroy注解指定的销毁方法
8. 如果bean实现了DiposibleBean接口，则调用DiposibleBean.destory()
9. 调用<bean>的destroy-method属性指定的初始化方法

