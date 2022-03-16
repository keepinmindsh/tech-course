# TDD 이해하기 

**TDD(테스트 주도 개발) Test Driven Development**  

업무 코드를 작성하기 전에 테스트 코드를 먼저 만드는 것  

### 기본 환경 

 - h2database : https://www.h2database.com/html/main.html

 - gradle 기본 설정 

 ```groovy
plugins {
	id 'org.springframework.boot' version '2.6.2'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'bong.lings'
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

	// https://mvnrepository.com/artifact/com.h2database/h2
	implementation 'com.h2database:h2:2.0.204'

	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	compileOnly 'org.projectlombok:lombok:1.18.22'
	annotationProcessor 'org.projectlombok:lombok:1.18.22'

	testCompileOnly 'org.projectlombok:lombok:1.18.22'
	testAnnotationProcessor 'org.projectlombok:lombok:1.18.22'
}

test {
	useJUnitPlatform()
}
```

- [Spring Boot 2.6.2 기준 test auto configuration ](https://docs.spring.io/spring-boot/docs/2.6.2/reference/htmlsingle/
#test-auto-configuration)

### Spring API TDD 

- Controller
  - @WebMvcTest
- Service 
  - mocking 

mock : 실제 객체를 만들어 사용하기에 시간, 비용 등의 Cost가 높거나 혹은 객체간의 의존성이 강해 구현하기 힘들경우 가짜객체를 만들어 사용하는 방법이다. 

- Repository
  - @DataJpaTest


### Sample 예제 - Controller 

- @WebMvcTest 
  - Controller에 있는 Bean들만 로드
  - 특정 컨트롤러에 대해서 지정 가능

```java

@RestController
@RequiredArgsConstructor
public class TddController {

    private final TDDService tddService;

    @GetMapping("/helloworld")
    List<Member> helloWorld(){
        return tddService.member();
    }

    @PostMapping("/helloworld")
    void helloWorldSave(String name){
        tddService.memberSave(name);
    }
}

public interface TDDService {
    List<Member> member();

    void memberSave(String name);
}

@Service
@RequiredArgsConstructor
public class TDDServiceImpl implements TDDService {

    private final MemberRepository memberRepository;

    @Override
    public List<Member> member() {
        return memberRepository.findAll();
    }

    @Override
    public void memberSave(String name) {

        Member member = new Member();
        member.setName(name);

        memberRepository.save(member);
    }
}

```

```java

import bong.lings.springfor_tests.controller.TddController;
import bong.lings.springfor_tests.service.TDDService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(TddController.class)
class SpringForTestsApplicationTests {

	@Autowired
	private MockMvc mockMvc;

	@MockBean
	private TDDService service;

	@Test
	public void helloWorldGetTests() throws Exception {
		when(service.member()).thenReturn(new ArrayList<Member>());

		this.mockMvc.perform(get("/helloworld"))
				.andDo(print())
				.andExpect(status().isOk())
				.andExpect(content().json("[]"));
	}

	@Test
	public void helloWorldPostTests() throws Exception {
		when(service.member()).thenReturn(new ArrayList<Member>());

		this.mockMvc.perform(post("/helloworld"))
				.andDo(print())
				.andExpect(status().isOk());

	}
}

```

### 참조 

> [Spring Boot - jpa,service,controller](https://data-make.tistory.com/717)  
> [martinfowler - BDD](https://martinfowler.com/bliki/GivenWhenThen.html)   
> [Spring - TDD](https://otrodevym.tistory.com/entry/Spring%EC%97%90%EC%84%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BC%80%EC%9D%B4%EC%8A%A4-TDD-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0)  
> [Spring - TDD 01](https://dzone.com/articles/test-driven-development-with-spring-boot-rest-api)  
> [Mockito](https://effortguy.tistory.com/142?category=841326)  
> [Spring Mock](https://gocheat.github.io/spring/spring_test-1/)
> [클린 코드와 좋은 설계를 이끄는 단위 테스트](http://www.yes24.com/Product/goods/11361087?scode=032&OzSrank=1)