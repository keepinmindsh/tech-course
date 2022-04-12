

![API 구성](https://github.com/keepinmindsh/tech-course/blob/main/assets/Eureka_Server_Client.png)

# Eureka Server 

### build.gradle

```groovy
dependencies {
    // Add Eureka Server Dependency
    compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-server')
}
```

### Application.yml

```
# -- Server Port

server:
  port: 8787

# -- Eureka
eureka:
  instance:
    hostname: 127.0.0.1
  client:
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    register-with-eureka: false
    fetch-registry: false

```

### Eureka Server 

```java
@EnableEurekaServer
@SpringBootApplication

public class EurekaServerApplication {

  public static void main(String[] args) {
    SpringApplication.run(EurekaServerApplication.class, args);
  }
}
```

# Eureka Client

### build.gradle

```groovy
dependencies {

    // Add Eureka Client Dependency

    compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-client')

}
```

### Application.yml

```
# -- Default spring configuration

spring:
  application:
    name: (Your Service Name)

# -- Eureka client
eureka:
  client:
    serviceUrl:
      defaultZone: ${EUREKA_URL:http://127.0.0.1:8787/eureka/}
```

### Eureka Client 

```java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientApplication {

  public static void main(String[] args) {
    SpringApplication.run(EurekaClientApplication.class, args);
  }
}
```


> 참조 : https://lion-king.tistory.com/12