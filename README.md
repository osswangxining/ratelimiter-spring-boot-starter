# ratelimiter-spring-boot-starter
Spring Boot Starter for Guava Rate Limter

## How to use this starter.

### add dependencies into pom.xml
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
  
