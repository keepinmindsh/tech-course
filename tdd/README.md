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

### Sample 예제 

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