# 2. Bootstrapping Using Spruing Boot

### 2.1 Maven or Gradle dependency
First, we'll need the spring-boot-started-web dependency:
```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-web</artifactId>
 <version>2.1.1.RELEASE</version>
</dependency>
```
```groovy
implementation group: 'org.springframework.boot', name: 'spring-boot-starter-web', version: '2.7.5'
```

This started includes:

* spring-web and spring-webmvc modules that we need for out Spring web application.
* a Tomcat starter so that we can run our web application directly without explicity installing any server

### 2.2 Creating a Spring Boot Application

the most straigth-forward way to started using Spring boot is create a main class and annotate it with @SpringBootApplication:

```java
@SpringBootApplication
public class SpringBootRestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootRestApplication.class, args);
    }
}
``` 
This single annotation is equivalent to using @Configuration, @EnableAutoConfiguration, @ComponentScand.

By default, it will scan all the components in the same package or below.

Next, for java-based configuration of Spring beans, we need to create a config class and annotate it with @Configuration annotation:
```java
@Configuration
public class WebConfig {
}
``` 
This annotation is the main artifact used by Java-based Spring configuration; 
it is itself meta-annotated with @Component, which make the annotated classes standar beans and as such, also candidates form component-scanning.

The main purpose of @Configuration classes is to be sources of bean definitions for the Spring IoC Container. For more detailed description, see the official docs.

Let's also have a look at solution detailed using the core spring-webmvc-library.

