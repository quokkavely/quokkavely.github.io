---
layout : single
title : "[Spring] Hamcrest , SliceTest"
categories: Spring
tag : [Spring, 개념, JPA, Testing]
author_profile: true
---


📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


# Hamcrest

## Hamcrest를 사용하는 이유

- Assertion을 위한 매쳐(Matcher)가 자연스러운 문장으로 이어지므로 **가독성이 향상**된다.
- 테스트 실패 메시지를 이해하기 쉽다.
- 다양한 Matcher를 제공한다.

### 문법

1. **AssertThat**
    
    ```java
    import org.junit.jupiter.api.DisplayName;
    import org.junit.jupiter.api.Test;
    
    import static org.hamcrest.MatcherAssert.assertThat;
    import static org.hamcrest.Matchers.equalTo;
    import static org.hamcrest.Matchers.is;
    
    public class HelloHamcrestTest {
    
        @DisplayName("Hello Junit Test using hamcrest")
        @Test
        public void assertionTest1() {
            String expected = "Hello, JUnit";
            String actual = "Hello, JUnit";
    
            assertThat(actual, is(equalTo(expected)));  // (1)
        }
    }
    
    ```
    
    - assertThat(actual, is(equalTo(expected))); :  ‘결과 값(actual)이 기대 값(expected)과 같다는 것을 검증(Assertion)’
    - assertThat() 메서드의 파라미터
        - 첫 번째 파라미터 :  테스트 대상의 실제 결과 값
        - 두 번째 파라미터 : 기대하는 값입니다. 즉, 이런 값일 거라고 기대(예상)하는 값
2. **Not Null** 테스트 : Hamcrest의 `is()`, `notNullValue()` 매쳐를 함께 사용
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6ff4c785-2bbd-4af1-ba4d-cd7159e6c2b0" width=400/>

<br/>    
<br/>
<br/>  

# 슬라이스 테스트(Slice Test)

- 개발자가 각 계층에 구현해 놓은 기능들이 잘 동작하는지 특정 계층만 잘라서(Slice) 테스트하는 것을 슬라이스 테스트
- 애플리케이션의 특정 계층(예: 웹 계층, 서비스 계층 등)만을 테스트하기 위한 방법.
- 전체 애플리케이션을 실행하지 않기 때문에 더 빠르고 효율적이다.

<br/>
<br/>

## API 계층 테스트

### 기본 구조(with @SpringBootTest , @AutoConfigureMockMvc)

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6268f8a5-bdb7-4bab-9e4f-8db2799df07a" width=400/>

1. **`@SpringBootTest`** 
    - `@SpringBootTest` 애너테이션은 Spring Boot 기반의 애플리케이션을 테스트하기 위한 Application Context를 생성한다.
    - 스프링 부트 애플리케이션의 전체 컨텍스트를 로드하여 통합 테스트를 수행할 수 있도록 해줌 (모든 Bean을 로드하기 때문에 무거움)
    - 결국  해당 애너테이션이 붙으면 스프링부트의 도움을 받겠습니다  즉 ,스프링의 코어 기능을 쓰겠다는 뜻
        
        → 그래서 @Autowired 사용할 수 있는 것임. 
        
        → Autowired 는 필드 주입 : 필드 주입은 사용하지 말라고 하지만 테스트 환경에서는 많이 사용한다
        
2. **`@AutoConfigureMockMvc`**
    - 환경설정을 해주는것.( Controller 테스트를 위한 애플리케이션의 자동 구성 작업을 해준다.)
    - MockMVC를 자동으로 구성하여 테스트에서 사용할 수 있게 해준다
