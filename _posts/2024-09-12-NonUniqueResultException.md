---
layout : single
title : "[Project] "NonUniqueResultException 발생"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---


오늘 OrderService 부분 마무리하려고 테스트해보다가 머리가 띵해졌다. 에러 메시지 보는데 `javax.persistence.NonUniqueResultException: query did not return a unique result: 2`라는 에러가 떠서 도대체 이게 뭔가 싶었다. 

<Br>
원래 Order 관련 부분은 전부 내가 건드리다가, 프론트에서 item쪽을 급하게 손봐달라고 해서<br>
하는 도중에 Order에 기본 키가 아닌 주문번호가 별도로 필요할 것 같다고 했다 <br>
item이 생각보다 오래걸려서 다른 팀원에게 양해를 구했고 <br>
주문코드를 생성 후에 Post 요청이 들어오면 서비스층에서 코드 생성 후 set 해주기로 했는데 <br>
코드는 잘 만들어졌다고 했는데 post 요청을 여러 번 하면 500 예외가 발생한다고 해서 <br>
전에는 그런적이 없었는데 하고 이제 다시 Order 쪽 refactoring 후 test 해보니 NonUniqueResultException이 발생했다

<img src="https://github.com/user-attachments/assets/3bc73035-67b3-4808-9765-6a12bd5a92ae" width=500/>

 <br>
 <br>

<img src="https://github.com/user-attachments/assets/e95c8a08-08e4-4a37-ac5a-b1d6117f97bb" width=500/>
Postman으로 연속해서 두 번의 POST 요청을 보냈는데, 자꾸 에러가 터지길래 <br> 크게 달라진 점이 주문번호를 생성하는 부분이라서 중복이 왜 발생하지 하고 주문번호 생성하는 부분을 주석 처리해보니까 예외가 안 터졌다. <br> 이걸 보고 문제는 확실히 주문번호 생성 부분에 있다고 생각하게 됐다.<br>
<img src= "https://github.com/user-attachments/assets/e15de1e8-bbac-497d-ba2f-584120661910" width =500/>?

### 문제의 원인

쭉 읽어보니  `findTopByOrderCdStartingWithOrderByOrderCdDesc(date)` 이 쿼리가 원래 하나의 결과만 반환해야 하는데, 두 개 이상의 결과가 반환됐다는 것 같았다.

```java
// 주문 코드 생성 메서드
private String createOrderCd() {
    // 현재 날짜를 "yyMMMdd" 형식으로 변환 (MMM은 월의 영어 약자)
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyMMMdd", Locale.ENGLISH);
    String date = LocalDate.now().format(formatter).toUpperCase();

    // 현재 날짜에 해당하는 마지막 주문 코드를 DB에서 조회
    Optional<String> lastOrderCdOptional = orderHeadersRepository.findTopByOrderCdStartingWithOrderByOrderCdDesc(date);

    // 마지막 주문 코드가 없으면 00001부터 시작
    int newOrderNumber = 1;

    if (lastOrderCdOptional.isPresent()) {
        String lastOrderCd = lastOrderCdOptional.get();
        // 마지막 주문 코드에서 번호 부분 추출 (예: "24DEC1100001" -> 00001 추출)
        String lastOrderNumberStr = lastOrderCd.substring(7); // 날짜(6자리) 이후 숫자(5자리) 부분 추출
        int lastOrderNumber = Integer.parseInt(lastOrderNumberStr);

        // 새로운 주문 번호는 마지막 번호 + 1
        newOrderNumber = lastOrderNumber + 1;
    }

    // 새로운 주문 코드 생성 (예: 24DEC1100002)
    String newOrderCd = String.format("%s%05d", date, newOrderNumber);

    return newOrderCd;
}

```

주문번호를 생성할 때, 같은 날에 여러 주문이 들어오면 그날 날짜로 시작하는 주문번호들이 DB에 여러 개 쌓이게 된다. 근데 여기서 `findTopByOrderCdStartingWithOrderByOrderCdDesc(date)`가 그 날짜로 시작하는 가장 최근의 주문번호를 찾는 쿼리다 보니, 같은 날짜에 중복된 결과가 여러 개 나올 수 있는 상황이었다.

결국 쿼리가 여러 결과를 반환하고, 그걸 하나로 묶지 못해서 에러가 발생한 것 같다.

### 해결 방법

1. 주문번호 생성할 때 중복을 피하기 위해 쿼리부터 수정 필요 
    - `findTopByOrderCdStartingWithOrderByOrderCdDesc()` 이 부분에서 가장 최근 하나의 주문번호만 정확하게 반환하도록 쿼리 조건을 수정

2. 동시성 문제도 고려
    - 내가 연속으로 두 번 POST 요청을 보내는 상황이라면 동시에 여러 주문이 들어올 수 있기 때문에, 주문번호 생성 로직에 동기화를 추가하거나, 
    - DB에서 순차적으로 주문번호를 관리하는 방법도 필요
        - 예를 들어, `synchronized` 키워드를 사용해서 주문번호 생성할 때 한 번에 하나의 쓰레드만 접근할 수 있도록 하거나, 
        - DB에서 고유한 주문번호를 관리하는 테이블을 따로 만들어서 그걸 참조하는 방법도 고려할 수 있을 것 같다.
        <br>
        <Br>

    ```java
    
        private synchronized String createOrderCd() {
        }
    ```
    - 이렇게 synchronized를 붙이면 해당 메서드에 하나의 스레드만 접근할 수 있기 때문에 동시성을 막을 수 있다고 한다.
    
    - **Synchronized**
        - 자바에서 멀티스레드 환경에서 동시 접근을 제어하기 위해 사용하는 키워드
        - 해당 메서드나 블록에 여러 스레드가 동시에 접근하지 못하도록 해서 동시성 문제를 방지




3. 내가 개선한 방법
    -  위의 방법을 고집하려고 하니까 솔직히 주문번호 생성하는데 DB도 조회하는 등등의 비용이 너무 많이 든다는 생각이들어 그냥 간단하게 구현하기로 결정했다.
    -  랜덤 생성할 수 있는 방법을 찾았더니 randonUUID가 거의 중복이 없다고 해서 이 방법으로 결정했고
    -  기존의 주문코드 생성 방법처럼 회사의 생성규칙을 
    
    ```java
            // 주문 코드 생성 메서드
        private String createOrderCd() {
            String uuid = UUID.randomUUID().toString().replace("-", "").substring(0, 12).toUpperCase();

            return "SHO" + uuid;
        }
    ```
   
    단순 주문코드 생성인데 처음 보는 예외 발생 때문에 시간이 사르르륵 녹아버렸다.
    주문이 동시성 문제를 해결하기가 어렵다고 했는데 벌써 코드 하나 생성하는 것에서 만나서 당황스럽다..

<br>

