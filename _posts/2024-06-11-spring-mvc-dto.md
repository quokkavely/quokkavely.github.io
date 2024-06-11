---
layout : single
title : "[Spring MVC] DTO(Data Transfer Object)"
categories: MVC
tag : [MVC,개념정리,Spring]
author_profile: true
---


<div class="notice" markdown="1">
🍒 `공지` 
<h4> - <u>정보 공유가 아닌 개인이 공부하고 기록하기 위한 용도입니다.</u></h4>
<h4> 잘못된 내용이 있을 수 있으니 참고부탁드립니다. </h4>
</div>



## HTTP 요청/응답에서의 DTO

### DTO 란?

- **Transfer**라는 의미에서 알 수 있듯이 데이터를 전송하기 위한 용도의 객체
- DTO는 주로 클라이언트에서 서버 쪽으로 전송하는 요청 데이터를 전달받을 때, 서버에서 클라이언트 쪽으로 전송하는 응답 데이터를 전송하기 위한 용도로 사용
- 레이어드 아키텍처의 기본흐름
    
    Client
    
    — ↓↑ —  **DTO(↓Request/Response↑)**
    
    - Request : 클라이언트 쪽에서 JSON 형식의 데이터를 서버 쪽으로 전송 ⇒ 서버에서 JSON 형식의 데이터를 DTO같은 JAVA객체로 변환=역직렬화**(Deserialization)**
    - Response : 서버쪽에서 client에게 응답데이터를 전송 , 응답데이터 java에서 JSON형식으로 변환 =직렬화 (Serialization)
    
    API (Controller)
    
    — ↓↑ — Entity (DB의 테이블과 1:1로 맵핑되는 객체)
    
    Service
    
    — ↓↑ — Entity
    
    Repository
    
    — ↓↑ — 
    
    DB
    

### DTO 사용이유

1. DTO 클래스를 이용한 코드의 간결성
    1. 회원정보를 등록할 때  정보가 많을 수록 `postMember()`에 파라미터로 추가되는 `@RequestParam`의 개수가 늘어남 ⇒ **DTO 클래스가 바로 요청 데이터를 하나의 객체로 전달받는 역할을 해줌.**
        
        **[기존코드]**
        
        ```java
        @RestController
        @RequestMapping("/v1/members")
        public class MemberController {
            @PostMapping
            public ResponseEntity postMember(@RequestParam("email") String email,
                                             @RequestParam("name") String name,
                                             @RequestParam("phone") String phone) {
                Map<String, String> map = new HashMap<>();
                map.put("email", email);
                map.put("name", name);
                map.put("phone", phone);
        
                return new ResponseEntity<Map>(map, HttpStatus.CREATED);
            }
        ```
        
        **[DTO]**
        
        ```java
        @RestController
        @RequestMapping("/v1/members")
        public class MemberController {
            @PostMapping
            public ResponseEntity postMember(MemberDto memberDto) {
                return new ResponseEntity<MemberDto>(memberDto, HttpStatus.CREATED);
            }
        ```
        
2. 데이터 유효성 검증의 단순화
    1. 서버 쪽에서 유효한 데이터를 전달받기 위해 데이터를 검증하는 것을 **유효성(Validation) 검증**
    2. HTTP 요청을 전달받는 핸들러 메서드는 요청을 전달받는 것이  주목적이기 때문에 최대한 간결하게 작성해야 함.
    3. **DTO 클래스를 사용하면 유효성 검증 로직을 DTO 클래스로 빼내어 핸들러 메서드의 간결함을 유지가능**
        
        
        **[기존 코드]**
        
        ```java
        @RestController
        @RequestMapping("/no-dto-validation/v1/members")
        public class MemberController {
            @PostMapping
            public ResponseEntity postMember(@RequestParam("email") String email,
                                             @RequestParam("name") String name,
                                             @RequestParam("phone") String phone) {
        				// (1) email 유효성 검증
                if (!email.matches("^[a-zA-Z0-9_!#$%&'\\\\*+/=?{|}~^.-]+@[a-zA-Z0-9.-]+$")) {
                    throw new InvalidParameterException();
                }
                Map<String, String> map = new HashMap<>();
                map.put("email", email);
                map.put("name", name);
                map.put("phone", phone);
        
                return new ResponseEntity<Map>(map, HttpStatus.CREATED);
            }
        		...
        		...
        }
        ```
        
        **[DTO]**
        
         DTO 클래스를 사용하면 유효성 검증 로직을 DTO 클래스로 빼내어 핸들러 메서드의 간결함을 유지가능 ⇒ Getter Setter로 불러오거나 , 하기 코드처럼 lombok 사용
        
        ```java
        import lombok.Getter;
        import lombok.Setter;
        
        @Getter @Setter
        public class MemberDto {
            @Email
            private String email;
            private String name;
            private String phone;
            }
        ```
        
        ```java
        @RestController
        @RequestMapping("/v1/members")
        public class MemberController {
            @PostMapping
            public ResponseEntity postMember(@Valid MemberDto memberDto) {
                return new ResponseEntity<MemberDto>(memberDto, HttpStatus.CREATED);
            }
        		...
        		...
        }
        
        ```
        
    
    > DTO 클래스 생성시 주의사항
    - 멤버 변수 이외에 getter 메서드가 있어야 함 ⇒ **Response Body에 해당 멤버 변수의 값이 포함되지 않는 문제가 발생**
    - getter & setter 생성 시 alt+insert 사용하여 간편하게 등록 가능.
    > 
    

