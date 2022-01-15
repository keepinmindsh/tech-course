# TDD 이해하기 

```java

@RestController
@RequiredArgsConstructor
public class TddController {

    private final TDDService tddService;

    @GetMapping("/helloworld")
    String helloWorld(){
        return tddService.greeting();
    }
}

public interface TDDService {
    String greeting();
}

@Service
public class TDDServiceImpl implements TDDService {
    @Override
    public String greeting() {
        return "Hello World";
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
	public void helloworldTests() throws Exception {
		when(service.greeting()).thenReturn("Hello World");
		this.mockMvc.perform(get("/helloworld"))
				.andDo(print())
				.andExpect(status().isOk())
				.andExpect(content().string(containsString("Hello World")));
	}
}

```