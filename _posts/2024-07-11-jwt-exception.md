---
layout : single
title : "[Security] 예외처리"
categories: Spring
tag : [Spring, 실습, Security]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## Spring Security에서 발생한 예외 처리 필요성

### 로그인에 잘못 정보를 기입했을 때

<img src="https://github.com/user-attachments/assets/43ee6cad-7f32-4b68-9194-afa0445a98c7"/>

### 만료된 토큰을 넣었을때

<img src="https://github.com/user-attachments/assets/d8dc31e4-c795-4023-9375-27d63963511a"/>

<img src="https://github.com/user-attachments/assets/b2cca9e8-059e-4988-9cda-4984c3e7fe18"/>

### 권한이 부여되지 않은 리소스에 request를 전송

(Admin이 아닌 User가 답변을 시도)

<img src="https://github.com/user-attachments/assets/27ced158-f876-41e2-8f82-8fdcb87683ed"/>

<br/>
<br/>

Spring Security에서 처리되지 않은 예외나 그 외의 예외가 발생했을 때, <br/>
서블릿 컨테이너인 Tomcat이 HTTP 상태 코드에 따라 기본 오류 페이지를 반환하게 된다.

MVC에서 발생하는 예외와 security의 예외는 다른 곳에서 발생하기 때문에 <br/>
별도로 관리해주어야 한다! <br/>
Spring Security의 예외는 필터 체인 내에서 발생하기 때문에 이 부분에서 잡아주어야 한다. <br/>

## 예외처리

### 1️⃣ JwtVerificationFilter에 예외 처리 로직 추가

```java
public class JwtVerificationFilter extends OncePerRequestFilter {
    ...
    ...

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // (1)
        try {
            Map<String, Object> claims = verifyJws(request);
            setAuthenticationToContext(claims);
        } catch (SignatureException se) {
            request.setAttribute("exception", se);
        } catch (ExpiredJwtException ee) {
            request.setAttribute("exception", ee);
        } catch (Exception e) {
            request.setAttribute("exception", e);
        }

        filterChain.doFilter(request, response);
    }
```

- **try -catch문이 추가**
    
    → 특정 예외 타입의 Exception이 catch 되면 해당 Exception을 `request.setAttribute("exception", Exception 객체)`와 같이 HttpServletRequest의 애트리뷰트(Attribute)로 추가
    
1. 기존 예외 처리와는 다르게 Exception을 catch한 후에 Exception을 다시 throw 한다든지 하는 처리를 하지 않고, 단순히 request.setAttribute()를 설정
2. SecurityContext에 클라이언트의 인증 정보(Authentication 객체)가 저장되지 않게 하기 위함
3. SecurityContext에 클라이언트의 인증 정보(Authentication 객체)가 저장되지 않은 상태로 다음(next) Security Filter 로직을 수행하다 보면 결국에는 Filter 내부에서 `AuthenticationException`이 발생하게 되고, 이 `AuthenticationException`은 바로 아래에서 설명하는 **AuthenticationEntryPoint**가 처리하게 된다.

### 2️⃣ AuthenticationEntryPoint 구현

 AuthenticationEntryPoint는 `SignatureException`, `ExpiredJwtException` 등 Exception 발생으로 인해  SecurityContext에 Authentication이 저장되지 않을 경우 등 `AuthenticationException`이 발생할 때 호출되는 핸들러 같은 역할을 한다.

