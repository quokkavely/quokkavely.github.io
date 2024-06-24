---
layout : single
title : "[DBMS] ERD 실습"
categories: Database
tag : [DBMS, 실습]
author_profile: true
---

# **ERD (Entity-Relationship Diagram)**

### **ERD의 주요 구성 요소**

1. **엔터티 (Entity)**
    1. 엔터티는 데이터베이스에서 관리하려는 실체나 개체
    2. 일반적으로 테이블과 일치하며, ERD에서는 직사각형으로 표현
2. **속성 (Attribute)**
    1. 속성은 엔티티의 특징이나 성질을 나타냄 
    2. 데이터베이스 테이블의 열(column)에 해당
    3. 속성은 ERD에서 원 또는 타원으로 표현되며, 해당 엔티티와 연결
3. **관계 (Relationship)**
    1. 서로 다른 엔티티 간의 연결을 나타냄
    2. 관계는 일대일, 일대다, 다대다 등의 다양한 형태로 존재
    3. ERD에서는 마름모로 표현되고, 관련된 엔티티와 직선으로 연결

### **ERD 작성 방법**

1. **엔티티 식별**: 데이터베이스에서 관리하고자 하는 개체나 실체를 파악하고, 엔티티를 정의.

2. **속성 결정**: 각 엔티티에 대한 속성을 식별하고, 엔티티와 속성을 연결

3. **관계 설정**: 서로 다른 엔티티 간의 관계를 파악하고, ERD에 관계를 표현

4. **정규화**: 데이터 중복을 최소화하고, 데이터 무결성을 보장하기 위해 정규화 과정을 수행

   <br>

## ERD 직접 작성해보기

### 키오스크

**[내가 작성한 것]**

| 판매상점store | 고객customer | 주문order | 메뉴menu | 결제payment |
| --- | --- | --- | --- | --- |
| 상점이름storename | 고객번호(PK)customernum-id | 주문번호(PK) | 메뉴이름menuname | 결제방법 |
| 상점번호(PK)store_id |  | 주문취소 | 메뉴가격 | 고객번호 |
|  | 이름customer name | 고객번호(FK) | 재고 | 주문번호 |
|  | 생년월일 | 주문수량 | 판매상점 | 할인 |
|  |  | 상점번호(FK) | 메뉴카테고리 | 적립 |
|  |  | 주문이력 |  |  |




**[Reference ]**

| Member | Order | Product | product-order<join table> |
| --- | --- | --- | --- |
| member_id(PK) | order_id (PK) | product_id (PK) | productorder_id(PK) |
| grade | member_id(FK) | name | product_id(FK) |
| phonenumber | product_id(FK) | price | order_id(FK) |
|  |  | size | count |
|  |  | kcal |  |
|  |  | type |  |

<br>

**[내가 생각한 관계와 Referecne의 차이]**

| 관계                 | Reference | **나**  |
| -------------------- | --------- | ------- |
| **영화 ↔ 상영관**    | **1:N**   | **1:1** |
| 영화 ↔ 영화관        | N:M       | N:M     |
| 영화관 ↔ 상영관      | 1:N       | 1:N     |
| Reservation ↔ Ticket | **1:N**   | **1:1** |
| 상영관 ↔ Ticket      | 1:N       | 1:N     |
| 고객 ↔ Resevation    | 1:N       | 1:N     |
| 고객 ↔ Review        | X         | N:M     |

<br>

    이렇게 생각했는데 Reservation과 ticket은
    
    하나의 티켓은 하나의 예약만을 가지고
    
    하나의 예약은 하나의 티켓이라 생각해서 1:1이라고 생각했는데
    
    하나의 예매에 여러장의 티켓을 출력할 일이 생길 수도 있어서
    
    일대다라고 한다...!
    
    또한 하나의 상영관은 하나의 영화만 튼다고 생각했는데
    
    하나의 상영관이 여러 영화를 트는 경우가 생길 수있으므로 1:N
    
    이게 여러 경우가 많고 어떤걸 기준으로 
    
    생각하면 달라질 수도 있어서 어려운 것 같다.

**[한번 더 짚어보기!✔️✨✨]**

관계를 생각하다보니 삼지창의 방향이 갑자기 헷갈렸는데

1:N의 경우에

N에 해당하는 경우가 삼지창이 향하고, 

1에 해당하는 기본키를 참조키로 가진다. 

그리고 외래키는 1개만 참조하고

기본키도 여러개가 올 수 있지만

하나로 두는 것을 권장!

<br>

