---
layout : single
title : "[MVC] controller"
categories: MVC
tag : [MVC,개념정리,Spring]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## Spring MVC

- 스프링은 서블릿 기반으로 동작함.
- Spring의 모듈 중에서 **서블릿(Servlet) API**를 기반으로 클라이언트의 요청을 처리하는 모듈이 바로 `spring-webmvc` = Spring MVC = Spring MVC 프레임워크
    - 서블릿 :  **클라이언트의 요청을 처리하도록 특정 규약에 맞추어서 Java 코드로 작성하는클래스 파일**
    - **아파치 톰캣(Apache Tomcat)**은 이러한 서블릿들이 웹 애플리케이션으로 실행이 되도록 해주는 **서블릿 컨테이너(Servlet Container)** 중 하나

### M

**Model**은 Spring **M**VC에서 **M**에 해당

처리한 작업의 결과 데이터를 클라이언트에게 응답으로 돌려줘야 하는데, 이때 클라이언트에게 응답으로 돌려주는 **작업의 처리 결과 데이터**를 **Model**

> 클라이언트의 요청 사항을 구체적으로 처리하는 영역을 **서비스 계층(Service Layer)**이라고 하며, 실제로 요청 사항을 처리하기 위해 Java 코드로 구현한 것을 **비즈니스 로직(Business Logic)**이라고 합니다.
> 

### V

**View**는 Spring M**V**C에서 **V**에 해당

**View는 앞에서 설명한 Model 데이터를 이용해서 웹브라우저 같은 클라이언트 애플리케이션의 화면에 보이는 리소스(Resource)를 제공하는 역할**

- **HTML 페이지의 출력**
- Spring MVC에서 지원하는 HTML 페이지 출력 기술에는 Thymeleaf, FreeMarker, JSP + JSTL, Tiles 등이 있습니다.

⇒ 앞으로 V는 안하고 json타입으로 만들 예정

**💙 JSON(JavaScript Object Notation)이란?**

