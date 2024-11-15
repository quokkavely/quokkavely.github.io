---
layout : single
title : "[MSA] Feign Client 예외 처리 : try-catch"
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

# Feign Client 예외 처리 정리

Feign Client 사용 중 API 통신에서 발생하는 다양한 예외 상황에 대해 다음과 같이 정리했다.

## 1. 잘못된 엔드포인트로 요청했을 때 발생하는 문제

API 요청 시 특정 서비스의 경로(endpoint)가 다른 부서나 벤더에 의해 변경되거나 잘못 전달된 경우, 예상치 못한 에러가 발생할 수 있다.

<img src= "https://github.com/user-attachments/assets/927eec2e-fed0-4c9e-8bd3-c96cd08443aa" width = 500/>

- 예를 들어, `order-service`의 경로가 변경되어 실제로 존재하지 않는 경로로 요청이 들어가면, 서버에서는 `404 Not Found` 오류를 반환하지만, 클라이언트 쪽에서는 `500 Internal Server Error`로 잘못 인식될 수 있다. 이는 경로 오류이므로, 500보다는 404 에러가 더 적절하다.
    
   <img src= "https://github.com/user-attachments/assets/192f2181-f891-4148-b3b0-19ddf257a672" width = 500/>
    

### 예외 처리 코드

```java

// Feign Client를 통해 API 호출
List<ResponseOrder> ordersList = null;
try {
    ordersList = orderServiceClient.getOrders(userId);
} catch (FeignException ex) {
    log.error(ex.getMessage());
}

```

- 이 코드에서는 FeignClient 호출 시 발생하는 `FeignException`을 try-catch로 감싸 로그를 기록함으로써 문제 발생 시 원인을 확인할 수 있도록 했다.
- 문제가 없는 경우에는 데이터가 출력되고, 문제 발생 부분은 데이터가 반영되지 않도록 조정했다.

## 2. UserService에 로깅 추가

<img src= "https://github.com/user-attachments/assets/43ef47fd-e8d7-4e88-9b2e-f62139269140" width = 500/>

로깅을 통해 오류를 기록하고 추적할 수 있다. UserService에 로깅을 추가하여 문제 상황을 파악하고 기록하는 방법을 설정했다.

1. **UserServiceApplication에 로깅 메서드 추가 후 빈 등록**
    - UserServiceApplication에 로깅을 위한 메서드를 추가하고 빈으로 등록해 로깅을 쉽게 사용할 수 있도록 했다.
        
        <img src= "https://github.com/user-attachments/assets/c100432c-a9af-4934-8af4-faad5299643b" width = 500/>
        
2. **OrderServiceClient 경로 수정 테스트**
    - `OrderServiceClient`의 기존 경로에서 잘못된 경로로 수정하여 예외 발생 상황을 재현했다.
    
    ```java
    @FeignClient(name = "order-service")
    public interface OrderServiceClient {
        @GetMapping("/order-service/{userId}/orders_ng") // 잘못된 경로로 수정
        List<ResponseOrder> getOrders(@PathVariable String userId);
    }
    ```
    

## 3. 예외 처리 적용 후 테스트

1. 사용자 등록
    
    <img src= "https://github.com/user-attachments/assets/bd0af75a-e24a-4969-8762-c11480e4039f" width = 500/>
    
    - userId 복사

2. userId를 넣어 사용자 정보 조회
    
    <img src= "https://github.com/user-attachments/assets/47a62c29-4b7b-4f96-9a90-4e12b8dcbe4f" width = 500/>
    
    - 사용자 등록 후 해당 `userId`로 주문 조회를 수행하면, 잘못된 경로로 인해 `404 Not Found`가 발생하지만 `500 Internal Server Error`가 반환되는 것을 확인할 수 있다.

3. log 출력 확인
    
    <img src= "https://github.com/user-attachments/assets/59b4d2b4-bbd0-4157-a8ea-e7b93134bdd6" width = 500/>
    
    - 로그를 통해 order-service  feignClient 에서 order-service/사용자id/orders_ng를 잘못된 경로로 호출된 것을 확인 할 수 있다.

### 예외 처리 코드 개선

잘못된 경로로 인해 예외가 발생하는 경우, 예외를 캐치하고 로그에 기록한 뒤 기본적으로 빈 리스트를 반환하도록 조정하여 오류가 발생한 부분만 비워지도록 했다.

```java
// Feign Exception handling

List<ResponseOrder> ordersList = null;
try {
    ordersList = orderServiceClient.getOrders(userId);
} catch (FeignException ex) {
    log.error("Failed to fetch orders for user {}: {}", userId, ex.getMessage());
    ordersList = new ArrayList<>(); // 빈 리스트로 처리
}

userDto.setOrders(ordersList);
return userDto;

```

## 4. 테스트 결과

1. 잘못된 경로로 요청을 보내면 `404 Not Found`가 서버에서 발생하며, 예외를 캐치하여 빈 리스트를 반환함으로써 에러가 생긴 주문 정보는 비워지고, 정상적인 사용자 정보는 출력된다.
2. 최종적으로 주문 조회 요청 시 200 OK를 반환하나 주문 정보가 출력되지 않는 상황을 확인할 수 있다.이 방식으로 Feign Client에서의 예외 상황을 효과적으로 처리할 수 있다.
    - 주문 등록
    
    <img src= "https://github.com/user-attachments/assets/b6237e10-0c6a-4ad8-9d6f-cdb286e71b79" width = 500/>
    
    - 주문 등록 후 주문 조회 시 200 OK를 반환하나 주문 정보가 출력되지 않는 상황
    
    <img src= "https://github.com/user-attachments/assets/f86504ea-551f-46d1-a627-a766cfb0cb09" width = 500/>


---
비슷한 상황으로 프로젝트 할 때 잘 되고 있던 feign 요청이 안된 적이 있었는데 employee 담당하던 팀원이 갑자기 endpoint를 변경해서 몇 시간을 고생한 적이 생각났다.<br>
그때는 반환 타입도 달라지는 바람에 더 문제를 해결하는데 오래 걸렸지만, 이런 예외처리와 log를 자세히 남겼더라면 조금 수월하게 해결했을 것 같다는 생각이 들었다.<br>
이 방법 외에 다른 방법도 있는 것 같아서 더 수강해본 다음에 프로젝트에 적용해봐야겠다.