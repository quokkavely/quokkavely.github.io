---
layout : single
title : "[MSA] FeignClient 예외처리 : ErrorDecoder"
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

# FeignClient 예외처리 : ErrorDecoder

이번에는 Feing 패키지에서 제공해주는 ErrorDecoder 라는 인터페이스를 구현하여eign Client에서 발생하는 에러를 효율적으로 처리하는 방법을 정리했다.

### ErrorDecoder

ErrorDecoder 인터페이스는 FeignClient 호출 시 발생한 에러를 상태 코드별로 구분해 처리할 수 있도록 지원한다.

이를 통해 더 세부적이고 체계적인 예외 처리가 가능하며, 상태 코드에 따라 커스텀 예외를 던질 수 있다.

<img src="https://github.com/user-attachments/assets/bf2464c2-bee8-4b49-b769-d6ba203ba000" width = 500/>

- `decode` 메서드를 구현하면 FeignClient 호출 시 발생한 오류에 대해 상태 코드별로 커스텀 로직을 작성할 수 있다.

### ErrorDecoder를 상속받는 클래스 생성

먼저, ErrorDecoder를 상속받는 클래스를 생성하고 상태 코드별로 로직을 작성한다.

<img src="https://github.com/user-attachments/assets/9740ff93-b3c2-4958-a464-3e43f424ad7d" width = 500/>

- 매개변수로 response 안에 포함되어있는 status 값으로 판단
- switch case 문자을 통해 400, 500, 404  오류에 대해서 체크
    - 404 오류에서 getOrders 메소드에서 발생했던 에러라면 그냥 메세지를 출력해서 사용자가 요청했던 주문이 없다고 표시를 나타낼 수 있다.
    - 둘다 아니라고 하면 default 문으로 예외 객체를 반환

### Application 클래스에 ErrorDecoder 빈 등록

빈으로 등록해야 메서드를 사용할 수 있기 때문에 application class 에 빈으로 등록 할 수 있다.

<img src="https://github.com/user-attachments/assets/9101fac4-5f11-4723-bf88-d057a62f9994" width = 400/>

### 기존 UserServiceImpl 코드 수정

- 지난번 예외처리를 위해 try~catch 문으로 변경했던 코드를 주석처리
    
    <img src="https://github.com/user-attachments/assets/261c2954-52e5-4193-8cf0-fe4818a02cfd" width = 300/>
    
    <img src="https://github.com/user-attachments/assets/5a5cba01-34ed-42ce-864f-6fd14b7ac35b" width = 300/>
    
- ErrorDecoder를 이용해서 error handling 하는 코드로 수정
    
    ```java
      /* ErrorDecoder */
            List<ResponseOrder> ordersList = orderServiceClient.getOrders(userId);
            userDto.setOrders(ordersList);
    
            return userDto;
    ```
    
- Test
    
    <img src="https://github.com/user-attachments/assets/6ded67a3-fc93-4c1d-91a9-2afc15b27799" width = 500/>
    
    <img src="https://github.com/user-attachments/assets/74a1008c-0503-4ac8-a1de-bf91c38b788b" width = 300/>
    

### 설정 파일에 등록하여 구현하는 방법

1. user-service.yml에서 예외 등록
    
    ```java
    order_service:
    	url: http://ORDER-SERVICE/order-service/%s/orders
    	exception:
    		order_is_empty: User's orders is empty
    ```
    
2. FeignErrorDecoder 에 생성자 주입 및 @Component 등록
    
    ErrorDecoder에서 환경 변수를 주입받아 메시지를 동적으로 설정한다
    
    ```java
    @Component
    public class FeignErrorDecoder implements ErrorDecoder {
        Environment env;
    
        @Autowired
        public FeignErrorDecoder(Environment env) {
            this.env = env;
        }
    
    ```
    
3. Component 로 등록했기 때문에 Application 클래스에 빈을 제거해야 한다.
    
    <img src="" width = 500/>
    

---

### 일반적인 try-catch와 ErrorDecoder의 차이점

- **try-catch 방식**
    - 간단한 예외 처리에는 적합하나, 코드가 복잡해지고 반복적이기 쉬움.
    - 모든 Feign 호출에 try-catch를 추가해야 하는 비효율이 발생.
- **ErrorDecoder 방식**
    - 상태 코드에 따라 중앙 집중식으로 예외 처리가 가능.
    - 상태 코드 기반 커스텀 예외 생성이 가능.
    - Feign과 자연스럽게 통합되며 유지보수가 간편.

---

📝 정리

ErrorDecoder는 Feign Client 호출 시 발생하는 다양한 에러를 체계적으로 처리하는 데 유용하다.

try-catch에 비해 코드가 간결하며, 상태 코드별로 세분화된 로직을 적용할 수 있어 추천하는 방식이다.

또한, 설정 파일을 통해 메시지를 관리하면 환경 변화에도 유연하게 대응할 수 있다.