- JSON의 기본 포맷
- **`{”속성”:”값”}`** 형태입니다.
- [https://jsonformatter.org/json-to-java](https://jsonformatter.org/json-to-java)

### C

**Controller는 클라이언트 측의 요청을 직접적으로 전달받는 엔드포인트(Endpoint)로써 Model과 View의 중간에서 상호 작용을 해주는 역할**

즉, 클라이언트 측의 요청을 전달받아서 비즈니스 로직을 거친 후에 Model 데이터가 만들어지면, 이 Model 데이터를 View로 전달하는 역할

```java
@RestController
@RequestMapping(path = "/v1/coffee")
public class CoffeeController {
    private final CoffeeService coffeeService;

    CoffeeController(CoffeeService coffeeService) {
        this.coffeeService = coffeeService;
    }

    @GetMapping("/{coffee-id}")  // (1)
    public Coffee getCoffee(@PathVariable("coffee-id") long coffeeId) {
        return coffeeService.findCoffee(coffeeId); // (2)
    }
}

```

- (1)의 `@GetMapping` 애노테이션을 통해 클라이언트 측의 요청을 수신
- (2)에서 `CoffeeService` 클래스의 `findCoffee()` 메서드를 호출해서 비즈니스 로직을 처리
- (2)에서 비즈니스 로직을 처리한 다음 리턴 받는 Coffee가 여기서는 **Model 데이터**

## MVC의 전체적인 동작흐름

- Client가 요청 데이터 전송
  
    → Controller가 요청 데이터 수신 → 비즈니스 로직 처리 → Model 데이터 생성
    
    → Controller에게 Model 데이터 전달 → Controller가 View에게 Model 데이터 전달
    
    → View가 응답 데이터 생성
    

## Spring MVC의 동작 방식과 구성 요소 ( 완전 중요 ✨✨✨✨)

- 직접 다루지는 않아도 흐름이 이렇다는 것은 꼭 알아야 함.
  
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/55d7bb50-2eb8-4e9e-83bd-af88b04a3d66" width=500/>
    
    1. 요청전송하면 바로 Controller로 가지않음  - DispatcherServlet 이 요청을 받음.
    2. DispatcherServlet이 handlerMapping에게 보냄(**클라이언트의 요청을 처리할 Controller에 대한 검색을 요청**). → handlerMapping이 컨트롤러 객체를 다 알고 있음. controller 찾아서 알려줌
    3. **HandlerMapping은 클라이언트 요청과 매핑되는 핸들러 객체를 다시 DispatcherServlet에게 리턴**
    4. `DispatcherServlet`은 Handler 메서드를 직접 호출하지 않고, HandlerAdpater에게 **Handler 메서드 호출을 위임**
    5. `HandlerAdapter`는 DispatcherServlet으로부터 전달받은 Controller 정보를 기반으로 **해당 Controller의 Handler 메서드를 호출**
    6.  `Controller`의 Handler 메서드는 비즈니스 로직 처리 후 리턴 받은 **Model 데이터를 HandlerAdapter에게 전달**
    7.  `HandlerAdapter`는 전달받은 **Model 데이터와 View 정보를 다시 DispatcherServlet에게 전달**
    8. `DispatcherServlet`은 전달받은 View 정보를 다시 ViewResolver에게 **전달해서 View 검색을 요청**
    9. `ViewResolver`는 View 정보에 해당하는 View를 찾아서 **View를 다시 리턴**
    10.  `DispatcherServlet`은 ViewResolver로부터 전달받은 View 객체를 통해 Model 데이터를 넘겨주면서 클라이언트에게 전달할 **응답 데이터 생성을 요청**
    11. `View`는 응답 데이터를 생성해서 다시 DispatcherServlet에게 전달
    12. `DispatcherServlet`은 **View로부터 전달받은 응답 데이터를 최종적으로 클라이언트에게 전달**

- **DispatcherServlet의 역할**
    - 클라이언트로부터 요청을 전달 받으면 HandlerMapping, HandlerAdapter, ViewResolver, View 등 대부분의 Spring MVC  구성 요소들과 상호 작용을 함
    - 바빠보이지만 실제로 요청에 대한 처리는 다른 구성 요소들에게 위임(Delegate)하고 있음.
    - DispatcherServlet이 애플리케이션의 가장 앞단에 배치되어 다른 구성요소들과 상호작용하면서 클라이언트의 요청을 처리하는 패턴을 **Front Controller Pattern**이라고 한다.

# Controller

## 커피 주문 애플리케이션 실습으로 MVC 배우기

[요구조건]

---

- 주인이 커피 정보를 관리하는 기능
    - 커피 정보 등록 기능
    - 등록한 커피 정보 수정 기능
    - 등록한 커피 정보 삭제 기능
    - 등록한 커피 정보 조회 기능
- 고객이 커피 정보를 조회하는 기능
    - 커피 정보 조회 기능
- 고객이 커피를 주문하는 기능
    - 커피 주문 등록 기능
    - 커피 주문 취소 기능
    - 커피 주문 조회 기능

### 커피 주문 애플리케이션에 필요한 리소스

- 고객이 주문한 커피를 주인이 조회하는 기능
    - 커피 주문 조회 기능
    - 고객에게 전달 완료한 커피에 대한 주문 완료 처리 기능

---

### Controller 설계

- REST API 기반의 애플리케이션에서는 일반적으로 애플리케이션이 제공해야 될 기능을 **리소스(Resource, 자원)**로 분류
- 커피주문 애플리케이션에서 기본적으로 필요한 Resourse는 회원/커피/주문 에 해당
- 주인의 기능을 따로 분리해야 되는 것 아니냐는 의문이 들 수도 있지만
  
    → 고객과 주인의 인증 프로세스가 복잡해질 가능성이 높음
    
    → 실제 큰 업체에는 백오피스를 구현해서 사용함
    

### 패키지 구조 생성

왼쪽은 기능 기반, 오른쪽은 계층 기반

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/934b2fd7-b8c0-4db8-95d8-0e43a889a503" width=500/>

계층 기반으로 나눌 수 있지만 기능 기반 기준으로 나눌 예정임.

→ 왜냐면 ? Spring Boot 팀에서는 **테스트와 리팩토링이 용이하고, 향후에 마이크로 서비스 시스템으로의 분리가 상대적으로 용이**한 **기능 기반 패키지 구조** 사용을 권장했기 때문

### 엔트리포인트(Entrypoint) 클래스 작성

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/2029ac22-fa02-4144-8fc2-7d18a619121b" width=500/>

**`@SpringBootApplication`**

- 자동 구성을 활성화
- `@Component`가 붙은 클래스를 검색한 후(scan), **Spring Bean**으로 등록하는 기능을 활성화
- `@Configuration` 이 붙은 클래스를 자동으로 찾아주고, 추가적으로 Spring Bean을 등록하는 기능을 활성화

**`SpringApplication.run(SpringStartApplication.class, args);`**

- Spring 애플리케이션을 부트스트랩하고, 실행하는 역할을 함

> 부트스트랩(Bootstrap)이란?
> 
> 
> 애플리케이션이 실행되기 전에 여러 가지 설정 작업을 수행하여 실행 가능한 애플리케이션으로 만드는 단계를 의미
> 

### 커피 주문 애플리케이션의 Controller 구조 작성

1. **MemberController 구조 작성**
   
    ```java
    package com.springboot.member;
    
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    @RequestMapping("/v1/members")
    public class MemberController {
    }
    ```
    
    1. `@RestController`   ⇒ 스프링컨테이너를 등록시켜줌과 동시에 컨트롤러를 구현할 거라는 것을 알림
       
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/8a113f32-4ca5-47ec-9c40-6c7123f36fb3" width=200/>
        
    2. 해당 주소(/v1/members)로 들어오는 요청은  MemberController가 담당
2. **OrderController 구조 작성**
   
    ```java
    package com.springboot.order;
    
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    @RequestMapping("/v1/orders")
    public class OrderController {
    }
    ```
    
3. **CoffeeController 구조 작성**
   
    ```java
    package com.springboot.coffee;
    
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    @RequestMapping("/v1/coffees")
    public class CoffeeController {
    
    }
    ```
    

### 핸들러 메서드(Handler Method)

- 클라이언트의 요청을 전달받아서 처리하기 위해서는 요청 핸들러 메서드(Request Handler Method)가 필요

- Postman 사용법 : [https://itvillage.tistory.com/38](https://itvillage.tistory.com/38)
- 하기 코드에 나오는 annotation은 링크 참조, 따로 정리할 예정!
  
     → [https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods.html](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods.html)


### 1 - 레거시코드

1. MemberController
   
    ```java
    package com.springboot.member;
    import org.springframework.http.MediaType;
    import org.springframework.web.bind.annotation.*;
    
    @RestController
    @RequestMapping(value = "/v1/members", produces = {MediaType.APPLICATION_JSON_VALUE})
    public class MemberController {
        @PostMapping
        public String postMember(@RequestParam("email") String email,
                                 @RequestParam("name") String name,
                                 @RequestParam("phone") String phone) {
            System.out.println("# email: " + email);
            System.out.println("# name: " + name);
            System.out.println("# phone: " + phone);
    
            String response =
                    "{\"" +
                            "email\":\"" + email + "\"," +
                            "\"name\":\"" + name + "\"," +
                            "\"phone\":\"" + phone +
                            "\"}";
            return response;
        }
    
        @GetMapping("/{member-id}")
        public String getMember(@PathVariable("member-id")long memberId) {
            System.out.println("# memberId: " + memberId);
    
            // not implementation
            return null;
        }
    
        @GetMapping
        public String getMembers() {
            System.out.println("# get Members");
    
            // not implementation
            return null;
        }
    }
    
    ```
    
    1) **postMember() 메서드**는 회원 정보를 등록해 주는 핸들러 메서드
    
    2) **@PostMapping**
    
    클라이언트의 요청 데이터(request body)를 서버에 생성할 때 사용하는 애너테이션, 클라이언트 쪽에서 요청 전송 시, HTTP Method 타입을 동일하게 맞춰주어야 함.
    
    3) **@RequestParam**
    
    핸들러 메서드의 파라미터 종류 중 하나, 클라이언트 쪽에서 전송하는 요청 데이터를 쿼리 파라미터(Query Parmeter 또는 Query String), 폼 데이터(form-data), x-www-form-urlencoded 형식으로 전송하면 이를 서버 쪽에서 전달받을 때 사용하는 애너테이션
    
    4) 클라이언트 쪽에서 JSON 형식의 데이터를 전송받아야 하기 때문에 응답 문자열을 JSON 형식에 맞게 작성
    
    5. **Postman** : **postMember() 핸들러 메서드** 매핑 URI로의 요청/응답 모습
    
       <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/7fdb3fcb-e030-4e55-8597-a0871ad96b92" width=500/>
    
    6) **GetMapping**
    
    클라이언트가 서버에 리소스를 조회할 때 사용하는 애너테이션
    
    7) PathVariable : 
    
    핸들러 메서드의 파라미터 종류 중 하나, 괄호 안에 입력한 문자열 값은 `@GetMapping("/{member-id}")`처럼 중괄호({ }) 안의 문자열과 동일해야 함,
    
    두 문자열이 다르다면 `MissingPathVariableException`이 발생
    
    7) **GetMember** 
    
    getMember() 메서드는 특정 회원의 정보를 클라이언트 쪽에 제공하는 핸들러 메서드
    
    7) **GetMembers** 
    
    - 회원 목록을 클라이언트에게 제공하는 핸들러 메서드
    - `@GetMapping` 에는 별도의 URI를 지정해주지 않았기 때문에 클래스 레벨의 URI(“/v1/members”)에 매핑
    
