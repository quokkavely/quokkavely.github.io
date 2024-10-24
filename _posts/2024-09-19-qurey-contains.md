---
layout : single
title : "[Project] QueryDSL로 검색 기능 개선 (contains, like)"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## QueryDSL로 검색 기능 개선: 대소문자 구분 없이 검색하기

프로젝트를 진행하다 보면, 검색 기능이 단순히 **완전 일치**만을 요구하는 것이 아니라, **일부 단어**만 입력해도 결과가 나오도록 하는 요구가 생길 수 있다. 특히 필드가 많고, 여러 엔티티와 관계된 데이터를 검색해야 하는 경우라면, 검색 기능의 유연성이 매우 중요하다. 이번 글에서는 QueryDSL로 **대소문자 구분 없이 검색**하고, **일부 단어**만으로도 결과를 반환하는 방법을 공유해보려고 한다.

### 문제 상황: 완전 일치만으로는 불편한 검색

기존에는 `eq()` 메서드를 사용해서 **DB에 있는 필드 값**과 **사용자가 입력한 값**이 **완전히 일치**해야만 결과가 반환됐다. 예를 들어, 바이어 이름이 "Footwear Hub"일 때, "Footwear Hub"로 정확히 검색해야 결과가 나왔고, "footwear", "hub" 같은 **부분 검색**은 불가능했다.

### 해결 방법: `containsIgnoreCase()` 사용

**QueryDSL**에서는 **대소문자 구분 없이** **포함된 단어만**으로도 검색할 수 있는 메서드가 있다. 바로 `containsIgnoreCase()`를 사용하는 방법이다. 이 메서드는 입력된 검색어가 필드 값에 **포함**되어 있으면 결과를 반환해준다.

1. 기존 코드 (`eq()` 사용)
    
    ```java
    public BooleanBuilder getBuyerPredicate(String buyerName) {
        QBuyer qBuyer = QBuyer.buyer;
        BooleanBuilder builder = new BooleanBuilder();
    
        if (buyerName != null && !buyerName.isEmpty()) {
            builder.and(qBuyer.buyerName.eq(buyerName));
        }
    ```
    
    이 코드는 **완전 일치**하는 경우에만 결과를 반환하기 때문에, 사용자가 정확히 "Footwear Hub"로 검색해야만 결과를 얻을 수 있다. 대소문자도 구분되기 때문에 "footwear hub"로 검색하면 결과가 나오지 않는다.
    
2. 개선된 코드 (`containsIgnoreCase()` 사용)
    
    ```java
    public BooleanBuilder getBuyerPredicate(String buyerName) {
        QBuyer qBuyer = QBuyer.buyer;
        BooleanBuilder builder = new BooleanBuilder();
    
        if (buyerName != null && !buyerName.isEmpty()) {
            builder.and(qBuyer.buyerName.containsIgnoreCase(buyerName));
        }
    ```
    
    이렇게 하면 대소문자 구분 없이, 입력된 검색어가 **일부만 포함**되어도 결과를 반환한다. 예를 들어, "footwear" 또는 "hub"만 입력해도 "Footwear Hub"라는 이름을 가진 바이어를 검색할 수 있다.
    
    <img src = "https://github.com/user-attachments/assets/32e8478e-e17b-4182-b67b-aec8c5cf9375" width =500/>
    
3. `containsIgnoreCase()`의 한계
    
    `containsIgnoreCase()`를 사용하면 어느 정도 검색의 유연성을 제공할 수 있지만, 몇 가지 **한계**가 있다.
    
    - DB에 띄어쓰기 되어있는 단어가 저장된 경우
        - 예를 들어, "Footwear Hub"라는 이름을 가진 바이어를 검색할 때, "wearhub"로 검색하면 결과가 나오지 않는다.
    - **단어 사이에 공백이 있는 경우**:
        - 만약 사용자가 "f o o t w e a r" 처럼 **공백을 포함한 단어**로 검색하려고 한다면, `containsIgnoreCase()`로는 원하는 결과를 얻을 수 없다.

### 더 강력한 검색을 위한 `like` 사용

만약 **더 복잡한 검색**이 필요하다면, 예를 들어 "wearhub"처럼 **단어의 일부분만 포함하는 경우**나, **단어 사이에 공백**이 있는 경우에도 검색을 지원하려면, `like` 문법을 사용해야 한다. `like`는 SQL에서 많이 사용되는 패턴 매칭 기능을 제공하며, 사용자가 입력한 값의 **일부** 또는 **변형된 형태**를 검색할 때 유용하다.

1. `like`를 사용한 검색
    
    ```java
    public BooleanBuilder getBuyerPredicate(String buyerName) {
        QBuyer qBuyer = QBuyer.buyer;
        BooleanBuilder builder = new BooleanBuilder();
    
        if (buyerName != null && !buyerName.isEmpty()) {
            // %는 와일드카드 문자로, 앞뒤로 어떤 문자가 와도 상관없다는 의미
            builder.and(qBuyer.buyerName.likeIgnoreCase("%" + buyerName + "%"));
        }
    ```
    
    이 코드는 **대소문자 구분 없이** 입력된 검색어의 일부만 일치해도 결과를 반환해준다. 예를 들어, "wearhub"로 검색하면 "Footwear Hub"도 검색될 수 있다.
    

### 데이터베이스 변경 없이 더 유연한 검색 제공

위에서 설명한 방법으로도 만족하지 못하는 경우, **데이터베이스 구조**를 변경해야 할 수도 있다. 예를 들어, 검색어의 각 단어를 **인덱싱**하거나, **n-gram** 같은 텍스트 분석 기법을 도입할 수 있다. 하지만 지금 프로젝트에서 검색이 중요한 부분을 차지하는 것이 아니라서 `containsIgnoreCase()`와 **`like`** 정도만으로도 충분할 것이라 생각했다.

### 결론

- `containsIgnoreCase()`는 **대소문자 구분 없이**, **입력된 단어가 포함**된 결과를 반환할 수 있는 좋은 방법이다.
- 더 복잡한 검색 기능이 필요하다면, `like`를 사용해 **일부 단어**나 **단어 순서**에 유연한 검색을 구현할 수 있다.
- 필드가 많고 검색 조건이 다양해질수록, 반복적인 작업을 줄이기 위해 공통 로직을 추출해 **재사용 가능한 메서드**로 만들어 두는 것도 좋은 전략이다.

사실 팀원 분이 일부 단어만 포함되도 검색할 수 있게 해달라고 요청했는데 그건 엘라스틱서치에서만 가능하지 않을까요?… 일단 시도해보겠습니다 라고하고 찾아봤는데 너무 쉽게 메서드 하나만 바꿔도 되어서 ‘와 QueryDSL 너무 좋은데…’라고 생각했다.

이번 프로젝트에서 가장 큰 수확은 QueryDSL 을 배우게 된 것 아닐까 ! 굿굿


<br>
<br><br><br><br><br>