3. API 계층과 서블릿
    - 스프링부트는 서블릿 컨테이너라는 환경없이는 실행할 수없다, 왜냐면 서블릿이 요청과 응답을 담당하고 있기 때문에
    - 서블릿. 하나의 서블릿 객체(자바에서 웹 요청을 처리하는 객체)가 실행
    - 서블릿을 담당하는게 서블릿 컨테이너 (= tomcat)
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5a962a9b-acab-40dc-a81f-2bf8f8278cb8" width=450/>
        
    - 디스패처 서블릿 앞에 서블릿 컨테이너가 있음. was라고도하고 tomcat이라고도 함.
    - 웹의 요청을 수행하는 역할을 담당하는 것이 서블릿이고. 이걸 담당하는게 서블릿 컨테이너  (서블릿컨테이너의 대표적인 예 : 톰캣)
    - 결국 API계층에서 서블릿은 필수 , 얘를 관리하는 주체가 필요함
    - 슬라이스 테스트를 하기위해서는 서블릿 컨테이너가 필수로 필요함  →? 왜냐면 거기까지 보내주는 애가 없기 때문에.
    - MockMVC 는 가짜로 MVC의 구조를 나타냄, 실제로 서블릿 없이 API 계층에게 요청과 응답을 보내는 역할을 한다.
    - 테스트 할때마다 톰캣을 실행하면 매번 많은 리소스가 들어가게 되니 MockMVC를 사용하기 위해 주입이 필요함.
    - 그래서 환경설정이 필요해서 AutoConfigureMockMVC와 SpringBootTest 애너테이션이 필요하게 된다.
    <br/>
    <br/>
    ****
    - 🔎 참고
            
        **서블릿(Servlet)**
            
        - 자바에서 웹 요청을 처리하는 객체
        - 클라이언트의 HTTP 요청을 받아서 적절한 응답을 생성하는 역할을 합니다.
            
        **서블릿 컨테이너(Servlet Container)**
            
        - 서블릿을 관리하고 실행하는 환경
         - 대표적인 예로 Tomcat
        - 서블릿 컨테이너는 웹 서버와 서블릿 사이의 통신을 관리하며, 서블릿의 생명주기를 관리
            
         **DispatcherServlet**
            
        - 스프링 MVC의 핵심 서블릿으로, 모든 HTTP 요청을 받아 적절한 컨트롤러로 요청을 전달하고, 응답을 생성
            
        **MockMVC**
            
        - 서블릿 컨테이너를 실행하지 않고도 스프링 MVC의 동작을 테스트할 수 있는 도구
        - MockMVC를 사용하면 실제 서버를 실행하지 않고도 컨트롤러의 동작을 테스트할 수 있다.
    
    ****
    
4. **MokcMvc**
    - `MockMvc`는 Tomcat 같은 서버를 실행하지 않고 Spring 기반 애플리케이션의 Controller를 테스트할 수 있는 완벽한 환경을 지원해 주는 일종의 Spring MVC 테스트 프레임워크
    - MockMvc로 테스트 대상 Controller의 핸들러 메서드에 요청을 전송하기 위해서는 기본적으로 `perform()` 메서드를 먼저 호출 해야 함. (아래  postMember( ) 예제의 then 부분 참고)
    - MockMvcRequestBuilders 클래스를 이용해서 빌더 패턴을 통해 request 정보를 채워 넣을 수 있다.
    - MockMvc의 `perform()` 메서드가 리턴하는 `ResultActions` 타입의 객체를 이용해서 request에 대한 검증을 수행 가능

<Br/>

### MemberCotroller 테스트 → **HTTP Post request에 대한 테스트**

1. Gson 추가
        
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/2af45685-2f83-4c75-9313-df975466c6a7" width=500/>
        
