---
layout : single
title : "[MSA] api-gateway login 후 보호경로 에러 해결 과정 기록"
categories: MSA
tag : [MSA, SpringCloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

인프런 Dowon Lee님의 Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의를 듣고 정리한 내용입니다.😊<br>
[Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의 들으러 가기👩‍🏫](https://inf.run/GHeRm)
{: .notice--warning}

<br>

# api-gateway login 후 보호경로 에러 해결 과정 기록

- API Gateway를 통해 보호된 경로로 접근 시 `500 Internal Server Error`가 발생하며, 인증이 필요한 보호 경로에서 JWT 필터가 정상적으로 작동하지 않는 문제가 발생했다.
- `/auth/login` 등 공개 경로는 정상적으로 작동하지만, `/auth/employees/all` 등 보호된 경로는 필터에서 문제가 발생하는 것 같다.
    - api-gateway 로 요청 : `localhost:8000/auth/employees/all?page=1&size=10` 로 요청 시 `500 Internal Server Error` 발생
    - auth-api 로 요청 : `localhost:8081/employees/all?page=1&size=10` 로 요청시 200 OK 및 response data 정상 반환된다.

### 첫 시도

1. application.yml
    
    logging level 을 DEBUG → ERROR 로 변경
    
    ```java
    logging:
      level:
        org.springframework.cloud.gateway: ERROR
        org.springframework.cloud.gateway.route: ERROR
    ```
    

1. JwtFilter 클래스에서 로그 추가
    
    ```java
    @Component
    @RequiredArgsConstructor
    public class JwtFilter implements GatewayFilter {
    
        private final JwtUtils jwtUtils;
    
        /**
         * 토큰의 signature, expiration 등을 확인하는 필터
         * 요청이 일치하면 경로 재지정 수행
         * @param exchange, chain
         */
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            String url = exchange.getRequest().getURI().getPath();
            System.out.println("Requested URL: " + url);  // 요청 URL 확인
    
            if (!url.startsWith("/auth/")) {
                return chain.filter(exchange);
            }
    
            String authHeader = exchange.getRequest().getHeaders().getFirst(HttpHeaders.AUTHORIZATION);
            if (authHeader == null || !authHeader.startsWith("Bearer ")) {
                System.out.println("No valid authorization header");  // 헤더 유효성 확인
                return onError(exchange, "No valid token found", HttpStatus.UNAUTHORIZED);
            }
    
            String token = authHeader.substring(7);
            System.out.println("JWT Token: " + token);  // 토큰 값 확인
    
            try {
                if (jwtUtils.getValidation(token)) {
                    System.out.println("Invalid JWT token");  // 토큰 검증 실패 시 로그
                    return onError(exchange, "Invalid token", HttpStatus.UNAUTHORIZED);
                }
            } catch (Exception e) {
                System.out.println("Error validating token: " + e.getMessage());
                return onError(exchange, "Error validating token: " + e.getMessage(), HttpStatus.UNAUTHORIZED);
            }
    
            // 경로 재작성 Logic
            if (url.startsWith("/auth/")) {
                String[] strs = url.split("/");
                String newPath = "/" + String.join("/", Arrays.copyOfRange(strs, 2, strs.length));
                ServerHttpRequest newRequest = exchange.getRequest().mutate().path(newPath).build();
                return chain.filter(exchange.mutate().request(newRequest).build());
            }
    
            return chain.filter(exchange);
        }
    
        private Mono<Void> onError(ServerWebExchange exchange, String err, HttpStatus httpStatus) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(httpStatus);
            response.getHeaders().add("Content-Type", "application/json");
            DataBuffer buffer = response.bufferFactory().wrap(err.getBytes(StandardCharsets.UTF_8));
            return response.writeWith(Mono.just(buffer));
        }
    }
    
    ```
    

1. `JwtUtils` 클래스의 `getValidation` 메서드 예외 처리 강화
    - 기존
        
        ```java
            public boolean getValidation(String token) {
                return !validateToken(token) || isTokenExpired(token);
            }
        ```
        
    - 변경
        
        ```java
            public boolean getValidation(String token) {
                try {
                    boolean isValid = validateToken(token) && !isTokenExpired(token);
                    System.out.println("Token validation result: " + isValid); // 검증 결과를 로그에 기록
                    return !isValid; // 유효하지 않은 경우 true 반환
                } catch (Exception e) {
                    System.out.println("Token validation failed with exception: " + e.getMessage());
                    return true; // 검증 중 예외가 발생하면 유효하지 않은 것으로 간주
                }
            }
        }
        ```
        

1. 결과
    
    요청했고 500에러가 발생했지만, 아무런 log 도 발생하지 않는다.
    
    img src= <"https://github.com/user-attachments/assets/8bd76b04-d398-46ed-939a-d43d65bd6d4c" width = 500/>
    
    img src= <"https://github.com/user-attachments/assets/0fcb1250-20d8-4588-981a-05cfda15c465" width = 500/>
    

### 두 번째 시도

`JwtFilter`에서 추가한 로그가 출력되지 않는다면, **필터가 제대로 작동하지 않거나, `filter` 메서드가 호출되지 않는 경우**일 가능성이 높아서 아래와 같이 시도해 보았다.

1. application.yml 수정
    
    ```java
    logging:
      level:
        root: ERROR //추가
        org.springframework.cloud.gateway: ERROR
        org.springframework.cloud.gateway.route: ERROR
    ```
    
2. **`RouteLocatorConfig`**에서 보호된 경로 로그 남기기
    
    ```java
    .route("auth-api-protected", r -> r.path("/auth/**")
            .filters(f -> f.filter((exchange, chain) -> {
                System.out.println("Gateway Filter - Request Path: " + exchange.getRequest().getPath());
                return chain.filter(exchange);
            }).filter(jwtFilter)  // 기존 jwtFilter 추가
            .rewritePath("/auth/(?<segment>.*)", "/${segment}"))
            .uri("lb://AUTH-API"))
    ```
    
    - 근데 경로 문제는 아닌 것 같은 것은 우선,
        - api-gateway 로 요청 : localhost:8000/auth/login 으로 시도 할 때와
        - auth-api 로 요청 : localhost:8081/login 으로 시도할 때
        
        모두 200 OK 와 header에 token을 담아서 응답하기 때문이다.
        

전부 ERROR 로 변경했더니, 아무런 로그가 남지 않아서 모두 DEBUG 로 변경하고 테스트 진행해보았지만 해결되지 않아서 JwtFilter와 JwtUtils 에 모든 로그를 남기기로 했다.

### 세 번째 시도

요청이 들어왔을 때 진행이 어떻게 되고 어디서 문제가 발생하는지 파악하기 위해서 로그를  자세하게 남기기로 했다.

1. **JwtFilter에 log 추가**
    
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class JwtFilter implements GatewayFilter {
        private static final Logger logger = LoggerFactory.getLogger(JwtFilter.class);
        private final JwtUtils jwtUtils;
        
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            System.out.println("JwtFilter - Starting JWT validation.");
    
            try {
                String url = exchange.getRequest().getURI().getPath();
                System.out.println("Requested URL: " + url);  // 요청 URL 확인
    
                // 보호 경로가 아닌 경우 필터 통과
                if (!url.startsWith("/auth/")) {
                    System.out.println("Non-protected path, skipping JwtFilter.");
                    return chain.filter(exchange);
                }
    
                // Authorization 헤더 확인 및 토큰 추출
                String authHeader = exchange.getRequest().getHeaders().getFirst(HttpHeaders.AUTHORIZATION);
                if (authHeader == null || !authHeader.startsWith("Bearer ")) {
    		            //디버깅용 log
                    System.out.println("No valid authorization header");
                    return onError(exchange, "No valid token found", HttpStatus.UNAUTHORIZED);
                }
    
                String token = authHeader.substring(7);
                System.out.println("JWT Token: " + token);  // 토큰 값 확인
    
                // 토큰 유효성 검증
                boolean isValid = jwtUtils.getValidation(token);
                
                //디버깅용 log
                log.debug("Token validation result: {}", isValid);
                System.out.println("JWT Token Validation Result: " + isValid);
    
                if (isValid) {  // 검증 실패 시 응답 반환
                    return onError(exchange, "Invalid token", HttpStatus.UNAUTHORIZED);
                }
    
                System.out.println("JwtFilter - Authorization header processed successfully.");
    
                // 경로 재작성
                String newPath = url.replaceFirst("/auth", "");
                System.out.println("Rewritten path: " + newPath);  // 재작성된 경로 확인 log
                ServerHttpRequest newRequest = exchange.getRequest().mutate().path(newPath).build();
                return chain.filter(exchange.mutate().request(newRequest).build());
    
            } catch (Exception e) {
    		        //예외 발생시 log
                log.error("Exception in JwtFilter: ", e);
                System.out.println("JWT Filter Exception: " + e.getMessage()); 
                return onError(exchange, "Internal error in JwtFilter", HttpStatus.INTERNAL_SERVER_ERROR);
            }
        }
    
        private Mono<Void> onError(ServerWebExchange exchange, String err, HttpStatus httpStatus) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(httpStatus);
            response.getHeaders().add("Content-Type", "application/json");
    
         
            byte[] bytes = err.getBytes(StandardCharsets.UTF_8);
            DataBuffer buffer = response.bufferFactory().wrap(bytes);
    
            return response.writeWith(Mono.just(buffer))
                    .doOnError(error -> {
                        System.out.println("Error writing response: " + error.getMessage()); // log 
                        DataBufferUtils.release(buffer);  
                    })
                    .then(response.setComplete());
        }
    }
    
    ```
    
2. **JwtUtils 의 getValidation 메서드에 log 추가**
    
    ```java
      public boolean getValidation(String token) {
            try {
                boolean isValid = validateToken(token) && !isTokenExpired(token);
                System.out.println("Token validation result: " + isValid); // 검증 결과를 로그에 기록
                return !isValid; // 유효하지 않은 경우 true 반환
            } catch (Exception e) {
                System.out.println("Token validation failed with exception: " + e.getMessage());
                return true; // 검증 중 예외가 발생하면 유효하지 않은 것으로 간주
            }
        }
    ```
      
3. 결과 
    - 근데 테스트 하다가 authorization을 header에 담지 않았을 때는 콘솔에 아래와 같은 log가 출력되었다.
        
        ```groovy
        Gateway Filter - Request Path: /auth/employees/all
        JwtFilter - Starting JWT validation.
        Requested URL: /auth/employees/all
        No valid authorization header
        2024-11-14 00:00:52.258 TRACE 28616 --- [     parallel-7] o.s.c.g.filter.GatewayMetricsFilter      : spring.cloud.gateway.requests tags: [tag(httpMethod=POST),tag(httpStatusCode=401),tag(outcome=CLIENT_ERROR),tag(routeId=auth-api-protected),tag(routeUri=lb://AUTH-API),tag(status=UNAUTHORIZED)]
        ```
        
        img src= <"https://github.com/user-attachments/assets/bf46ee0d-62b5-457a-b358-9f994cccadb4" width = 500/>
        
        - token 체크를 해제하고 요청했을 때는 401 error 가 발생한다는 것을 확인할 수 있었다.
    - token 체크할 경우에는 해당 로그가 나오지 않는다.

### 네 번째 시도

- 우선 내가 토큰을 담든 안담든 auth-api에 도달이 안되고 있다. = 즉 auth-api에 아무런 로그가 남지 않고있다
- 토큰을 안담으면 api-gateway 실행콘솔에 log라도 나오는데 token 을 안담으면 api-gateway에서는 log도 나오지 않는다. (해당 log는 세 번째 시도 참고)
- 내가 내린 결론은 보호된 경로는 auth-api에 도달을 못하는 것 같다.
보호되지 않은 경로인 localhost:8000/auth/login에 post 요청 했을때는 auth-api에 도달하는 것을 확인했기 때문이다.

1. 그래서 **경로 재작성 필터를 제거**하고 test 진행 해보았다. 
    - 기존 코드
        
        ```java
        .route("auth-api-protected", r -> r.path("/auth/**")
                                .filters(f -> f.filter((exchange, chain) -> {
                                            System.out.println("Gateway Filter - Request Path: " + exchange.getRequest().getPath());
                                            return chain.filter(exchange);
                                        }).filter(jwtFilter)  // 기존 jwtFilter 추가
                                        .rewritePath("/auth/(?<segment>.*)", "/${segment}"))
                                .uri("lb://AUTH-API"))
        ```
        
    - 경로 재작성 필터 제거된 코드
        
        ```java
        .route("auth-api-protected", r -> r.path("/auth/**")
            .filters(f -> f.filter(jwtFilter))  // 경로 재작성 필터 제거
            .uri("lb://AUTH-API"))
        
        ```
        
    
    여전히 log도 남지않고, 500에러가 발생한다.
    
2. **경로 단순화**
    - test 1
        
        ```java
          @Bean
            public RouteLocator routeLocator(RouteLocatorBuilder builder, JwtFilter jwtFilter) {
                return builder.routes()
                        // Auth Service (8081)
                        .route("auth-api-protected", r -> r.path("/auth/**")
                                .uri("lb://AUTH-API"))  // 필터 없이 단순 라우팅만 
                ...
                
                ...
                }
           }
        ```
        
    - test 2
        
        ```java
            @Bean
            public RouteLocator routeLocator(RouteLocatorBuilder builder) {
                return builder.routes()
                        .route("auth-api-protected", r -> r.path("/auth/**")
                                .filters(f -> f.rewritePath("/auth/(?<segment>.*)", "/${segment}"))
                                .uri("http://localhost:8081"))  // 직접 포트 명시
                        .build();
            }
        ```
        
    - test 3
        
        ```java
        @Bean
        public RouteLocator routeLocator(RouteLocatorBuilder builder) {
            return builder.routes()
                .route("auth-api-protected", r -> r.path("/auth/**")
                    .uri("http://localhost:8081"))  // rewritePath 없이 단순 라우팅
                .build();
        }
        ```
        
    
    모두 실패했다, 그래도 알게 된 건 JwtUtils가 포함되지 않았을 때도 500 에러가 발생하기 때문에 JwtUtils의 문제가 아니라 라우팅 문제인 듯 하다.
    
3. 추가로
    
    ```java
    @Bean
    public RouteLocator routeLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("auth-api-public", r -> r.path("/auth/signup", "/auth/login")
                .uri("http://localhost:8081"))
            .route("auth-api-protected", r -> r.path("/auth/**")
                .uri("http://localhost:8081"))
            .build();
    }
    ```
    
    - uri("http://localhost:8081") 로 진행 했을 때는 auth/login 도 not found 가 발생했다.
    - .uri("lb://AUTH-API")) auth/login 로 요청 했을 때는 202 OK 응답 발생..

---

아무리 해도 안되서,, 일단 오늘은 여기서 마무리하기로 했다…강의에서 해당과정을 조금 더 상세하게 해줄 거라 생각햇는데, 그렇지 않아서 이 부분은 넘어가거나 다른 강의자료 또는 Stack Over Flow 사이트에 문의해봐야 할 것 같다.