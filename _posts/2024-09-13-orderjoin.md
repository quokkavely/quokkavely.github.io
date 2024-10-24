---
layout : single
title : "[Project] Order N+1 문제 해결: 쿼리 구조 변경"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## 중복된 Order 출력 문제 해결: 쿼리 구조 변경

이번에는 **주문(Order)** 조회 시 발생한 **중복 출력 문제**를 해결한 과정을 공유해보려고 한다. 

문제가 된 상황은 `localhost:8080/orders?page=1&size=10`로 요청을 보내면 **OrderItems의 개수만큼 OrderHeaders가 여러 번 중복**되어 출력되는 것이었다. 

이 문제를 해결하기 위해 **쿼리 구조**를 변경하고, 중복 문제를 해결하는 방법을 적용했다.

### 문제 상황: OrderItems 개수만큼 OrderHeaders가 중복 출력됨

- 현재의 API 호출 결과를 보면, 동일한 주문(`orderId`)이 **OrderItems**의 개수만큼 중복되어 반환되고 있다.<br> 예를 들어, 하나의 주문에 두 개의 아이템이 있을 경우, 동일한 주문이 두 번 출력되는 문제가 발생했다.
    
    ```json
    {
        "data": [
     {
                "orderId": 1,
                "employeeId": "TL001",
                "employeeNm": "팀장",
                "buyerCd": "B002",
                "buyerNm": "Sole Store",
                "orderCd": "SHOABB152367145",
                "createdAt": "2024-09-22T01:49:27.347086",
                "requestDate": "2024-09-25T18:00:00",
                "status": "REQUEST_TEMP",
                "orderItems": [
                    {
                        "orderItemId": 1,
                        "startDate": "2024-09-01T10:00:00",
                        "endDate": "2024-09-30T18:00:00",
                        "qty": 100,
                        "itemCd": "AD001",
                        "unit": "ea",
                        "unitPrice": 1000.50
                    },
                    {
                        "orderItemId": 2,
                        "startDate": "2024-09-02T12:00:00",
                        "endDate": "2024-09-30T17:00:00",
                        "qty": 150,
                        "itemCd": "AD002",
                        "unit": "ea",
                        "unitPrice": 3000.00
                    }
                ],
                "message": null
            },
            {
                "orderId": 1,
                "employeeId": "TL001",
                "employeeNm": "팀장",
                "buyerCd": "B002",
                "buyerNm": "Sole Store",
                "orderCd": "SHOABB152367145",
                "createdAt": "2024-09-22T01:49:27.347086",
                "requestDate": "2024-09-25T18:00:00",
                "status": "REQUEST_TEMP",
                "orderItems": [
                    {
                        "orderItemId": 1,
                        "startDate": "2024-09-01T10:00:00",
                        "endDate": "2024-09-30T18:00:00",
                        "qty": 100,
                        "itemCd": "AD001",
                        "unit": "ea",
                        "unitPrice": 1000.50
                    },
                    {
                        "orderItemId": 2,
                        "startDate": "2024-09-02T12:00:00",
                        "endDate": "2024-09-30T17:00:00",
                        "qty": 150,
                        "itemCd": "AD002",
                        "unit": "ea",
                        "unitPrice": 3000.00
                    }
                ],
                "message": null
            }
        ],
        "pageInfo": {
            "page": 1,
            "size": 10,
            "totalElements": 2,
            "totalPage": 1
        }
    }
    
    ```
<br>

### 문제 원인: JPA와 Join의 동작 방식

- 이 문제는 **OrderHeaders**와 **OrderItems** 간의 **1:N관계** 에서 발생하는 일반적인 문제다.
- join 을 사용하여 여러 개의 아이템을 함께 조회할 때, **N개의 아이템**이 있을 경우 **OrderHeaders가 N번 중복**되어 반환되게 된다.

<br>

### 해결 방법: `DISTINCT` 사용 및 `fetchJoin` 적용