1. **MemberAuthenticationEntryPoint :**  AuthenticationEntryPoint 인터페이스를 구현하는 클래스
    
    ```java
    package com.springboot.auth.handler;
    ...
    ...
    
    @Slf4j
    @Component
    public class MemberAuthenticationEntryPoint implements AuthenticationEntryPoint {
        @Override
        public void commence(HttpServletRequest request, 
    										    HttpServletResponse response,
    											  AuthenticationException authException) 
    							throws IOException, ServletException {
    							
            Exception exception = (Exception) request.getAttribute("exception");
            ErrorResponder.sendErrorResponse(response, HttpStatus.UNAUTHORIZED);
    
            logExceptionMessage(authException, exception);
        
        }
    
        private void logExceptionMessage(AuthenticationException authException, 
    																	   Exception exception) {
    																	   
            String message = exception != null ? exception.getMessage() : authException.getMessage();
            log.warn("Unauthorized error happened: {}", message);
        }
    }
    ```
    
    - commence 메서드 : 인증되지 않은 사용자가 리소스에 접근하려 할 때 호출
        - request에 담겨져 있는 예외 객체를 가져 온다. (위의 JwtVerificationFilter의 doFilterInternal 메서드에서 예외 발생시 request.setAttribute("exception", se); 로 예외를 담았기 때문에 전에 예외가 발생했다면 담겨져 있다.
        - ErrorResponder.sendErrorResponse(response, HttpStatus.UNAUTHORIZED) → 에러 응답을 전송
    - logExceptionMessage 메서드
        - authException은 인증 예외, 두번째 파라미터의 exception은 추가 예외
        - String message = exception != null ? exception.getMessage() ⇒ 추가 예외가 있으면 그 메세지를 사용하고 , 인증 예외 메세지가 있다면 인증 예외를 사용한다

1. **ErrorResponder**
    
    `AuthenticationException` 발생하면 ErrorResponse를 생성해서 클라이언트에게 전송
    
    ```java
    package com.springboot.auth.utils;
    ...
    ...
    public class ErrorResponder {
        public static void sendErrorResponse(HttpServletResponse response,
                                             HttpStatus httpStatus)
                throws IOException {
    
            Gson gson = new Gson();
            ErrorResponse errorResponse = ErrorResponse.of(httpStatus);
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.setStatus(httpStatus.value());
            response.getWriter().write(gson.toJson(errorResponse,ErrorResponse.class));
        }
    ```
    
    - HttpStatus를 받아 ErrorREpnose 객체를 생성
    - response.setContentType(MediaType.APPLICATION_JSON_VALUE); ⇒ 응답을 json type으로 보내준다.
    - HTTP 응답에 JSON 형식의 에러 메시지를 작성하여 클라이언트에게 전송하는 역할을 한다.

### 3️⃣ AccessDeniedHandler 구현

인증에는 성공했지만 해당 리소스에 대한 권한이 없으면 호출되는 핸들러

```java
package com.springboot.auth.handler;
...

@Slf4j
@Component
public class MemberAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, 
                       HttpServletResponse response, 
                       AccessDeniedException accessDeniedException) 
            throws IOException, ServletException {

        ErrorResponder.sendErrorResponse(response, HttpStatus.FORBIDDEN);
        log.warn("Forbidden error happend: {}", accessDeniedException.getMessage());
    }
}
```

- `sendErrorResponse` 메소드는 HTTP 응답의 콘텐츠 타입을 `application/json`으로 설정하고, 상태 코드를 403으로 설정하며, `ErrorResponse` 객체를 JSON 문자열로 변환하여 응답 본문에 작성
- `accessDeniedException.getMessage()`는 접근 거부 예외의 상세 메시지를 반환한다.

### 4️⃣ SecurityConfiguration에 AuthenticationEntryPoint 및 AccessDeniedHandler 추가

```java
public class SecurityConfiguration {
   ...
   ...

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers().frameOptions().sameOrigin()
            .and()
            .csrf().disable()
            .cors(withDefaults())
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .formLogin().disable()
            .httpBasic().disable()
            .exceptionHandling()
            .authenticationEntryPoint(new MemberAuthenticationEntryPoint())  // (1) 추가
            .accessDeniedHandler(new MemberAccessDeniedHandler())            // (2) 추가
            .and()
            .apply(new CustomFilterConfigurer())
            .and()
            .authorizeHttpRequests(authorize -> authorize
                    .antMatchers(HttpMethod.POST, "/*/members").permitAll()
                    .antMatchers(HttpMethod.PATCH, "/*/members/**").hasRole("USER")
                    .antMatchers(HttpMethod.GET, "/*/members").hasRole("ADMIN")
                    .antMatchers(HttpMethod.GET, "/*/members/**").hasAnyRole("USER", "ADMIN")
                    .antMatchers(HttpMethod.DELETE, "/*/members/**").hasRole("USER")
                    .anyRequest().permitAll()
            );
        return http.build();
    }
    
    ...
    ...
    ...
    ...
    
    }
```

- exceptionHandling(): 예외 처리 설정을 구성
- `authenticationEntryPoint(new MemberAuthenticationEntryPoint())`: 인증 실패 시 처리할 핸들러 설정.
- `accessDeniedHandler(new MemberAccessDeniedHandler())`: 권한 부족 시 처리할 핸들러 설정.