**[참고] 이해가 안가면 아래처럼 표 만들어서 생각해보기!**

| name | member id | grade | phoneumber | p_id | p_name | p_price | p_size | p_type | p_kcal | orderid | member id | prodcut id | count |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 이정민 | 1 | 1 | 010-1000 | 1 | 아아 | 2000 | L | HOT |  |  |  |  |  |
| 김러키 | 2 | 1 |  | 2 | 아아 | 2500 | L | ICE |  |  |  |  |  |
|  | 3 | 1 |  | 3 | 아아 | 1500 | S | ICE |  |  |  |  |  |
|  | 4 | 1 |  | 4 | 아아 | 2500 | L | ICE |  |  |  |  |  |
|  | 5 | 1 |  | 5 | 바라 | 3500 | S | HOT |  |  |  |  |  |
|  | 6 |  |  | 6 | 바라 | 4000 | L | HOT |  |  |  |  |  |
|  | 7 |  |  | 7 | 바라 | 4000 | S | ICE |  |  |  |  |  |
|  | 8 |  |  | 8 | 바라 | 4500 | L | ICE |  |  |  |  |  |

<br>

<br>

### 조인테이블

✔️🔑다대다 관계일때 필수사용!

- 다대다 테이블의 경우 주문을 나눠서쓴경우,

- order1~3까지 같은주문인지를 알 수 없음 ⇒ 연계성이없어서 ⇒ 연결해줄 테이블이 필요 = 조인테이블을 가져야 함

| Productorder_id | product_id | order_id | order_id | count | member_id |
| --- | --- | --- | --- | --- | --- |
| 1 | 8 | 1 | 1 | 1 | 1 |
| 2 | 2 | 1 | 2 | 2 | 3 |
| 3 | 4 | 1 | 3 | 3 | 1 |
| 4 | 5 | 2 | 1 | 4 | 1 |
| 5 | 5 | 1 | 1 |  |  |
| 6 | 8 | 3 | 1 |  |  |
| 7 | 4 | 4 | 1 |  |  |

- ORDER_ID를 통해서 주문을 몇 개했는지 알 수 있고 제품이 몇개 주문이 들어왔는지는 Product_id로 알 수 있음
- 내가 여태껏 주문 했던 내역을 찾고 싶으면. member의 기본키인 member_id로 확인할 수 있음



<br>

### 도서관리시스템

처음에는 책과 대출시스템은 일대다 관계라고 생각함.

왜냐면 책은 하나만 빌릴 수 있는지?<br>

➡️ 하나의 대출은 여러 개의 책을 가져서라고 생각했는데 하나의 책만 빌릴 수 있는 것이 아니라서 (나는 여러 개의 책을 빌릴 수 있음) 그렇기때문에 다대다 관계이다.

다대다는 조인테이블이 필요한데

일대다 일 경우에는 그때도 해당 PK를 참조키로 가져오는건지? <br>
➡️다가 일의 PK를 FK로 가져옴

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e69e591e-e03c-4d26-9cbd-b9ce8103df63"/>

**[Reference]**



<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/db97d1e4-1e4a-46e7-9d84-c70e439d03b2">



Loan은 null값이 들어 올 수 있음 = 반납을 안하는 동안은 null

실제 조회는 Book, Loan, Book_Loan 여기서 빈번하게 발생

Book - Author     N:M <br>
Book-Publisher   N:1 <br>
Book-Category   N:M <br>
Book-Loan          N:M <br>
Member - Loan   1:N <br>

예약기능이라던지 확장을 많이 하게되면 고려할 것이 더 많아짐 ⇒ 성능이슈도 고려

바나프레소 어플의 스탬프는 1:1관계임

등급제로 회원할인적용은 별개

**[Reference와의 차이점]**

book의 카테고리를 따로 빼서 일대다로 적용했고

나는 사서시스템의 테이블을 따로 만들었는데 , 이건 너무 확장한 기능이라 복잡해지기 쉽다.

관계설정은 거의 동일함.



### 영화예매시스템

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b58d8f57-3072-44e6-a55c-e68f28246677"/>

Theater와 Review는 상관x

예매시스템이기때문에 Movie와 Member는 상관x - 관람객은 영화를 보는 것도 예약이라는 행위를 통해서만 함.

**Theater ↔ Movie N:M**

하나의 영화관에서는 하나의 영화 ? X

하나의 영화는 하나의 영화관에서? X

**ScreeningHall↔movie 1:1**

하나의 상영관에서 하나의 영화? o

