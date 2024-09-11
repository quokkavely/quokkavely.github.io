---
layout : single
title : "[Project] "NonUniqueResultException"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---


오늘도 코드 만지다가 머리가 띵해졌다. 에러 메시지 보는데 `javax.persistence.NonUniqueResultException: query did not return a unique result: 2`라는 에러가 떠서 도대체 이게 뭔가 싶었다.

이게 무슨 상황이냐면, 다른 팀원이 짠 주문번호 생성 코드를 쓰는 도중에 발생한 문제다. 

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

이 코드 자체는 괜찮아 보이는데, 문제는 여기서 발생한 `NonUniqueResultException`이다.</br>
<img src="https://github.com/user-attachments/assets/3bc73035-67b3-4808-9765-6a12bd5a92ae" width=500/>

무슨 말이냐면, `findTopByOrderCdStartingWithOrderByOrderCdDesc(date)` 이 쿼리가 원래 하나의 결과만 반환해야 하는데, 두 개 이상의 결과가 반환됐다는 뜻이다.

내가 Postman으로 연속해서 두 번의 POST 요청을 보냈는데, 주문번호 생성 부분에서만 자꾸 에러가 터지길래 주문번호 생성하는 부분을 주석 처리해보니까 예외가 안 터지는 거다. 이걸 보고 문제는 확실히 주문번호 생성 부분에 있다고 생각하게 됐다.
<img src="https://github.com/user-attachments/assets/e95c8a08-08e4-4a37-ac5a-b1d6117f97bb" width=500/>

### 문제의 원인

주문번호를 생성할 때, 같은 날에 여러 주문이 들어오면 그날 날짜로 시작하는 주문번호들이 DB에 여러 개 쌓이게 된다. 근데 여기서 `findTopByOrderCdStartingWithOrderByOrderCdDesc(date)`가 그 날짜로 시작하는 가장 최근의 주문번호를 찾는 쿼리다 보니, 같은 날짜에 중복된 결과가 여러 개 나올 수 있는 상황이었다.

결국 쿼리가 여러 결과를 반환하고, 그걸 하나로 묶지 못해서 에러가 발생한 것 같다.

### 해결 방법

일단 주문번호 생성할 때 중복을 피하기 위해 쿼리부터 수정해야 할 것 같다. `findTopByOrderCdStartingWithOrderByOrderCdDesc()` 이 부분에서 가장 최근 하나의 주문번호만 정확하게 반환하도록 쿼리 조건을 조금 더 강화해야 한다.

또 하나, 동시성 문제도 고려해야 할 것 같다. 내가 연속으로 두 번 POST 요청을 보내는 상황이라면 동시에 여러 주문이 들어올 수 있기 때문에, 주문번호 생성 로직에 동기화를 추가하거나, DB에서 순차적으로 주문번호를 관리하는 방법도 필요할 것 같다. 예를 들어, `synchronized` 키워드를 사용해서 주문번호 생성할 때 한 번에 하나의 쓰레드만 접근할 수 있도록 하거나, DB에서 고유한 주문번호를 관리하는 테이블을 따로 만들어서 그걸 참조하는 방법도 고려할 수 있을 것 같다.

### 회고

에러 메시지가 처음엔 되게 생소했는데, 결국엔 쿼리에서 중복된 결과가 나와서 발생한 문제였다. `NonUniqueResultException`을 처음 겪어봤는데, 앞으로 같은 날짜에 여러 주문이 들어올 상황을 고려해서 쿼리를 짤 때는 좀 더 꼼꼼하게 설계해야겠다는 생각이 들었다. 그리고 동시성 문제도 미리 대비해야겠다는 교훈을 얻었다. 역시 개발은 삽질과 배움의 연속이다.