---
layout : single
title : "[Project] 중복된 Sale History 저장 문제 해결"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## 주문 수정 시 중복된 Sale History 저장 문제 해결기

이번 프로젝트에서 **PATCH 요청**을 처리하면서 또 하나의 난관에 부딪혔다. 주문을 수정하는 요청(PATCH)을 보내면 수정된 데이터가 없을 때도 **200 OK** 응답이 뜨면서, 변경사항이 없는데도 불구하고 **Sale History**에 중복된 데이터가 저장되는 문제를 발견했다.

### 문제 상황: 변경 사항이 없어도 Sale History가 저장됨

주문 내역을 수정하는 요청을 보낼 때마다 실제로 변경된 데이터가 없더라도 **Sale History**에 중복된 기록이 저장되는 상황이 발생했다. 심지어 **Sale History**에 저장된 기록을 확인해보면, 변경된 내역이 없음에도 동일한 주문에 대해 **여러 개의 중복된 히스토리**가 계속해서 저장되고 있었다.

```json

{
    "data": [
        {
            "saleHistoryId": 6,
            "orderId": 2,
            "orderStatus": "REQUEST_TEMP",
            "saleHistoryItems": [...]
        },
        {
            "saleHistoryId": 5,
            "orderId": 2,
            "orderStatus": "REQUEST_TEMP",
            "saleHistoryItems": [...]
        },
        {
            "saleHistoryId": 2,
            "orderId": 2,
            "orderStatus": "REQUEST_TEMP",
            "saleHistoryItems": [...]
        }
    ],
    "pageInfo": {
        "totalElements": 3,
        "totalPage": 1
    }
}

```

위 JSON 예시처럼 동일한 주문(`orderId: 2`)에 대해 여러 번 중복된 Sale History가 저장되고 있었는데, 이로 인해 데이터가 계속 불필요하게 누적되는 문제가 발생했다.

### 문제 원인: 저장 순서의 문제

코드를 살펴보니 문제의 원인은 **저장 순서**에 있었다. 내가 작성한 코드는 **Sale History**를 먼저 저장한 후에 **Order**를 저장하고 있었는데, 이렇게 하면 실제로 **Order**가 수정되기 전에 Sale History가 먼저 생성되기 때문에 **실제 변경 사항이 반영되지 않은 채** 히스토리가 저장되고 있었다.

```java
saleHistoryRepository.save(saleHistoryMapper.orderToSaleHistory(findOrder, member));

return orderHeadersRepository.save(findOrder);
```

이 코드에서는 `findOrder`를 **OrderHeadersRepository**에 저장하기 전에 **SaleHistoryRepository**에 먼저 저장하고 있었기 때문에, **변경 사항이 없는 주문**도 Sale History에 저장되면서 중복 데이터가 발생하고 있었다

### 해결책: 저장 순서 변경

해결 방법은 생각보다 간단했다. **Order**가 먼저 저장된 후에 **Sale History**가 저장되도록 순서를 변경하면, 실제로 변경된 사항이 있을 때만 Sale History가 기록되게 할 수 있다.

변경 후 코드는 다음과 같다:

```java
orderHeadersRepository.save(findOrder);  // 먼저 Order를 저장
saleHistoryRepository.save(saleHistoryMapper.orderToSaleHistory(findOrder, member));  // 그 후 Sale History 저장

return findOrder;
```

이렇게 **Order**가 먼저 저장된 후에 Sale History를 저장하게 되면, **변경 사항이 없는 주문**은 Sale History에 기록되지 않기 때문에 중복된 히스토리가 저장되는 문제를 해결할 수 있었다.

### 결론

이번 이슈는 **저장 순서의 중요성**을 다시 한 번 깨닫게 해준 사례였다. 특히 주문 내역과 그에 따른 이력을 관리하는 시스템에서는 **변경 사항이 없을 때 불필요한 데이터가 저장되지 않도록** 세심하게 처리하는 것이 중요하다.

이번 문제를 해결하면서 얻은 교훈은:

1. **저장 순서**를 신중하게 설정해야 한다는 것.
2. 데이터베이스의 **정합성**을 유지하려면, 먼저 변경 사항을 확정한 후 이력 데이터를 기록해야 한다는 것.

이 문제를 해결하고 나니, 이제 **중복된 Sale History 문제**는 발생하지 않으며, 오직 실제로 변경된 내역만 기록되는 안정적인 시스템을 구축할 수 있었다. 프로젝트를 진행하면서 이런 디테일한 부분도 잘 챙겨야겠다는 생각이 들었다.