2. MemberDto 클래스
        
    기존 Dto클래스는 다 나눠져 있었는데  하나의 dto클래스에  post/patch/response를 이너클래스로  통합해서 만들어 놓음
        
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5a844426-8c7b-4b49-b2ea-209977b29c5c" width=500/>
        
    
3.  MemberController의 postMember() 테스트
        
    ```java
    package com.springboot.member.controller;
    //...
    ...import com.google.gson.Gson;
    //...
        
    @SpringBootTest
    @AutoConfigureMockMvc
    class MemberControllerTest {
        @Autowired
        private MockMvc mockMvc;
        @Autowired
        private Gson gson;
        
        @Test
        void postMemberTest() throws Exception {
             //(1) given
            MemberDto.Post post = new MemberDto.Post
                    ("jerry@gmail.com", "박제리", "010-1111-1111");
            String content = gson.toJson(post);
                
            //(2) when
            //실제 api 요청을 보내는 것과 유사하게 테스트를 진행
            ResultActions actions = mockMvc.perform(
            //(3)  
        	        post("/v11/members")
                        .accept(MediaType.APPLICATION_JSON) //응답은 JSON
                        .contentType(MediaType.APPLICATION_JSON) 
                        .content(content)
                );
                //then
            actions.andExpect(status().isCreated())
                    .andExpect(header().string("Location",is(startsWith("/v11/members/"))));
            }
        }
    ```
        
    
    1️⃣ Given
    
    - Postman을 사용할 때 request body에 포함시키는 요청 데이터와 동일한 역할
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/429a5ab4-3c1e-4fca-8ba4-e260cbc94eee" width=400/>
        
    - Postman 사용시 JSON 포맷으로 바디에 포함시켰던 것처럼 Gson이라는 JSON 변환 라이브러리를 사용하기 위해 MemberDto.Post 객체를 JSON포맷으로 변환
    
    2️⃣ When
    
    - **perform()**메서드 호출 → MockMvc로 Controller의 핸들러메서드에 요청을 전송하기 위해서 필요함, perform 내에는 호출을 위한 세부적인 정보들이 포함된다.
    - (3)부터는 HTTP request에 대한 정보이며, MockMvcRequestBuilders 클래스를 이용해서 빌더 패턴을 통해 request 정보를 채워 넣을 수 있다.
    - **post()** 를 통해  HTTP POST METHOD와 request URL을 설정
    - **accept()** 메서드를 통해 클라이언트 쪽에서 리턴 받을 응답 데이터 타입으로 JSON 타입을 설정 (데이터는 JSON 타입으로 받을거야)
    - **contentType()** 메서드를 통해 서버 쪽에서 처리 가능한 Content Type으로 JSON 타입을 설정 (우리가 보내는 데이터는 JSON 타입으로 보낼거야)
    - **content()** 메서드를 통해 request body 데이터를 설정
    
    3️⃣ Then
    
    - MockMvc의 **perform()** 메서드는 **ResultActions 타입의 객체를 리턴**하는데, 이 ResultActions 객체를 이용해서 우리가 전송한 request에 대한 검증을 수행
    - **[ResultActions 인터페이스 →  andExpect라는 메서드를 통해 검증]**
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/979f8520-28b7-4de7-bf2c-70ec76f4f31e" width=500/>
        
        - andExpect() 메서드를 통해 파라미터로 입력한 매처(Matcher)로 예상되는 기대 결과를 검증, 해당 메서드는 throw로 예외도 함께 던져야 한다!
    - `status().isCreated()`를 통해 response status를 매치
    - (header().string("Location",is(startsWith("/v11/members/")) ⇒ HTTP header에 추가된 Location의 문자열 값이 `“/v11/members/”`로 시작하는지 검증
5. 실제 결과 
    - Body에 status 값 확인 해야 함. Postman사용할 때와 마찬가지로 @Valid 검증이 들어가서 오류가 뜨는 것.
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/35b1aef9-ddd6-4e73-8f44-0b4e29efdeb6" width=500/>
        
    - 테스트 실행시 서블릿 컨테이너에 대한 내용이 없음.
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/13afa694-b87d-425a-be3b-30c0ba06ff9a" width=500/>
        
    - cotroller의 테스트만 해야하는데 Hibernate의 도움을 받고 잇음.
        
       <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/01acd42b-980e-4c3a-bbf8-dacd249c0794" width=500/>
        
    
### MemberCotroller 테스트 → **HTTP get request에 대한 테스트**
    
 ```java
    @Test
    void getMemberTest() throws Exception {
    	     
     //Given , 앞서 작성했던 postMember()를 이용한 테스트 데이터 시작   
        MemberDto.Post post = new MemberDto.Post("jerry@gmail.com","wpfl","010-5555-5555");
        String content = gson.toJson(post);
        ResultActions postActions=  mockMvc.perform(post("/v11/members")
                .accept(MediaType.APPLICATION_JSON)
                .contentType(MediaType.APPLICATION_JSON)
                .content(content));
    
        postActions.andExpect(status().isCreated())
                .andExpect(header().string("Location", is(startsWith("/v11/members"))));
    
            
        //"v11/member/1"
        String location = postActions.andReturn().getResponse().getHeader("Location");
            
        //when then
        ResultActions response =  mockMvc.perform(get(location).accept(MediaType.APPLICATION_JSON));
        response.andExpect(status().isOk())
                .andExpect(jsonPath("$.data.email").value(post.getEmail()))
                .andExpect(jsonPath("$.data.name").value(post.getName()))
                .andExpect(jsonPath("$.data.phone").value(post.getPhone()));
    
        //get 요청을 보내야한다 db에 값을 가져올 수 있는지
    }}

```
    
1️⃣ **Given** 
    
- postMember()와 동일한 코드, 테스트 데이터를 백엔드 서버 측의 데이터베이스에 먼저 저장
- `postActions.andReturn().getResponse().getHeader("Location")`로 접근해서 Location header의 값을 얻어 올 수 있음
    
2️⃣ **When** 
    
- location header 값을 get의 URI로 전달
- Location header에서 얻게 되는 값이 given에서 등록한 회원정보의 위치를 의미 (`”/v11/members/1”`)
    
3️⃣ **Then**
    
- 기대하는 Http statusrk 200 OK인지 검증
- jsonPath()메서드를 통해 response body(JSON 형식)의 각 프로퍼티로 응답받는 데이터가 response body로 전송한 값과 일치하는지 검증
    
✔️ 테스트 결과
    
- 전부 테스트 하면 get만 통과함 왜냐면 post할 Member는 이미 존재하는 멤버이기 때문
        
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9b4c564f-8aa8-42e3-b6a9-5f5c16a96005" width=450/>
        
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/a00424c9-d114-4ab5-b86b-7a150a57624f" width=450/>
        
- @Transactional 추가시 모두 통과함. → 몇가지 문제가 있음.
        
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/47de4c73-ce36-4bac-a275-8f3821fde88f" width=450/>
        
    
**🤔 각각의 Test  케이스는 독립적인 역할을 해야하는데 원칙을 벗어나고 있음**
    
- 슬라이스테스트는 독립적으 컨트롤러만을 테스트해야되는데 DB도 접근하고 있고, DB에 접근하려면 서비스도 거쳐야 함.
- controller service repository까지 모두 구현되어야 테스트 가능  (Timely 불가)
- 사실상 슬라이스테스트라기 보다 서블릿을 사용하지 않는 통합테스트라고 볼 수 있음
    
✔️ 이러한 문제들은 **Mock(가짜) 객체를 사용해 계층 간의 연결을 끊어줌으로써 해결이 가능 →  다음에 배울 내용!**
    
✔️ `@AutoConfigureMockMvc` 말고 @webMvcTest 라는 것도 있는데 DI등 설정해줘야 할게 많아서 테스트 코드에는 사용하지 않음→ 문서화할 때 나중에 사용할 것.
    
   

## 데이터 액세스 계층 테스트

### 🔑 꼭 지키기 : 각각의 TestCase를 수행하고 나서는 롤백해야 함!

테스트 케이스는 여러 개의 테스트 케이스를 일괄적으로 실행시키더라도 각각의 테스트 케이스에 독립성이 보장되어야 한다.

즉, DB의 상태를 테스트 케이스 실행 이전으로 되돌려서 깨끗하게 만드는 것이 필요

### 예제 - MemberRepository 테스트

```java
package com.springboot.member.repository;

import com.springboot.member.entity.Member;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest //(1)
class MemberRepositoryTest {
    @Autowired
    private MemberRepository memberRepository;

    @Test
    public void saveMemberTest(){
        //given
        Member member = new Member();
        //초기화 할 때 Id = 0; 이 멤버의 id는 항상 0임
        //test 환경에서는 기본키는 검증하지 않는 것이 좋음.

        member.setEmail("jerry@gmail.comn=");
        member.setName("제리");
        member.setPhone("010-0430-0430");

        //When
        Member savedMember = memberRepository.save(member);

        //then
        assertNotNull(savedMember);
        assertEquals(member.getEmail(), savedMember.getEmail());
        assertEquals(member.getName(),savedMember.getName());
        assertEquals(member.getPhone(),savedMember.getPhone());
    }

    @Test
    public void findByEmailTest(){
        //given
        Member member  = new Member();
        member.setEmail("jerry@gmail.comn=");
        member.setName("제리");
        member.setPhone("010-0430-0430");

        //when
        memberRepository.save(member);
        Optional<Member> findMember = memberRepository.findByEmail(member.getEmail());
        //repository는 모두 Optional 타입이었음.

        //then
        assertTrue(findMember.isPresent());  //null이 아닌지를 검증
        assertEquals(findMember.get().getEmail(), member.getEmail());
        assertEquals(findMember.get().getPhone(),member.getPhone());
        assertEquals(findMember.get().getMemberStatus(), member.getMemberStatus());
    }
}
```

1.  (1) **@DataJpaTest** 
    
    데이터 액세스 계층에 필요한 자동 구성을 활성화 하는 기능
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c502b411-e61a-4007-8824-63d7fe8ab500" width=400/>
    
    - @Transactional 을 가지고 있어 하나의 테스트 케이스 실행이 종료되는 시점에 데이터베이스에 저장된 데이터는 rollback 처리된다. → 자동으로 롤백됨.
2. 테스트 대상 클래스인 `MemberRepository`를 DI 받는다.
3. given : 테스트 할 회원 정보를 준비 후  
4. when : repository에 저장
5. then :  회원 정보가 잘 저장되었는지 검증(Assertion)