- 이를 해결하기 위해 **쿼리 구조를 변경**하고, 중복을 제거하기 위한 두 가지 방법을 적용했다:
    
    ```java
    List<OrderHeaders> results = queryFactory
            .selectDistinct(orderHeaders)  // 중복된 OrderHeaders 제거
            .from(orderHeaders)
            .leftJoin(orderHeaders.buyer, buyer)
            .leftJoin(orderHeaders.orderItems, orderItems).fetchJoin()  // fetchJoin으로 관련 데이터를 한 번에 로드
            .where(builder)
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .orderBy(orderHeaders.createdAt.desc())
            .fetch();
    
    long total = queryFactory
            .select(orderHeaders.countDistinct())  // totalCount를 계산할 때도 DISTINCT 사용
            .from(orderHeaders)
            .leftJoin(orderHeaders.buyer, buyer)
            .leftJoin(orderHeaders.orderItems, orderItems)
            .where(builder)
            .fetchOne();
    
    ```
    
    1. **`selectDistinct(orderHeaders)`**:
        - `OrderHeaders` 테이블의 중복을 제거하기 위해 **DISTINCT**를 사용했다. 이 방법을 통해 **OrderItems** 개수와 상관없이 **OrderHeaders**가 한 번만 출력되도록 할 수 있다.
    2. **`fetchJoin`**:
        - `fetchJoin`을 사용해 **OrderHeaders**와 **OrderItems**를 한 번의 쿼리로 함께 로드한다. 이렇게 하면 **N+1 문제**도 방지되고, 성능 최적화에도 도움이 된다.

### 해결
```java
{
    "data": [
        {
            "orderId": 2,
            "employeeId": "TL001",
            "employeeNm": "팀장",
            "buyerCd": "B007",
            "buyerNm": "Kickstore",
            "orderCd": "SHOD58B08C771C7",
            "createdAt": "2024-09-13T01:49:27.676745",
            "requestDate": "2024-09-30T10:00:00",
            "status": "REQUEST_TEMP",
            "orderItems": [
                {
                    "orderItemId": 3,
                    "startDate": "2024-09-01T10:00:00",
                    "endDate": "2024-09-30T18:00:00",
                    "qty": 110,
                    "itemCd": "AD003",
                    "unit": "ea",
                    "unitPrice": 1000.50
                },
                {
                    "orderItemId": 4,
                    "startDate": "2024-09-02T12:00:00",
                    "endDate": "2024-09-30T17:00:00",
                    "qty": 50,
                    "itemCd": "AD004",
                    "unit": "ea",
                    "unitPrice": 3000.00
                }
            ],
            "message": null
        },
        {
            "orderId": 1,
            "employeeId": "TL001",
            "employeeNm": "팀장",
            "buyerCd": "B002",
            "buyerNm": "Sole Store",
            "orderCd": "SHOABB152367145",
            "createdAt": "2024-09-13T01:49:27.347086",
            "requestDate": "2024-09-25T18:00:00",
            "status": "REQUEST_TEMP",
            "orderItems": [
                {
                    "orderItemId": 1,
                    "startDate": "2024-09-01T10:00:00",
                    "endDate": "2024-09-30T18:00:00",
                    "qty": 100,
                    "itemCd": "AD001",
                    "unit": "ea",
                    "unitPrice": 1000.50
                },
                {
                    "orderItemId": 2,
                    "startDate": "2024-09-02T12:00:00",
                    "endDate": "2024-09-30T17:00:00",
                    "qty": 150,
                    "itemCd": "AD002",
                    "unit": "ea",
                    "unitPrice": 3000.00
                }
            ],
            "message": null
        }
    ],
    "pageInfo": {
        "page": 1,
        "size": 10,
        "totalElements": 2,
        "totalPage": 1
    }
}

```
<br>

- **중복 제거**: `selectDistinct()` 덕분에 동일한 **OrderHeaders**가 여러 번 출력되는 문제가 해결되었다.
- **성능 최적화**: `fetchJoin`을 통해 N+1 문제도 해결하고, 성능을 최적화했다. 이는 **OrderItems**가 여러 개 있는 경우에도 성능이 크게 떨어지지 않는다.

### 결론

이번 문제를 해결하면서 **쿼리의 구조**를 어떻게 개선해야 중복 문제와 성능 문제를 동시에 해결할 수 있을지에 대해 배울 수 있었다. <br> 특히, **1 :N관계**에서 발생하는 중복 문제는 종종 발생할 수 있기 때문에,<br> 이를 해결하기 위한 **DISTINCT**와 **fetchJoin**의 사용법을 익혀두는 것이 중요하다. <br> 이제 동일한 주문이 여러 번 중복되어 출력되지 않고, 각 주문에 포함된 **OrderItems**가 깔끔하게 하나의 결과로 반환되는 API를 완성할 수 있었다.<br><br>


<br>
<br><br><br><br><br>