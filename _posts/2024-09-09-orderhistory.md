---
layout : single
title : "[Project] SaleHisotry 구현, 스냅샷 시도 실패"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---
## 주문 내역 수정 문제와 스냅샷 방식 도입기

최근 프로젝트에서 **주문 이력 관리**에 큰 난관을 겪었다. 문제는 간단해 보였지만, 실제 구현하면서 예상치 못한 이슈들이 연달아 터지면서 여러 가지 해결책을 시도하게 됐다. 이번에는 **Sale History**와 관련된 이슈와 그 해결 과정을 블로그에 기록해보려고 한다.

### 문제 상황: 수정된 주문 내역이 덮어씌워지는 문제

처음에는 **OrderHeader**와 **OrderHistory**를 연관관계로 연결하여, 주문 상태나 아이템이 수정될 때 **OrderHistory**를 기록하려고 했다.<br> 그런데, 이렇게 구현했더니 주문이 수정되거나 상태가 변경될 때마다 **최신 데이터로 모든 내역이 덮어씌워지는** 문제가 발생했다.<br>  즉, 이전 상태의 주문 내역을 남기고 싶었지만, 모든 주문 내역이 최신 정보로 바뀌어버리는 상황이었다.<br> 

```java
{
    "data": [
        {
            "createdAt": "2024-09-09T23:48:46.22745",
            "employeeId": "EMP002",
            "orderId": 1,
            "orderStatus": "PURCHASE_REQUEST",
            "orderDate": "2024-09-09T23:48:46.22745",
            "requestDate": "2024-09-01T10:00:00",
            "buyerNm": "이름",
            "buyerCd": "B001",
            "orderItems": [
                {
                    "orderItemId": 1,
                    "startDate": "2024-09-01T10:00:00",
                    "endDate": "2024-09-15T18:00:00",
                    "quantity": 10,
                    "itemCD": "I001",
                    "unit": "ea",
                    "unitPrice": 100.50
                },
                {
                    "orderItemId": 2,
                    "startDate": "2024-09-02T12:00:00",
                    "endDate": "2024-09-16T17:00:00",
                    "quantity": 5,
                    "itemCD": "I002",
                    "unit": "ea",
                    "unitPrice": 200.00
                }
            ]
        },
        {
            "createdAt": "2024-09-09T23:48:46.22745",
            "employeeId": "EMP002",
            "orderId": 1,
            "orderStatus": "PURCHASE_REQUEST",
            "orderDate": "2024-09-09T23:48:46.22745",
            "requestDate": "2024-09-01T10:00:00",
            "buyerNm": "이름",
            "buyerCd": "B001",
            "orderItems": [
                {
                    "orderItemId": 1,
                    "startDate": "2024-09-01T10:00:00",
                    "endDate": "2024-09-15T18:00:00",
                    "quantity": 10,
                    "itemCD": "I001",
                    "unit": "ea",
                    "unitPrice": 100.50
                },
                {
                    "orderItemId": 2,
                    "startDate": "2024-09-02T12:00:00",
                    "endDate": "2024-09-16T17:00:00",
                    "quantity": 5,
                    "itemCD": "I002",
                    "unit": "ea",
                    "unitPrice": 200.00
                }
            ]
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

### 해결1.  스냅샷 방법 도입

이 문제를 해결하기 위해 검색하다가, **스냅샷(Snapshot) 방식**이라는 것을 알게 되었다. 

이 방식은 **주문 내역의 상태를 그대로 저장**하는 개념으로, **OrderItems** 리스트를 JSON 형식으로 직렬화해 하나의 스냅샷으로 저장하는 방법이다.

이를 위해 Jackson 라이브러리를 활용해 **OrderItems** 리스트를 JSON 문자열로 변환하여 데이터베이스에 저장하는 방식으로 처리했다.

1. 스냅샷 방법
    
    ```java
    public void setOrderItemsSnapshot(List<OrderItems> orderItems) {
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            objectMapper.registerModule(new JavaTimeModule());
            objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
            this.orderItemsSnapshot = objectMapper.writeValueAsString(orderItems);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Failed to serialize order items", e);
        }
    }
    ```
    
    이 방식을 사용하면 각 주문의 **아이템 내역이 그대로 스냅샷**으로 남기 때문에, 상태가 변경되더라도 과거의 내역을 잃지 않고 유지할 수 있다. 
    
    그러나 여기서도 예상치 못한 문제들이 발생했다.
    
2. **스냅샷 방식의 문제: 보기 불편한 데이터**
    <img src = "https://github.com/user-attachments/assets/c2fa2fa0-5432-440e-9c8e-2d269c1b21fe" width=800 />
    
    스냅샷 방식은 기본적으로 JSON으로 데이터를 직렬화하기 때문에, 데이터베이스에 저장된 내역이 텍스트 형태로 저장된다.
    
    이로 인해 DB에서 데이터를 직접 조회할 때는 **JSON 텍스트가 복잡하게 보여서 가독성이 떨어지는 문제**가 있었다. 
    
    또한 프론트에서도 이걸 받으면 구현하기 힘들 것이라 예상되었다.
    
    결국, **연관관계를 끊고 별도의 엔티티**로 관리하는 방법으로 방향을 전환했다.
    

### 해결 2. 연관관계 제거 후 SaleHistory 및 SaleItemHistory 분리

연관관계를 제거하고 **SaleHistory**와 **SaleItemHistory**를 각각 별도의 엔티티로 만들어 관리하기로 했다. 이 방법을 적용한 후, 문제없이 이력을 관리할 수 있게 되었고, 데이터베이스도 더 깔끔해졌다. 그러나 이렇게 분리한 후에는 또 다른 문제가 발생했다.

1. 무한 참조 문제: 순환 참조로 인한 무한 데이터 발생 <br/>

    <img src= "https://github.com/user-attachments/assets/fee32b8c-173e-460d-b8e8-496c9c72dd60" width=500>
    
    연관관계를 끊은 후, 주문 내역을 조회하는 API를 호출했더니 **SaleHistory**와 **SaleItemHistory**가 서로를 무한히 참조하면서 끝없는 데이터가 반환되는 문제가 발생했다. 이는 서로가 서로를 계속 참조하면서 무한 루프가 발생한 것이다.
    
2. 해결책: @JsonManagedReference와 @JsonBackReference 사용

    <img src= "https://github.com/user-attachments/assets/ce482d54-c3a9-40bd-afb4-de8d2956ec29" width=500>
    
    이 문제는 Jackson의 **@JsonManagedReference**와 **@JsonBackReference**를 사용해서 순환 참조를 끊어 해결할 수 있었다. **@JsonManagedReference**는 부모 객체를 직렬화하고, **@JsonBackReference**는 자식 객체에서 다시 부모를 참조하지 않도록 한다. 이렇게 설정한 후 무한 참조 문제는 깔끔하게 해결되었다.
    
    이 설정 덕분에 순환 참조 문제를 피할 수 있었고, **API 호출 시 무한 데이터 반환** 문제도 해결되었다.
    

### 결론

이번 이슈를 해결하면서 **주문 내역 관리**의 복잡함을 다시 한 번 깨달았다. 단순히 주문 데이터를 저장하고 이력을 관리하는 것처럼 보였지만, **데이터의 불변성**을 보장하면서도 효율적으로 저장하는 것이 얼마나 중요한지 배울 수 있었다. 스냅샷 방식부터 연관관계 제거, 순환 참조 문제 해결까지 다양한 시도를 거치며 얻은 교훈들을 바탕으로, 앞으로도 데이터 구조 설계에 대해 더 깊이 고민해야 할 것 같다.

특히 이번 경험에서 배운 점은 **데이터베이스의 구조적 설계**와 **연관관계 관리**가 중요하다는 것. 그리고 스냅샷 방식처럼 새로운 개념을 적용해보는 것도 큰 도움이 될 수 있다는 점이다. 앞으로는 더 효율적인 방식으로 주문 내역을 관리하면서, 시스템의 확장성과 유지보수성을 함께 고려해야겠다.