### HTTP 요청/응답 데이터에 DTO 적용하기

⇒ 기존 코드에서 리팩토링하기.

1. HTTP Request Body가 JSON 형식
    1. **`@RequestBody` 애너테이션**
        1. JSON 형식의 Request Body를 MemberPostDto 클래스의 객체로 변환을 시켜주는 역할 (클라이언트 쪽에서 전송하는 Request Body는 JSON 형식이어야 한다)
    2. **`@ResponseBody` 애너테이션**
        1.  JSON 형식의 Response Body를 클라이언트에게 전달하기 위해 DTO 클래스의 객체를 Response Body로 변환하는 역할
        2. Spring MVC에서는 핸들러 메서드에  **`@ResponseBody`** 애너테이션이 붙거나 핸들러 메서드의 리턴 값이 `ResponseEntity`일 경우, 내부적으로 **`HttpMessageConverter`가 동작하게 되어 응답 객체(여기서는 DTO 클래스의 객체)를 JSON 형식으로 바꿔줌**

## DTO 유효성 검증

![Untitled](%5BSpring%20MVC%5D%20DTO(Data%20Transfer%20Object)%20f32eb9b837194a0cac8d1a71e9bc8b77/Untitled.png)

해당 회원가입 백엔드에서 고려해볼 만한 사항.

1. 이메일 검증
2. 이름 - 숫자X, 한국인만가능? -한글만 → annotation없음
3.  휴대전화 3개, 4개, 4개 숫자만 올 수 있게
4. 공백검증해야됨. → annotation 없음

### DTO 클래스에 유효성 검증 적용

1. 유효성 검증을 위한 의존 라이브러리 추가
    
    ```groovy
    dependencies {
    	implementation 'org.springframework.boot:spring-boot-starter-validation'
    	...
    	...
    }
    ```
    
2. 유효성 검증시 사용되는 Annotation
    
    
    | Annotation | Description | Null 허용 = null일 경우 유효성검증 x, not null이면 검증함 |
    | --- | --- | --- |
    | @NotBlank | ”” (공백) X, ” “(스페이스) X, 
    검증 실패시 에러메세지 출력됨 | x |
    | @NotNull | "", " "  둘 다 허용 | x |
    | @NotEmpty | "" (공백)X, “ “ (스페이스)허용 | x |
    | @Email | 유효한 이메일 주소인지를 검증 | x |
    | @Pattern(regexp=”~”) | 정규표현식(~안에 들어와야 함.),  | O |
    | @Min/ @Max | 지정된 값 이하/ 지정된 값 이상이어야 함. | O |
    | @Range(min = , max = ) | 값이 min 이상, max 이하여야 함(min과 max는 long 타입) |  |
    | @Length(min = , max = ) | 문자열의 길이가 min 이상, max 이하여야 함 |  |

1. 유효성 검증시 Controller Metnod에 작성해야 하는 것.
    
    ```groovy
    @RestController
    @RequestMapping("/v1/members")
    @Validated
    public class MemberController {
        ...
    		...
    
        @PatchMapping("/{member-id}")
        public ResponseEntity patchMember(@PathVariable("member-id") @Min(2) long memberId,
                                        @Valid @RequestBody MemberPatchDto memberPatchDto) {
            memberPatchDto.setMemberId(memberId);
    
            // No need Business logic
    
            return new ResponseEntity<>(memberPatchDto, HttpStatus.OK);
        }
    }
    ```
    
    - @Valid @RequestBody 는 꼭 작성해야 함. Valid 작성하지 않을 경우 유효성 검증이 이루어지지 않음.
    - `@PathVariable("member-id") long memberId` 변수는 URI path에 사용되는데 일반적으로 데이터 식별자는 0이상의 숫자이므로 @PathVariable("member-id") @Min(1) long memberId - 이런식으로 제약조건 걸 수 있음.
        
        → PathVariable에 유효성 검증이 정상적으로 수행되려면 클래스 레벨에 @Validated 붙여줘야 함.
        
2. Jakarta Bean Validation이란?
    1.  DTO 클래스의 유효성 검증을 위해서 사용한 애너테이션은 **Jakarta Bean Validation**이라는 유효성 검증을 위한 표준 스펙에서 지원하는 내장 애너테이션
    2. **Jakarta Bean Validation**은 라이브러리처럼 사용할 수 있는 API가 아닌 스펙(사양, Specification) 자체
    3. **Jakarta Bean Validation** 스펙을 구현한 구현체가 바로 **Hibernate Validator**
    
3. Custom Validator를 사용한 유효성 검증
    - 목적에 맞는 애너테이션이 없을경우 직접 만들어서 유효성 검증 가능.
    - 구현 과정
        1. Custom Validator를 사용하기 위한 Custom Annotation을 정의
        2. 정의한 Custom Annotation에 바인딩되는 Custom Validator를 구현
        3. 유효성 검증이 필요한 DTO 클래스의 멤버 변수에 Custom Annotation을 추가