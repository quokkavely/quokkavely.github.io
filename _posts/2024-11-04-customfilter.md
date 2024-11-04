---
layout : single
title : "[MSA] chapter 3_Spring Cloud Gateway :Custom Filter"
categories: MSA
tag : [MSA, SpringCloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

인프런 Dowon Lee님의 Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의를 듣고 정리한 필기입니다.😊<br>
[Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의 들으러 가기👩‍🏫](https://inf.run/GHeRm)
{: .notice--warning}


## gateway 실습 1 :  application.yml에서 설정

### 1. **커스텀 필터 구현 및 등록**

Spring Cloud Gateway에서 사용자 정의 필터(Custom Filter)를 통해 특정 조건에 맞는 요청을 필터링하고, 로그 기록이나 인증 처리 같은 기능을 추가할 수 있다. `CustomFilter` 클래스는 반드시 `AbstractGatewayFilterFactory`를 상속받아야 하며, `apply` 메서드를 오버라이드하여 원하는 작업을 정의한다.

apply라는 메서드를 정의해서 구현 → 작동하고자 하는 내용을 기술하면 된다. ( GatewayFilter)라는 것을 반환시켜 줌으로써 어떤 작업을 할 것인지 정의할 수 있다)

```java
@Component
@Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {
    public CustomFilter() {
        super(Config.class);
        //Configuration 정보가 있다면 자기 자신의 클래스 안에서 Config라는 내부클래스를 매개변수로 등록
    }

    @Override
    public GatewayFilter apply(CustomFilter.Config config) {
        // 1 Custom Pre Filter. Suppose we can extract JWT and perform Authentication
        return ((exchange, chain) -> {
            
            // 2
            // Netty를 사용하기 때문에 비동기 방식이고
            // 비동기 방식일때는 request, response 를 대신해서 사용한다.
            ServletHttpRequest request = exchange.getRequest();
            ServletHttpResponse response = exchange.getResponse();
            
            log.info("Custom PRE filter: request uri -> {}", request.getId());
            //Custom Post Filter.Suppose we can call error response handler based on error code.
            //어떤 필터가 적용될 건지 지정할 수 있는데
            // WebFlux라는 기능을 이용해서 서버구축할 때는 반환 값으로 MonoDataType을 사용할 수 있다. 
            return chain.filter(exchange).then(Mono.fromRunnable() -> {
                log.info("Custom POST filter: response code -> {}", response.getStatusCode());
            }));
        }
    }
}
```

- **프리필터**: `apply` 메서드 내부의 `request` 객체에서 URI를 가져와 로그로 출력.
- **포스트필터**: `chain.filter(exchange).then`을 사용해, 응답이 반환될 때 상태 코드를 로그로 출력.

### 2. **Configuration 설정**

필요에 따라 `Configuration` 정보가 있으면 `Config`라는 내부 클래스를 통해 설정을 전달할 수 있다.

### 3. **필터 설정과 활용**

- **프리필터와 포스트필터**: 요청이나 응답 처리 과정에서 추가 작업이 필요할 때 필터를 설정해 활용할 수 있다. 예를 들어, 프리필터에서 사용자 인증 정보를 확인하고, 포스트필터에서는 응답에 로그를 남기는 방식으로 서버의 요청 흐름을 파악할 수 있다.
- **ID 출력과 로그 기록**: 필터를 통해 요청 ID를 출력하거나, 인증 상태를 로그로 기록해 디버깅에 유용하게 활용 가능하다.

### 4. **서버 설정: Netty**

이전까지 tomcat을 사용했지만 지금 사용하는 것은 Netty라는 내장서버를 사용한다.

- **Netty**: 비동기 방식의 내장 서버로, 서버 리소스 관리가 효율적이며 비동기 HTTP 요청을 지원한다. Tomcat과 같은 동기 방식의 서버와는 다른 방식으로 request와 response 객체를 다룬다.
- **서버 response객체 사용**: Netty에서는 비동기 처리를 위해 서블릿 요청 방식 대신 HTTP 리퀘스트와 리스판스를 객체로 다룬다. 이로 인해 서버의 동시 처리 성능이 향상될 수 있다.

### 5. **JWT 인증을 활용한 인증 처리**

- **JWT 토큰 인증**: 사용자가 로그인하면 JWT(JSON Web Token)를 발급받아 프리필터에서 이를 확인한다. 이후 각 요청마다 이 토큰을 통해 인증 및 인가가 이루어진다.
- **게이트웨이 연동**: 게이트웨이 앞단에서 인증 정보를 전달하여 각 서비스가 요청을 처리할 때 필요한 보안성을 확보한다.

### 6. **YAML 설정 추가**

yml 파일에는 filters에 CustomFilter 를 추가한다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f7511b05-f7b2-446f-8b14-0575185a1028/e057cf0e-9af2-45e5-a67d-399f16f73879/2f46b934-cc81-4f6c-a191-ad71bf97e033.png)

### 7. **테스트와 검증**

- **서비스 컨트롤러에 체크 메서드 등록**: 첫 번째와 두 번째 서비스 컨트롤러에 체크 메서드를 추가해 필터가 정상적으로 작동하는지 검증한다.
- **로그 확인**: 프리필터와 포스트필터가 요청과 응답을 처리하는 순서를 로그로 남겨, 필터가 올바르게 작동하고 있는지 확인한다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f7511b05-f7b2-446f-8b14-0575185a1028/2794879c-f48c-4383-825b-650a0234f99f/3b858a14-9348-42a5-a994-d1d1121953c6.png)

<br>
<br>
<br>
<br>