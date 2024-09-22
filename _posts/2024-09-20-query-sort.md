---
layout : single
title : "[Project] QueryDSL 문법 정리 및 Sorting 문제 해결"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## QueryDSL 문법 정리 및 Sorting 문제 해결

전체 백엔드 코드 수정하면서 검색기능을 사용해야 하는 곳에는 모두 QueryDSL로 변경하기로 했다. <br> 수정하면서도 자주 사용하는 eq 정도랑 fetch만 알고 있었는데 containsIgnoreCase() 알게되면서 <br> 다음에 사용할 때 조금 더 유용하거나 기본적인 문법들은 알고 넘어가야겠다 생각해서 정리해보았다.

### 1. QueryDSL 기본 문법

1. 조건 메서드
    - **`eq()`**: `=` 연산 (Equal)
    - **`ne()`**: `!=` 연산 (Not Equal)
    - **`not()`**: 해당 조건을 부정
    - **`isNotNull()`**: `IS NOT NULL` 연산
    - **`in()`**: `IN` 연산
    - **`notIn()`**: `NOT IN` 연산
    - **`between(a,b)`**: `BETWEEN a AND b` 연산
    - **`goe()`**: `>=` 연산 (Greater or Equal)
    - **`gt()`**: `>` 연산 (Greater Than)
    - **`loe()`**: `<=` 연산 (Less or Equal)
    - **`lt()`**: `<` 연산 (Less Than)
    - **`like()`**: SQL의 `LIKE` 연산. `%`를 수동으로 넣어줘야 한다.
    - **`contains()`**: 양쪽에 `%`가 자동으로 추가된 **LIKE** 연산.
    - **`startsWith()`**: 뒤에 `%`가 추가된 **LIKE** 연산.
    - **`containsIgnoreCase()`**: 대소문자 구분 없이 포함된 단어 검색.
    - **`likeIgnoreCase()`**: 대소문자 구분 없이 **LIKE** 연산.
2. 데이터 조회 메서드
    - **`fetch()`**: 여러 건의 데이터를 조회할 때 주로 사용하고, 결과가 없으면 빈 리스트(`[]`)를 반환
    - **`fetchOne()`**: 정확히 한 건의 결과를 기대할 때 사용. 결과가 없거나 두 개 이상일 경우 예외가 발생.
    - **`fetchFirst()`**: 여러 결과 중에서 **가장 첫 번째 결과**만 가져온. 결과가 없으면 `null`을 반환. 내부적으로 `limit 1`을 추가한 쿼리를 실행한다.
    - **`fetchResults()`**: **페이징 처리**를 포함한 결과를 조회할 때 사용. **총 개수**를 구하는 **count 쿼리**가 추가로 실행된다. **QueryResults** 객체로 결과를 리턴하며, **페이징 정보**와 **실제 데이터**를 함께 가져올 수 있다.
        - **`getTotal()`**: 총 개수를 가져온다.
        - **`getLimit()`**: 쿼리에서 설정된 limit 값을 가져온다.
        - **`getOffset()`**: 쿼리에서 설정된 offset 값을 가져온다. (0부터 시작)
        - **`getResults()`**: fetch()처럼 실제 데이터를 리스트로 리턴한다.
    - **`fetchCount()`**:쿼리 조건에 맞는 **전체 레코드 수를 반환**할 때 사용, 데이터 자체가 아니라, **몇 개의 결과**가 있는지만 확인할 때 유용

### 2. Pagination 및 Sorting 구현

프로젝트에서 기존에 잘 동작하던 정렬 기능이 **QueryDSL**로 변경한 후에 제대로 동작하지 않는 문제가 발생했다.

정렬 제대로 설정되지 않은 문제라서 이때는 OrderSpecifier를 사용해 정렬 기준을 명확하게 정의해야 한다..

### 해결 방법: `OrderSpecifier`를 사용해 정렬 적용

정렬 조건을 동적으로 처리하고자 한다면, **`getSortOrder()`** 메서드를 사용해 **OrderSpecifier** 리스트를 생성한 후, 이를 orderBy() 에 적용해야 한다.

1. 정렬 메서드
    
    ```java
    private List<OrderSpecifier<?>> getSortOrder(Pageable pageable, QItemManufacture itemManufacture) {
        // OrderSpecifier<?> 리스트를 초기화한다.
        List<OrderSpecifier<?>> orders = new ArrayList<>();
    
        // Pageable 객체에서 Sort 정보를 가져온다.
        for (Sort.Order order : pageable.getSort()) {
            // PathBuilder는 필드에 대한 경로 정보를 가져오는 역할을 한다.
            // itemManufacture 엔티티에서 타입과 메타데이터를 가져와 PathBuilder를 생성.
            PathBuilder pathBuilder = new PathBuilder(itemManufacture.getType(), itemManufacture.getMetadata());
    
            // Sort.Order 객체가 오름차순이면 ASC, 내림차순이면 DESC로 OrderSpecifier를 추가한다.
            orders.add(new OrderSpecifier(order.isAscending() ? Order.ASC : Order.DESC,
                                          pathBuilder.get(order.getProperty())));
        }
    
        // OrderSpecifier 리스트를 반환.
        return orders;
    }
    ```
    
    - **`Pageable`**: 스프링에서 페이징과 정렬을 관리하는 객체. 페이지 번호, 페이지 크기, 정렬 기준 등을 포함한다.
    - **`PathBuilder`**: 동적으로 **경로**를 생성해주는 클래스. 여기서는 **필드명**을 동적으로 접근하기 위해 사용된다.
    - **`OrderSpecifier`**: 정렬을 위한 객체로, 오름차순과 내림차순을 설정할 수 있다.
2. 전체 코드 (Sorting과 Pagination 적용)
    
    ```java
    // Sort 처리: getSortOrder 메서드를 사용해 OrderSpecifier 목록을 생성.
    List<OrderSpecifier<?>> orderSpecifiers = getSortOrder(pageable, itemManufacture);
    
    // QueryDSL로 데이터 조회
    List<ItemManufacture> results = queryFactory
            .selectFrom(itemManufacture)
            .where(builder)  // 조건절 추가
            .orderBy(orderSpecifiers.toArray(new OrderSpecifier[0]))  // OrderSpecifier 배열로 정렬 조건 추가
            .offset(pageable.getOffset())  // 페이징 offset 설정 , 페이지의 시작점
            .limit(pageable.getPageSize())  // 페이징 limit 설정, 데이터의 개수
            .fetch();  // 결과 리스트로 가져옴
    ```
    
    - **`getSortOrder(pageable, itemManufacture)`**:
        - 위에서 정의한 메서드를 호출해, **정렬 조건**을 가져온다.
    - **`orderBy(orderSpecifiers.toArray(new OrderSpecifier[0]))`**:
        - 정렬 조건을 **OrderSpecifier 배열**로 변환하여 **orderBy**에 전달한다.

.