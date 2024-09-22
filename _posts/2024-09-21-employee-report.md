---
layout : single
title : "[Project] ERP 시스템에서 사원별 실적 관리 구현 : 잘못된 설계로 인한 어려움"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## ERP 시스템에서 사원별 실적 관리 구현 

이번 프로젝트에서는 사원별로 **실적 관리**를 하면서, **판매 건수**, **판매 금액**, 그리고 **마진률**까지 계산하는 기능을 구현해야 했다. 

특히, **마진률** 계산이 어려웠는데, 그 이유는 단순히 **판매 금액**만 가져오는 것이 아니라, **제조 단가**까지 고려해야 했기 때문이다😥😥

### 1. 실적 관리

**판매 건수**와 **판매 금액**을 계산하는 것은 비교적 수월했다. <Br> `OrderHeader`와 `Member` 테이블 간의 관계를 통해 필요한 데이터를 쉽게 가져올 수 있었다.<br> 하지만 **마진률**은 복잡한 문제였다.

마진률은 단순히 **판매 금액**에서 **제조 단가**를 뺀 값을 기반으로 계산해야 하는데, <Br> 문제는 **OrderItem**에서 **제조 단가**까지 접근하는 경로가 너무 복잡했다.

- `OrderItem` → `OrderHeader` → `Buyer` → `BuyerItem` → `Item` → **`ItemManufacture`**로 이어지는 관계를 따라가야 했는데, 이 관계 자체가 너무 비효율적이었다.

### 2. 1차 시도 실패

처음에는 다음과 같이 코드를 작성했다. `OrderItem`에서 `BuyerItem`을 찾아 제조 단가를 가져오고, 이를 활용해 마진률을 계산하려고 했다.

```java
for (OrderItems orderItem : orderItems) {
    totalOrderPrice = totalOrderPrice.add(orderItem.getUnitPrice());

    // 아이템 코드에 맞는 제조 단가 찾기
    List<BuyerItem> buyerItems = orderItem.getOrderHeaders()
            .getBuyer()
            .getBuyerItems()
            .stream()
            .filter(buyerItem -> buyerItem.getItem() != null && buyerItem.getItem().getItemCd().equals(orderItem.getItemCd()))
            .collect(Collectors.toList());

    // 첫 번째 일치하는 BuyerItem에서 제조 단가 가져오기
    if (!buyerItems.isEmpty()) {
        BigDecimal manufacturePrice = buyerItems.get(0).getItem().getUnitPrice(); 
        // 제조 단가
        totalManufacturePrice = totalManufacturePrice.add(manufacturePrice);
    }
}
```

이 방식에서는 **제조 단가**를 제대로 가져오는 것처럼 보였지만, 결과적으로 **마진률이 0**으로만 나왔다.<br> 여러 번 디버깅을 해봤지만, 필터링 로직이 예상대로 동작하지 않는 것 같았다.

### 3. 설계 문제 발견

문제는 ERP 시스템 설계 자체에 있었다.<br> **ItemManufacture**(제조 단가)는 제조사의 납품 내역으로 설정되어 있었는데,<Br> 이것이 실제로는 **제조 단가관리와** **납품 내역**이 혼재된 상태였다. <br> 원래 **ItemManufacture**는 **제조 단가**만을 관리하고, **납품 내역**은 별도로 **ItemManufactureHistory**로 관리하는 것이 맞다는 결론을 내렸다.

하지만, 이미 시간이 많이 촉박한 상황이라서 **설계를 수정**할 시간이 없었고 <Br> 내가 변경하면 프론트도 모두 변경되어야 했다..<br> 안그래도 지금 프론트가 일이 많은 상황이고 이제 validation 후 정리해야하는 상황이라서 <Br> 결국 현재의 구조를 유지하면서 문제를 해결하기로 했다.

### 4. 해결 방법: 판매가와 제조가격을 따로 계산

결국 해결책은 **판매 리스트**와 **제조 단가 리스트**를 모두 가져온 후,<Br> 이를 별도의 for 문으로 비교해 계산하는 방식이었다.<br> 각 `OrderItem`과 `ItemManufacture`를 비교하여, **수량**과 **단가**를 계산해 총 판매 가격과 총 제조 가격을 구했다.