2. OrderController
   
    **✔️ 요청에 필요한 주문(Order) 정보**
    
    - 회원 식별자: **memberId**
    - 커피 식별자: **coffeeId**
    
    ```java
    package com.springboot.order;
    
    import org.springframework.http.MediaType;
    import org.springframework.web.bind.annotation.*;
    
    @RestController
    @RequestMapping(value = "/v1/orders", produces = MediaType.APPLICATION_JSON_VALUE)
    public class OrderController {
        @PostMapping
        public String postOrder(@RequestParam("memberId") long memberId,
                                @RequestParam("coffeeId") long coffeeId) {
            System.out.println("# memberId: " + memberId);
            System.out.println("# coffeeId: " + coffeeId);
    
            String response =
                    "{\\"" +
                        "memberId\\":\\""+memberId+"\\"," +
                        "\\"coffeeId\\":\\""+coffeeId+"\\"" +
                    "}";
            return response;
        }
    
        @GetMapping("/{order-id}")
        public String getOrder(@PathVariable("order-id") long orderId) {
            System.out.println("# orderId: " + orderId);
    
            // not implementation
            return null;
        }
    
        @GetMapping
        public String getOrders() {
            System.out.println("# get Orders");
    
            // not implementation
            return null;
        }
    }
    
    ```
    
    1) postOrder() 메서드는 회원 고객이 주문한 커피 주문 정보를 등록해 주는 핸들러 메서드
    
    - 고객이 주문한 커피에 필요한 주문 정보는 **어떤 고객이 어떤 커피를 주문했느냐 하는 것**
    - ‘**어떤 고객**’에 해당하는 정보가 회원 식별자(memberId)
    - ‘**어떤 커피**’에 해당하는 정보가 바로 **커피 식별자**(coffeeId)
    

