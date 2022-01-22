# **Spring Boot 이해하기** 

## **Spring Initializer** 

- https://start.spring.io/

## **Spring Docs**

- https://docs.spring.io/spring-boot/docs/

## **Spring Boot Starter ( Web, JPA, Lombok )**

**기능**  

- Create stand-alone Spring applications

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)

- Provide opinionated 'starter' dependencies to simplify your build configuration

- Automatically configure Spring and 3rd party libraries whenever possible

- Provide production-ready features such as metrics, health checks, and externalized configuration

- Absolutely no code generation and no requirement for XML configuration


### **구조** 

![](https://github.com/keepinmindsh/tech-course/blob/main/assets/springboot_001.png)

> src  

> resources  
  - application.yml 

> .gitignore  

> build.gradle  

> settings.gradle  

### **진입점(Entry Points)**

```java 

package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}

```

### **테스트(Spring Boot Tests)**

```java

package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DemoApplicationTests {

	@Test
	void contextLoads() {
	}

}

```

### **기본 Gradle 설정(build.gradle)**

```gradle

plugins {
  id 'org.springframework.boot' version '2.6.2'
  id 'io.spring.dependency-management' version '1.0.11.RELEASE'
  id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'org.springframework.boot:spring-boot-starter-web'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
  useJUnitPlatform()
}

```

### **Spring Boot**

##### 우리가 선언하는 @SpringBootApplication 은 

```java

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

##### Annotations 내에 아래의 코드를 정의하고 있는데, 

```java 

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {


```

##### @SpringBootConfiguration

##### @EnableAutoConfiguration

- Spring Boot의 의존성중 하나인 org.springframework.boot:spring-boot-autoconfigure 를 확인해 보자

![alt text](https://github.com/keepinmindsh/tech-course/blob/f23397c1a08afff627511fe9a133628183cfd87e/assets/springboot_002.png)

##### @ComponentScan

- @Component @Configuration @Repository @Service @Controller @RestController  
해당 어노테이션이 선언된 하위 패키지에서 위와 같은 Annotation을 찾아서 Bean으로 등록한다.