```java
// 영업사원이 판매한 제품 리스트
List<OrderItems> orderItems = orderItemsRepository.findByOrderHeadersRequestDateBetween(employeeId, start, end);

// 판매된 제품의 제조단가 리스트
List<ItemManufacture> mfItems = mfItemRepository.findManufacturedItemsForOrderItems(employeeId, start, end);

// 총 판매 가격 계산
for (OrderItems orderItem : orderItems) {
    totalOrderPrice = totalOrderPrice.add(orderItem.getUnitPrice().multiply(BigDecimal.valueOf(orderItem.getQty())));
}

// 총 제조 가격 계산
for (ItemManufacture mfItem : mfItems) {
    Optional<OrderItems> correspondingOrderItem = orderItems.stream()
        .filter(orderItem -> orderItem.getItemCd().equals(mfItem.getItem().getItemCd())) // ItemCd가 일치하는지 확인
        .findFirst();

    if (correspondingOrderItem.isPresent()) {
        OrderItems orderItem = correspondingOrderItem.get();
        // 해당 판매 수량 가져오기
        totalManufacturePrice = totalManufacturePrice.add(mfItem.getUnitPrice().multiply(BigDecimal.valueOf(orderItem.getQty())));
    }
}

```

이 방식을 통해 마진률을 올바르게 계산할 수 있었다...

하지만 **`findManufacturedItemsForOrderItems`** 메서드에서 발생하는 쿼리가 상당히 복잡해졌다.<br> 설계가 제대로 되어 있었다면, 더 간단하게 구현할 수 있었을 것이다.

```java

    @Override
    public List<ItemManufacture> findManufacturedItemsForOrderItems(String employeeId, LocalDateTime start, LocalDateTime end) {

        QItemManufacture itemManufacture = QItemManufacture.itemManufacture;
        QOrderItems orderItems = QOrderItems.orderItems;
        QOrderHeaders orderHeaders = QOrderHeaders.orderHeaders;
        QMember member = QMember.member;
        QItem item = QItem.item;

        // 판매된 제품의 itemCd 목록을 가져오기
        List<String> orderItemCds = queryFactory.select(orderItems.itemCd)
                .from(orderItems)
                .join(orderItems.orderHeaders, orderHeaders) // orderHeaders와 조인
                .join(orderHeaders.member, member) // member와 조인
                .where(
                        member.employeeId.eq(employeeId)
                                .and(orderHeaders.requestDate.between(start, end))
                )
                .fetch();

        // 제조된 제품의 itemCd와 조인하여 일치하는 항목 찾기
        return queryFactory.selectFrom(itemManufacture)
                .join(itemManufacture.item, item)  // ItemManufacture와 Item을 조인
                .where(
                        item.itemCd.in(orderItemCds)  // Item의 itemCd와 OrderItems의 itemCd 비교
                                .and(itemManufacture.createdAt.before(
                                        JPAExpressions.select(orderHeaders.requestDate.max()) // requestDate의 최대값을 선택
                                                .from(orderItems)
                                                .join(orderItems.orderHeaders, orderHeaders)
                                                .where(orderItems.itemCd.eq(item.itemCd))
                                ))
                )
                .fetch();
    }

```

---

이 문제를 해결하고 나니, 각 사원의 **판매 건수**, **판매 금액**, 그리고 **마진률**을 정확하게 계산할 수 있게 되었다.<Br> 하지만 이번 경험을 통해 느낀 점은, **처음부터 올바른 설계가 얼마나 중요한지**였다.

특히 **ERP 시스템**은 처음 접하는 경우가 많았고, <Br> 서로 다른 팀원들이 각자 생각하는 방식이 다르다 보니<br> 계속 정리를 해도 또 다시 얘기해보면 어느 순간 다른 ERP를 보고 있었다. <Br> 결국 설계 단계에서 충분한 논의가 이루어지지 않았다. <br> 그 결과, 시간이 지나면서 점점 더 복잡한 문제가 발생했다.

ERP와 같은 복잡한 시스템을 구축할 때는, **설계가 모든 것의 기초**다. <Br> 처음부터 올바른 관계를 설정하고, 데이터 흐름을 명확하게 이해하고 있으면 나중에 발생하는 문제들을 훨씬 줄일 수 있다.

이번 경험을 통해 얻은 가장 큰 교훈은, **설계에 충분한 시간을 투자하고, 팀원들과 충분한 논의를 거쳐야 한다는 것**이다. <br> 설계를 잘못하면 나중에 모든 작업이 꼬이기 마련이다. <Br> 앞으로는 이런 부분에 더 신경을 쓰고, 초기 단계에서부터 명확한 구조를 잡을 수 있도록 노력해야겠다.