하나의영화에서 하나의 상영관? o

갈릴 수 있지만 1:1

**Movie ↔ Resevation N:1**

하나의 영화는 하나의 예약만? X

하나의 예약은 하나의 영화만? O

**Reservation ↔ Ticket 1:1**

하나의 예약은 하나의 티켓? O 

하나의 티켓은 하나의 예약? O

**Member↔ Resevation 1:N**

하나의 멤버는 하나의 예약만? 여러개를 예약할 수 있지않나. ???

하나의 예약은 하나의 멤버만? O

**Movie ↔ Review N:1**

하나의 영화는 하나의 리뷰? X M

하나의 리뷰는 하나의 영화 ? O 1

---

[Reference] 예매관리시스템

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/1b8bfd2e-85c2-40e8-96e6-207972a3eba5"/>

Member/Movie/Ticket/Order/Cinema(영화관)/Theater(상영관)/

**Member - Order - Ticket**

 1——— N, 1———N

**Cinema - theater**

 1——— N

**Ticket - MovIe**

N————1

**Movie - Theater**

1———-N

**`외래키를 가지는 것은 N`,삼지창 방향이 N으로 향해야함.**

둘다  누군가의 조인테이블일때 서로게이트? 

영화와 시네마의 조인테이블 역할을 theater가 함.

cinema와 movie의 조인테이블 역할로 theater를 사용할 수 있음

seat를 넣게 되면 티켓에 seat가 다다르게 오기때문에  티켓은 다 다른 걸로 봐야해서 전부 틀어짐

**Reference와의 차이점**

| 관계 | Reference | 나 |
| --- | --- | --- |
| 영화 ↔ 상영관 | 1:N | 1:1 |
| 영화 ↔ 영화관 | N:M | N:M |
| 영화관 ↔ 상영관 | 1:N | 1:N |
| Reservation ↔ Ticket | 1:N | 1:1 |
| 상영관 ↔ Ticket | 1:N | 1:N |
| 고객 ↔ Resevation | 1:N | 1:N |
| 고객 ↔ Review | X | N:M |


Reference 영화와 영화관의 조인테이블로 상영관을 넣었고

나는 상영관과 영화를 1대1이라 생각함 왜냐면 상영관에서 영화는 하나만 트니까,

근데 또 시간 기준으로 생각하면 그런데 다른 영화를 상영할 수 도 잇음….!

그리고 나는 예매시스템과 티켓의 관계는 1:1로 생각했는데

하나의 예매내역은 하나의 티켓을가지고 하나의 티켓은 하나의 예매내역을 가질 수있는데

또 경우에 따라서는 여러장의 티켓을 가질 수도 있어서 일대 다가 올 수도 있다….!

나는 영화와 예매를 엮었는데 Reference에서는 엮지 않음.

<br>



### 인스타그램

- 인스타그램 구현조건

    - 좋아요, 댓글, 팔로워 팔로잉 구현 W

    - 대댓 스토리은 없음 X

    - 해쉬태그 - 해쉬태그달려있는 글 조회 O

    - 게시글

    - 사진

    - 멤버 , 회원가입

    미리 큰 주요기능을 잡고 부가기능 구현.

    좋아요는 한번누르면 좋아요, 두번누르면 취소됨. - 구현하기

    1. 글작성 → 파생
    
    2. 부가기능
        1. 사진추가
        2. 좋아요
        3. 댓글
        4. 해시태그

    3. 팔로, 팔로잉

        <br>

    **[내가 작성한 인스타그램 ERD]**

    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/81ed75dd-9e81-4785-b6e2-c801a5c66968"/>

    Member를 기준으로 작성

    **[Reference ERD]**

    **좋아요**는 있고 없고로(true or false) 데이터를 만들 수 있음.

    게시글id ➡️ 하나의 사진이 하나의 게시글에 올라감 ⇒ 사진이 여러개의 게시글에 올라간다는 것은 내가 올렸는데 다른 게시글에서도 사진이 올라가는것.

    게시글 사진은 따로 저장해두고 주소값만 저장하고 찾아오는 방식임

    포토를 올릴때는 누가올리는지 멤버를 알아야하는데

    포스트를FK로 받아서 포스트가 받는 MEMBER  ID로 올리는 멤버를 참조할 수 있음.

    팔로우 팔로워는 즐겨찾기라고 생각 , 팔로우 팔로워는 단방향임.

    멤버와 멤버의 관계로 봐야함 . 심지어 다대다
    
    src(source)  = 원본대상
    
    dst(destination) =복사대상