### 개선 포인트

1. 제일 불편한 것 response 
   
    ⇒ JSON 문자열 수작업을 Map 객체로 대체하여 개선 가능
    
    ⇒ 리턴 값을 ResponseEntity 객체로 변경
            Ex) return new ResponseEntity<>(HttpStatus.OK);
    
2. `@RequestParam` 애너테이션을 사용한 요청 파라미터 수신
   
    ⇒ 요청 파라미터들이 다섯 개라면 총 다섯 개의  @RequestParameter를 사용해서 파라미터로 입력
    

### 2 - ResponseEntity 적용

기존 1의 레거시 코드에서 가장 불편했던 부분을 개선한 코드

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/v1/members") // (1) produces 설정 제거됨
public class MemberController {
    @PostMapping
    public ResponseEntity postMember(@RequestParam("email") String email,
                                     @RequestParam("name") String name,
                                     @RequestParam("phone") String phone) {
        // (2) JSON 문자열 수작업을 Map 객체로 대체
        Map<String, String> map = new HashMap<>();
        map.put("email", email);
        map.put("name", name);
        map.put("phone", phone);

        // (3) 리턴 값을 ResponseEntity 객체로 변경
        return new ResponseEntity<>(map, HttpStatus.CREATED);
    }

    @GetMapping("/{member-id}")
    public ResponseEntity getMember(@PathVariable("member-id") long memberId) {
        System.out.println("# memberId: " + memberId);

        // not implementation

        // (4) 리턴 값을 ResponseEntity 객체로 변경
        return new ResponseEntity<>(HttpStatus.OK);
    }

    @GetMapping
    public ResponseEntity getMembers() {
        System.out.println("# get Members");

        // not implementation

        // (5) 리턴 값을 ResponseEntity 객체로 변경
        return new ResponseEntity<>(HttpStatus.OK);
    }
}

