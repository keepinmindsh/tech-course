# REST의 개념
## 1) Representational State Transfer

자원을 이름으로 구분하여 해당 자원의 상태를 주고 받는 모든 것을 의미한다.  

간단한 의미로는, 웹 상의 자료를 HTTP위에서 SOAP이나 쿠키를 통한 세션 트랙킹 같은 별도의 전송 계층 없이 전송하기 위한 아주 간단한 인터페이스를 말한다.  

레이 필딩(HTTP의 창시자 중의 하나인 2000년 논문의 의해서 소개됨)의 REST 아키텍처 형식을 따르면 HTTP나 WWW이 아닌 아주 커다란 소프트웨어 시스템을 설계하는 것도 가능하다.  

또한, 리모트 프로시저 콜 대신에 간단한 XML과 HTTP 인터페이스를 이용해 설계하는 것도 가능하다.  

## 2) REST의 요소

- 리소스 : URI
모든 자원에 고유한 ID가 존재하고, 이 자원은 Server에 존재한다.  
자원을 구별하는 ID는 ‘/groups/:group_id’와 같은 HTTP URI 다.  
Client는 URI를 이용해서 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청한다.  
- 메서드 : Method
HTTP 프로토콜의 Method를 사용한다.
HTTP 프로토콜은 GET, POST, PUT, DELETE 와 같은 메서드를 제공한다.  
- 메시지
Client가 자원의 상태(정보)에 대한 조작을 요청하면 Server는 이에 적절한 응답(Representation)을 보낸다.   
REST에서 하나의 자원은 JSON, XML, TEXT, RSS 등 여러 형태의 Representation으로 나타내어 질 수 있다.    
JSON 혹은 XML를 통해 데이터를 주고 받는 것이 일반적이다.  

REST에 대한 의미 해석
- REST 예제

배럭에서 마린을 생성한다.

```

HTTP POST, http://starcraft/barrack/
{  
    "barrack":{  
        "unit":"marine"
    }
}

``

## 3) Respresentation, State Transfer

- 자원의 표현 (Represent Resource)
- 소프트웨어에서 관리되는 모든 것과 그에 대한 표현의 방식
- 상태의 전달 (State Transfer)
- Json 또는 XML 등을 통해서 자원을 주고 받을 수 있다.

## 4) REST

    REST는 기본적으로 웹의 기존 기술과 HTTP 프로토콜을 그대로 활용하기 때문에 웹의 장점을 최대한 활용할 수 있는 아키텍처 스타일이다.
    즉, REST는 자원 기반의 구조 (ROA, Resource Oriented Architecture) 설계의 중심에 Resource 가 있고 HTTP Method 를 통해 Resource 를 처리하도록 설계된 아키텍처를 의미한다.

## 5) CRUD Operation

- GET : Read(조회)
- POST : Create(생성)
- PUT : Update(수정)
- DELETE : Delete(삭제)
- HEAD : header 정보 조회

```java

package com.lines.demo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RestController;

import com.lines.demo.biz.UserHandler;

@RestController
public class UserController {
  
  @Autowired
  UserHandler userHandler;
  
  @GetMapping("/users")
  public Object usersGet() {
    return userHandler.getUserList();
  }
  
  @PostMapping("/user")
  public Object userPost() {
    return userHandler.getUserList();
  }
  
  @PutMapping("/user")
  public Object userPut() {
    return userHandler.getUserList();
  }
  
  @DeleteMapping("/user")
  public Object userDelete() {
    return userHandler.getUserList();
  }
}

