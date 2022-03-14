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
  - 기본 소스 경로 

> resources  
  - application.yml : SpringBoot의 설정값을 세팅하는 파일 

> .gitignore  
  - git을 사용할 경우, 불필요한 파일을 제외시켜야할 경우 

> build.gradle  
  - gradle 설정 파일 

> settings.gradle  
  - gradle의 모듈 정의 및 세팅 파일 

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

## **Spring Boot**

### **우리가 선언하는 @SpringBootApplication 은**

```java

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

### **Annotations 내에 아래의 코드를 정의하고 있는데, 선언된 Annotations 들 중에 중요한 것은 아래에 별도로 표시한다.** 

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

### **@SpringBootConfiguration**

스프링 부트의 설정을 나타내는 어노테이션이다. 스프링의 @Configuration을 대체하며 스프링 부트 전용 어노테이션이다. 테스트 어노테이션을 사용할 때 계속 이 어노테이션을 찾기 때문에 스프링 부트에서는 필수 어노테이션이다.

### **@EnableAutoConfiguration**

- Spring Boot의 의존성중 하나인 org.springframework.boot:spring-boot-autoconfigure 를 확인해 보자

![alt text](https://github.com/keepinmindsh/tech-course/blob/f23397c1a08afff627511fe9a133628183cfd87e/assets/springboot_002.png)

### **@ComponentScan**

- @Component @Configuration @Repository @Service @Controller @RestController  
해당 어노테이션이 선언된 하위 패키지에서 위와 같은 Annotation을 찾아서 Bean으로 등록한다.


### **@Component & @Configuration**

@Component 와 @Configuration은 큰 차이는 없습니다.   

위에서 보앗듯이, @Configuration의 내부를 살펴보면 @Component가 정의되어 있습니다. 따라서 @Component와 @Bean을 비교하는 것이 맞습니다.   

우선적으로, 개발자가 직접 작성한 클래스에 대하여 @Component는 위와 같이 bean으로 등록 할 수 있습니다. 반대로 라이브러리 혹은 내장 클래스등 개발자가 직접 제어가 불가능한 클래스의 경우 @Configuration + @Bean 어노테이션을 이용하여 bean으로 등록하여 사용하면 됩니다.

- Configuration 

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

   /**
    * Explicitly specify the name of the Spring bean definition associated with the
    * {@code @Configuration} class. If left unspecified (the common case), a bean
    * name will be automatically generated.
    * <p>The custom name applies only if the {@code @Configuration} class is picked
    * up via component scanning or supplied directly to an
    * {@link AnnotationConfigApplicationContext}. If the {@code @Configuration} class
    * is registered as a traditional XML bean definition, the name/id of the bean
    * element will take precedence.
    * @return the explicit component name, if any (or empty String otherwise)
    * @see AnnotationBeanNameGenerator
    */
   @AliasFor(annotation = Component.class)
   String value() default "";

   /**
    * Specify whether {@code @Bean} methods should get proxied in order to enforce
    * bean lifecycle behavior, e.g. to return shared singleton bean instances even
    * in case of direct {@code @Bean} method calls in user code. This feature
    * requires method interception, implemented through a runtime-generated CGLIB
    * subclass which comes with limitations such as the configuration class and
    * its methods not being allowed to declare {@code final}.
    * <p>The default is {@code true}, allowing for 'inter-bean references' via direct
    * method calls within the configuration class as well as for external calls to
    * this configuration's {@code @Bean} methods, e.g. from another configuration class.
    * If this is not needed since each of this particular configuration's {@code @Bean}
    * methods is self-contained and designed as a plain factory method for container use,
    * switch this flag to {@code false} in order to avoid CGLIB subclass processing.
    * <p>Turning off bean method interception effectively processes {@code @Bean}
    * methods individually like when declared on non-{@code @Configuration} classes,
    * a.k.a. "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore
    * behaviorally equivalent to removing the {@code @Configuration} stereotype.
    * @since 5.2
    */
   boolean proxyBeanMethods() default true;

}
```
- Service 

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

   /**
    * The value may indicate a suggestion for a logical component name,
    * to be turned into a Spring bean in case of an autodetected component.
    * @return the suggested component name, if any (or empty String otherwise)
    */
   @AliasFor(annotation = Component.class)
   String value() default "";

}
```

- Repository 

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

   /**
    * The value may indicate a suggestion for a logical component name,
    * to be turned into a Spring bean in case of an autodetected component.
    * @return the suggested component name, if any (or empty String otherwise)
    */
   @AliasFor(annotation = Component.class)
   String value() default "";

}
```

- Controller

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

   /**
    * The value may indicate a suggestion for a logical component name,
    * to be turned into a Spring bean in case of an autodetected component.
    * @return the suggested component name, if any (or empty String otherwise)
    */
   @AliasFor(annotation = Component.class)
   String value() default "";

}

```

### **@RestController VS @Controller** 

- @RestController 

@RestController는 Spring MVC Controller에 @ResponseBody가 추가된 것입니다. 당연하게도 RestController의 주용도는 Json 형태로 객체 데이터를 반환하는 것입니다. 최근에 데이터를 응답으로 제공하는 Restful API를 개발할 때 주로 사용합니다.

@RestController가 Data를 반환하기 위해서는 viewResolver 대신에 HttpMessageConverter가 동작합니다. HttpMessageConverter에는 여러 Converter가 등록되어 있고, 반환해야 하는 데이터에 따라 사용되는 Converter가 달라집니다. 단순 문자열인 경우에는 StringHttpMessageConverter가 사용되고, 객체인 경우에는 MappingJackson2HttpMessageConverter가 사용되며, 데이터 종류에 따라 서로 다른 MessageConverter가 작동하게 됩니다. Spring은 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해 적합한 HttpMessageConverter를 선택하여 이를 처리합니다.

- @Controller 

전통적인 Spring MVC의 컨트롤러인 @Controller는 주로 View를 반환하기 위해 사용합니다. 아래와 같은 과정을 통해 Spring MVC Container는 Client의 요청으로부터 View를 반환합니다.


### **Application.yml**

- 가독성
- 리스트 표현
- 주석 
- Spring Profile의 적용 


### **Study Cases - Gradle Multi Modules** 

- operation_systems
  - forpms
    -  /forpms/a
  - forcms
    -  /forcms/a
  - forbwi
    -  /forbwi/a
  - forexp
    -  /forexp/a
  - forwith
    -  /forwith/a
  - starter 
    - modules 
      - context 
    - starter 
      - StarterApplication 	   


### **참조** 

> [[Spring] @Controller와 @RestController 차이](https://mangkyu.tistory.com/49) - 출처: https://mangkyu.tistory.com/49 [MangKyu's Diary]