```

1.  **`@RequestMapping`의 ‘produces’ 의 attribute 사라짐.**
2. **JSON 문자열을 개발자가 직접 수작업으로 작성하던 부분이 `Map` 객체로 대체**
   
    → 이를 통해  `@RequestMapping`의 ‘produces’ attribute를 생략가능해짐
    
    → Map 객체를 리턴하게 되면 내부적으로 ‘이 데이터는 JSON 형식의 응답 데이터로 변환해야 되는구나’라고 이해하고 JSON 형식으로 자동 변환해줌
    
3. **리턴 값으로 JSON 문자열을 리턴하던 부분이 ResponseEntity 객체를 리턴**
    - new ResponseEntity<>(map, HttpStatus.CREATED);처럼 **ResponseEntity** 객체를 생성하면서 생성자 파라미터로 응답 데이터(map)와 **HTTP 응답 상태**를 함께 전달
    - HTTP 응답 상태를 명시적으로 함께 전달하면 **클라이언트의 요청을 서버가 어떻게 처리했는지를 쉽게 알 수 있음**
    - ResponseEntity 참고링크 : [https://itvillage.tistory.com/44](https://itvillage.tistory.com/44)
4. (4), (5) getMember(), getMembers() 핸들러 메서드 역시 ResponseEntity 객체를 리턴하는 걸로 수정하였으며, HttpStatus.OK 응답 상태를 전달하도록 수정
5. Postman 결과 - `postMember()` 핸들러 메서드에 요청
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9248654c-5325-44e2-8afe-5627fcf9c83c" width=500/>
    
    - POST Method 형식의 클라이언트 요청에 대한 응답 상태는 `HttpStatus.OK`보다는 `HttpStatus.CREATED`가 조금 더 자연스러
    

## HTTP 헤더(Header)

- HTTP 메시지(Messages)의 구성 요소 중 하나로써 클라이언트의 요청이나 서버의 응답에 포함되어 부가적인 정보를 HTTP 메시지에 포함할 수 있도록 함.
- 개발자가 헤더를 건드릴 일은 많이 없지만, 코드 레벨에서 직접 컨트롤 해야 할 경우가 있음
1. Authorization
    1. 클라이언트가 적절한 자격 증명을 가지고 있는지를 확인하기 위한 정보
    2.  REST API 기반 애플리케이션의 경우 클라이언트와 서버 간의 로그인(사용자 ID/비밀번호) 인증에 통과한 클라이언트들은 ‘Authorization’ 헤더 정보를 기준으로 인증에 통과한 클라이언트가 맞는지 확인하는 절차를 거침
2. User-Agent
    1. 데스크톱, 노트북. 스마트폰, 태블릿 등 들어오는 요청을 구분해서 응답 데이터를 다르게 보내줘야 하는 경우가 있는데 구분해서 처리할 수 있게 해줌.
    2. 특히 갤럭시 폴드 같은 경우는 화면 크기를 미리 체크해서 내보내야 함.
    

### HTTP Request 헤더 정보 얻기

Spring MVC는 아래와 같이 HTTP 헤더 정보를 읽어오는 몇 가지 방법을 제공하고 있음

1. @RequestHeader로 개별 헤더 정보 받기
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/f7b0203d-e0db-41f7-af95-772d95bd5026" width=500/>
    
2. `@RequestHeader`로 전체 헤더 정보 받기
   
    ```java
    import com.codestates.section3.week1.api.mvc_examples.common.Member;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;
    
    import java.util.Map;
    
    @RestController
    @RequestMapping(path = "/v1/members")
    public class MemberController {
        @PostMapping
        public ResponseEntity postMember(@RequestHeader Map<String, String> headers,//(1)
                                         @RequestParam("email") String email,
                                         @RequestParam("name") String name,
                                         @RequestParam("phone") String phone) {
            for (Map.Entry<String, String> entry : headers.entrySet()) {
                System.out.println("key: " + entry.getKey() +
                        ", value: " + entry.getValue());
            }
    
            return new ResponseEntity<>(new Member(email, name, phone),
                    HttpStatus.CREATED);
        }
    }
    
    //출력결과
    key: user-agent, value: PostmanRuntime/7.29.0
    key: accept, value: */*
    key: cache-control, value: no-cache
    key: postman-token, value: 6082ccc2-3195-4726-84ed-6a2009cbae95
    key: host, value: localhost:8080
    key: accept-encoding, value: gzip, deflate, br
    key: connection, value: keep-alive
    key: content-type, value: application/x-www-form-urlencoded
    key: content-length, value: 54
    
    ```
    
3. HttpServletRequest 객체로 헤더정보 얻기
   
    ```java
    @RestController
    @RequestMapping(path = "/v1/orders")
    public class OrderController {
        @PostMapping
        public ResponseEntity postOrder(HttpServletRequest httpServletRequest,//(1)
                                        @RequestParam("memberId") long memberId,
                                        @RequestParam("coffeeId") long coffeeId) {
            System.out.println("user-agent: " + httpServletRequest.getHeader("user-agent"));
    
            return new ResponseEntity<>(new Order(memberId, coffeeId),
                    HttpStatus.CREATED);
        }
    }
    
    ```
    
    - 잘 안 씀
    - 이 코드가 들어오는 순간 http웹환경에 종속되어버려서 가능한 피하는 것이 좋음
4. `HttpEntity` 객체로 헤더 정보 얻기
   
    ```java
    @RestController
    @RequestMapping(path = "/v1/coffees")
    public class CoffeeController{
        @PostMapping
        public ResponseEntity postCoffee(@RequestHeader("user-agent") String userAgent,//(1)
                                         @RequestParam("korName") String korName,
                                         @RequestParam("engName") String engName,
                                         @RequestParam("price") int price) {
            System.out.println("user-agent: " + userAgent);
            return new ResponseEntity<>(new Coffee(korName, engName, price),
                    HttpStatus.CREATED);
        }
    
        @GetMapping
        public ResponseEntity getCoffees(**HttpEntity httpEntity**) {
            for(Map.Entry<String, List<String>> entry : httpEntity.getHeaders().entrySet()){
                System.out.println("key: " + entry.getKey()
                        + ", " + "value: " + entry.getValue());
            }
    
            System.out.println("host: " + httpEntity.getHeaders().getHost());
            return null;
        }
    }
    
    //출력결
    key: user-agent, value: [PostmanRuntime/7.29.0]
    key: accept, value: [*/*]
    key: cache-control, value: [no-cache]
    key: postman-token, value: [368ad61b-b196-4f75-9222-b9a5af750414]
    key: host, value: [localhost:8080]
    key: accept-encoding, value: [gzip, deflate, br]
    key: connection, value: [keep-alive]
    host: localhost:8080
    ```
    
    - HttpEntity는 Request 헤더와 바디 정보를 래핑하고 있으며, 조금 더 쉽게 헤더와 바디에 접근할 수 있는 다양한 API를 지원
    

### HTTP Response 헤더 정보 추가

1. `ResponseEntity`와 `HttpHeaders`를 이용해 헤더 정보 추가
   
    ```java
    @RestController
    @RequestMapping(path = "/v1/members")
    public class MemberController{
        @PostMapping
        public ResponseEntity postMember(@RequestParam("email") String email,
                                         @RequestParam("name") String name,
                                         @RequestParam("phone") String phone) {
            // (1) 위치 정보를 헤더에 추가
            HttpHeaders headers = new HttpHeaders();
            headers.set("Client-Geo-Location", "Korea,Seoul");
    
            return new ResponseEntity<>(new Member(email, name, phone), headers,
                    HttpStatus.CREATED);
        }
    }
    ```
    
    - HttpHeaders의 set() 메서드를 이용해서 헤더 정보를 추가하는 것
2. `HttpServletResponse` 객체로 헤더 정보 추가
   
    ```java
    @RestController
    @RequestMapping(path = "/v1/members")
    public class MemberController{
        @GetMapping
        public ResponseEntity getMembers(HttpServletResponse response) {
            response.addHeader("Client-Geo-Location", "Korea,Seoul");
    
            return null;
        }
    }
    
    ```
    
    > HttpServletRequest와 HttpServletResponse는 저수준(Low Level)의 서블릿 API를 사용할 수 있기 때문에 복잡한 HTTP Request/Response를 처리하는 데 사용가능
    > 
    > 
    > 반면에 ResponseEntity나 HttpHeaders는 Spring에서 지원하는 고수준(High Level) API로써 간단한 HTTP Request/Response 처리를 빠르게 진행가능
    > 
    > **복잡한 처리가 아니라면 코드의 간결성이나 생산성 면에서 가급적 Spring에서 지원하는 고수준 API를 사용하길 권장**
    > 

## Rest Client

### 클라이언트(Client)와 서버(Server)의 관계

​	<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/23febf5c-ca00-4ffd-9221-5c127ecd7955" width=400/>

1. **웹 브라우저**는 웹 서버로부터 HTML 콘텐츠를 제공받는 **클라이언트**
   
    즉, 서버 쪽의 리소스(Resource, 자원)를 이용하는 측이 클라이언트
    
2. **서버도 다른 서버로부터 리소스를 제공받아야 하는 경우가 굉장히 많음**
   
    ⇒ 대표적인 예 : Frontend 서버와 Backend 서버의 관계
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/8dcf275c-1ca4-479d-89f3-46deb391dc2a" width=500/>
    
    **클라이언트와 서버의 관계는 상대적**
    
    - 브라우저 입장에서는 Frontend의 리소스를 제공받기 때문에 명백한 클라이언트
    - **Frontend가 Backend에 동적인 데이터를 요청하게 된다면** 클라이언트가 됨
    
3. 백엔드 서버
    - Backend 서버 내부적으로 다른 서버에게 HTTP 요청을 전송해서 작업을 나누어 처리하는 경우가 굉장히 많음.
    - backend A가 Frontend에게 리소스를 제공해 주기 때문에 서버 역할을 하지만 Backend A에서 Backend B의 리소스를 다시 이용하기 때문에 Backend B의 리소스를 이용할 때는 Backend A도 클라이언트의 역할을 하게 됨.
4. 어떤 서버가 HTTP 통신을 통해서 다른 서버의 리소스를 이용한다면 그때만큼은 클라이언트의 역할을 하는 것을 기억하기

### Rest Client란?

- **Rest API 서버에 HTTP 요청을 보낼 수 있는 클라이언트 툴 또는 라이브러리**를 의미
- Postman은 UI가 갖춰진 Rest Client라 볼 수 있음
- **RestTemplate**
    - Spring에서 제공하는 RestClient API
    - HTTP Client 라이브러리 중 하나를 이용해서 원격지에 있는 다른 Backend 서버에 HTTP 요청을 보낼 수 있음
    - Rest 엔드 포인트 지정, 헤더 설정, 파라미터 및 body 설정을 한 줄의 코드로 손쉽게 전송 가능
    - java.net.HttpURLConnection, Apache HttpComponents, OkHttp 3, Netty 같은 HTTP Client 라이브러리 중 하나를 유연하게 사용 가능

### Rest Template

1. 객체 생성
   
    ```java
    public class RestClientExample01 {
        public static void main(String[] args) {
            // (1) 객체 생성
            RestTemplate restTemplate =
                    new RestTemplate(new HttpComponentsClientHttpRequestFactory());
        }
    }
    
    ```
    
    - `RestTemplate`의 생성자 파라미터로 HTTP Client 라이브러리의 구현 객체를 전달
    - `HttpComponentsClientHttpRequestFactory` 클래스를 통해 Apache HttpComponents를 전달
        - Apache HttpComponents를 사용하기 위해서는 builde.gradle의 dependencies 항목에 아래와 같이 의존 라이브러리를 추가해야 함
        
        ```java
        dependencies {
            ...
            ...
            implementation 'org.apache.httpcomponents:httpclient'
        }
        
        ```
    
2. URI 생성
   
    객체 생성 되었다면 HTTP Request를 전송할 Rest 엔드포인트의 URI를 지정해 주어야 함
    
    ```java
    import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
    import org.springframework.web.client.RestTemplate;
    import org.springframework.web.util.UriComponents;
    import org.springframework.web.util.UriComponentsBuilder;
    
    import java.net.URI;
    
    public class RestClientExample01 {
        public static void main(String[] args) {
            // (1) 객체 생성
            RestTemplate restTemplate =
                    new RestTemplate(new HttpComponentsClientHttpRequestFactory());
    
            // (2) URI 생성
            UriComponents uriComponents =
                    UriComponentsBuilder
                            .newInstance()
                            .scheme("http")
                            .host("worldtimeapi.org")
    //                        .port(80)
                            .path("/api/timezone/{continents}/{city}")
                            .encode()
                            .build();
            URI uri = uriComponents.expand("Asia", "Seoul").toUri();
        }
    }
    
    ```
    
    - UriComponentsBuilder 클래스에서 제공하는 API 메서드의 기능
        - `newInstance()`
            - `UriComponentsBuilder` 객체를 생성합니다.
        - `scheme()`
            - URI의 scheme을 설정합니다.
        - `host()`
            - 호스트 정보를 입력합니다.
        - `port()`
            - 디폴트 값은 80이므로 80 포트를 사용하는 호스트라면 생략 가능
        - `path()`
            - URI의 경로(path)를 입력합니다.
            - [코드 3-18]에서는 URI의 path에서 {continents}, {city}의 두 개의 템플릿 변수를 사용하고 있음
            - 두 개의 템플릿 변수는 `uriComponents.expand("Asia", "Seoul").toUri();`에서 `expand()` 메서드 파라미터의 문자열로 채워짐
            - 즉, 빌드 타임에 {continents}는 ‘Asia’, {city}는 ‘Seoul’로 변환됨
        - `encode()`
            - URI에 사용된 템플릿 변수들을 인코딩해줍니다.
            - 여기서 인코딩의 의미는 non-ASCII 문자와 URI에 적절하지 않은 문자를 Percent Encoding 한다는 의미
        - `build()`
            - `UriComponents` 객체를 생성
        
    - `UriComponents`에 사용된 API 메서드
        - `expand()`
            - 파라미터로 입력한 값을 URI 템플릿 변수의 값으로 대체
        - `toUri()`
            - `URI` 객체를 생성
    
3. 요청 전송
    1. **`getForObject()`를 이용한 문자열 응답 데이터 전달받기**
    
       ```java
       public class RestClientExample01 {
           public static void main(String[] args) {
               // (1) 객체 생성
               RestTemplate restTemplate =
                       new RestTemplate(new HttpComponentsClientHttpRequestFactory());
       
               // (2) URI 생성
               UriComponents uriComponents =
                       UriComponentsBuilder
                               .newInstance()
                               .scheme("http")
                               .host("worldtimeapi.org")
       //                        .port(80)
                               .path("/api/timezone/{continents}/{city}")
                               .encode()
                               .build();
               URI uri = uriComponents.expand("Asia", "Seoul").toUri();
       
               // (3) Request 전송
               String result = restTemplate.getForObject(uri, String.class);
       
               System.out.println(result);
           }
       }
       
       ```
    
       `getForObject(URI uri, Class<T> responseType)`
    
       - 기능 설명
    
         - `getForObject()` 메서드는 HTTP Get 요청을 통해 서버의 리소스를 조회
    
       - 파라미터 설명
    
         - `URI uri`
           - Request를 전송할 엔드포인트의 URI 객체를 지정
         - `Class<T> responseType`
           - 응답으로 전달받을 클래스의 타입을 지정
           - 코드 3-19에서는 응답 데이터를 문자열로 받을 수 있도록 `String.class`로 지정
    
       - 실행결과
    
         ```java
         abbreviation: KST
         client_ip: 125.129.191.130
         datetime: 2022-04-28T09:49:44.492621+09:00
         day_of_week: 4
         day_of_year: 118
         dst: false
         dst_from:
         dst_offset: 0
         dst_until:
         raw_offset: 32400
         timezone: Asia/Seoul
         unixtime: 1651106984
         utc_datetime: 2022-04-28T00:49:44.492621+00:00
         utc_offset: +09:00
         week_number: 17
         
         ```
    2. `getForObject()`**를 이용한 커스텀 클래스 타입으로 원하는 정보만 응답으로 전달받기**
    
       ```java
       public class RestClientExample02 {
           public static void main(String[] args) {
               // (1) 객체 생성
               RestTemplate restTemplate =
                       new RestTemplate(new HttpComponentsClientHttpRequestFactory());
       
               // (2) URI 생성
               UriComponents uriComponents =
                       UriComponentsBuilder
                               .newInstance()
                               .scheme("http")
                               .host("worldtimeapi.org")
       //                        .port(80)
                               .path("/api/timezone/{continents}/{city}")
                               .encode()
                               .build();
               URI uri = uriComponents.expand("Asia", "Seoul").toUri();
       
               // (3) Request 전송. WorldTime 클래스로 응답 데이터를 전달받는다.
               WorldTime worldTime = restTemplate.getForObject(uri, WorldTime.class);
       
               System.out.println("# datatime: " + worldTime.getDatetime());
               System.out.println("# timezone: " + worldTime.getTimezone());
               System.out.println("# day_of_week: " + worldTime.getDay_of_week());
           }
       }
       
       ```
    
       [응답데이터 받기 위한 `WorldTime` 클래스]
    
       ```java
       public class WorldTime {
           private String datetime;
           private String timezone;
           private int day_of_week;
       
           public String getDatetime() {
               return datetime;
           }
       
           public String getTimezone() {
               return timezone;
           }
       
           public int getDay_of_week() {
               return day_of_week;
           }
       }
       
       ```
    
       [실행결과]
    
       ```java
       # datatime: 2021-10-10T11:39:15.099207+09:00
       # timezone: Asia/Seoul
       # day_of_week: 4
       ```
    
    3. **`getForEntity()`를 사용한 Response Body(바디, 콘텐츠) + Header(헤더) 정보 전달받기**
    
       ```java
       public class RestClientExample02 {
           public static void main(String[] args) {
               // (1) 객체 생성
               RestTemplate restTemplate =
                       new RestTemplate(new HttpComponentsClientHttpRequestFactory());
       
               // (2) URI 생성
               UriComponents uriComponents =
                       UriComponentsBuilder
                               .newInstance()
                               .scheme("http")
                               .host("worldtimeapi.org")
       //                        .port(80)
                               .path("/api/timezone/{continents}/{city}")
                               .encode()
                               .build();
               URI uri = uriComponents.expand("Asia", "Seoul").toUri();
       
               // (3) Request 전송. ResponseEntity로 헤더와 바디 정보를 모두 전달받을 수 있다.
               ResponseEntity<WorldTime> response =
                       restTemplate.getForEntity(uri, WorldTime.class);
       
               System.out.println("# datatime: " + response.getBody().getDatetime());
               System.out.println("# timezone: " + response.getBody().getTimezone()());
               System.out.println("# day_of_week: " + response.getBody().getDay_of_week());
               System.out.println("# HTTP Status Code: " + response.getStatusCode());
               System.out.println("# HTTP Status Value: " + response.getStatusCodeValue());
               System.out.println("# Content Type: " + response.getHeaders().getContentType());
               System.out.println(response.getHeaders().entrySet());
           }
       }
       ```
    
        `getForEntity()` 메서드를 사용해서 헤더 정보와 바디 정보를 모두 전달받고있음
    
    4. **`exchange()`를 사용한 응답 데이터 받기**
    
       ```java
       public class RestClientExample03 {
           public static void main(String[] args) {
               // (1) 객체 생성
               RestTemplate restTemplate =
                       new RestTemplate(new HttpComponentsClientHttpRequestFactory());
       
               // (2) URI 생성
               UriComponents uriComponents =
                       UriComponentsBuilder
                               .newInstance()
                               .scheme("http")
                               .host("worldtimeapi.org")
       //                        .port(80)
                               .path("/api/timezone/{continents}/{city}")
                               .encode()
                               .build();
               URI uri = uriComponents.expand("Asia", "Seoul").toUri();
       
               // (3) Request 전송. exchange()를 사용한 일반화 된 방식
               ResponseEntity<WorldTime> response =
                       restTemplate.exchange(uri,
                               HttpMethod.GET,
                               null,
                               WorldTime.class);
       
               System.out.println("# datatime: " + response.getBody().getDatetime());
               System.out.println("# timezone: " + response.getBody().getTimezone());
               System.out.println("# day_of_week: " + response.getBody().getDay_of_week());
               System.out.println("# HTTP Status Code: " + response.getStatusCode());
               System.out.println("# HTTP Status Value: " + response.getStatusCodeValue());
           }
       }
       
       ```
    
       `exchange(URI uri, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType)`
    
       - 기능 설명
         - `getForObject()`, `getForEntity()` 등과 달리 `exchange()` 메서드는 HTTP Method, RequestEntity, ResponseEntity를 직접 지정해서 HTTP Request를 전송할 수 있는 가장 일반적인 방식
       - 파라미터 설명
         - `URI url`
           - Request를 전송할 엔드포인트의 URI 객체를 지정
         - `HttpMethod method`
           - HTTP Method 타입을 지정
         - `HttpEntity<?> requestEntity`
           - HttpEntity 객체를 지정
           - HttpEntity 객체를 통해 헤더 및 바디, 파라미터 등을 설정가능
         - `Class<T> responseType`
           - 응답으로 전달받을 클래스의 타입을 지정

<br/>

 ### comment

배운듯 안배운듯 생소한 느낌, 
이거 아는데 싶다가도 모르겠다...



<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<div class="notice" markdown="1">
🍒 `공지` 
<h4> - <u>정보 공유가 아닌 개인 공부 기록용입니다.</u></h4>
</div>