```

# 6) REST의 장점

- HTTP 프로토콜의 인프라를 그대로 사용하므로 REST API 사용을 위한 별도의 인프라를 구축할 필요가 없다.
- HTTP 프로토콜의 표준을 최대한 활용하여 여러 추가적인 장점을 함께 가져갈 수 있게 해준다.
- HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.
- Hypermedia API의 기본을 충실히 지키면서 범용성을 보장한다.
- REST API 메시지가 의도하는 바를 명확하게 나타내므로 의도하는 바를 쉽게 파악할 수 있다.
- 여러가지 서비스 디자인에서 생길 수 있는 문제를 최소화한다.
- 서버와 클라이언트의 역할을 명확하게 분리한다.

# REST의 특징
- Server-Client(서버-클라이언트 구조)
  - 자원이 있는 쪽이 Server, 자원을 요청하는 쪽이 Client 가 된다.
  - REST Server 는 API를 제공하고 비즈니스 로직 처리 및 저장을 책임 진다.
  - Client : 사용자 인증 이나 Context 등을 직접 관리하고 책임 진다.
- Stateless(무상태)
  - HTTP 프로토콜은 Stateless Protocol이므로 REST 역시 무상태성을 갖는다.
- Cacheable(캐시 처리 가능)
  - 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라를 그대로 활용할 수 있다.
  - 즉, HTTP가 가진 가장 강력한 특징 중 하나인 캐싱 기능을 적용할 수 있다.
  - HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이용하면 캐싱 구현이 가능하다.
  - 대량의 요청을 효율적으로 처리하기 위해 캐시가 요구된다.
- Layered System(계층화)
  - Client는 REST API Server만 호출한다.
  - REST Server는 다중 계층으로 구성될 수 있다.
  - API Server는 순수 비즈니스 로직을 수행하고 그 앞 단에 보안, 로드밸런싱, 암호화, 사용자 인증 등을 추가하여 구조 상의 유연성을 줄 수 있다.
  - 또한 로드밸런싱, 공유 캐시 등을 통해 확장성과 보안성을 향상 시킬 수 있다.
- Uniformed Interface(인터페이스 일관성)
  - URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다.
  - HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.
  - 특정 언어나 기술에 종속되지 않는다.

#  REST API의 개념
API(Application Programming Interface)란

데이터와 기능의 집합을 제공하여 컴퓨터 프로그램 간 상호작용을 촉진하며, 서로 정보를 교환 가능 하도록 하는 것

응용 프로그램에서 사용할 수 있도록, 운영 체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스를 뜻한다.

# REST API의 정의

REST 기반으로 서비스 API를 구현한 것

최근 OpenAPI(누구나 사용할 수 있도록 공개된 API: 구글 맵, 공공 데이터 등), 마이크로 서비스(하나의 큰 애플리케이션을 여러 개의 작은 애플리케이션으로 쪼개어 변경과 조합이 가능하도록 만든 아키텍처) 등을 제공하는 업체 대부분은 REST API를 제공한다.

#  REST API의 설계 규칙
- URI는 정보의 자원을 표현해야 한다.
- 동사 보다는 명사를, 대문자 보다는 소문자를 사용한다.
- 도큐먼트(객체 인스턴스나 데이터베이스 레코드와 유사한 개념)의 이름으로 단수 명사를 사용해야 한다.
- 컬렉션(서버에서 관리하는 디렉터리라는 리소스)의 이름으로는 복수 명사를 사용해야 한다.
- 스토어(클라이언트에서 관리하는 리소스 저장소)의 이름으로는 복수 명사를 사용해야 한다.
- 자원에 대한 행위는 HTTP Method(GET, PUT, POST, DELETE 등)로 표현한다.
- URI에 HTTP Method가 들어가면 안된다.
- URI에 행위에 대한 동사 표현이 들어가면 안된다. (즉, CRUD 기능을 나타내는 것은 URI에 사용하지 않는다.)
- 경로 부분 중 변하는 부분은 유일한 값으로 대체한다. (즉 하나의 특정 Resource를 나타내는 고유 값이다. )
- 슬래시 구분자(//)는 계층 관계를 나타내는 데 사용한다.
- URI 마지막 문자로 슬래시(//)를 포함하지 않는다.
- 밑줄(_)은 URI에 사용하지 않는다.
- URI 경로에는 소문자가 적합하다.
- 파일확장자에는 UIR를 포함하지 않는다.

Ex) GET / starcraft/barrack/345/photo HTTP/1.1 Host: restapi.example.com Accept: image/jpg

**리소스 간에 연관 관계가 있는 경우**    

응답 상태 코드  
- 1XX : 전송 프로토콜 수준의 정보 교환
- 2XX : 클라이언트의 요청이 성공적으로 수행됨
- 3XX : 클라이언트는 요청을 완료하기 위해 추가적인 행동을 취해야 함
- 4XX : 클라이언트의 잘못된 요청
- 5XX : 서버쪽 오류로 인한 상태코드

#  RESTful의 개념
RESTful은 일반적으로 REST라는 아키텍처를 구현하는 웹 서비스를 나타내기 위해 사용되는 용어이다.
REST API를 제공하는 웹 서비스를 'RESTful' 하다고 할 수 있다.
RESTful은 REST를 REST답게 쓰기 위한 방법으로, 누군가가 공식적으로 발표한 것이 아니다.
즉, REST 원리를 따르는 시스템은 RESTful이란 용어로 지칭된다.
