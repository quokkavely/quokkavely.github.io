---
layout : single
title : "[JPA] Persistence, 매핑"
categories: Spring
tag : [Spring, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# JPA(Java Persistence API)

## JPA란?

### **JPA**

- ORM(Object-Relational Mapping) 기술의 표준 사양(또는 명세, Specification)
- 표준사양이란 인터페이스로 사양이 정의되었기 때문에
- JPA구현한 구현체는 따로 있다는 것을 의미
- 구현체로는  Hibernate ORM, EclipseLink, DataNucleus 등이 있는데  Hiberante ORM을 가장 기본으로 사용
- JPA는 Java Persistence API의 약자이지만 현재는 Jakarta Persistence라고도 불림

### Data Access Layer에서 JPA의 위치

- Layerd Architecture에서 데이터 액세스 계층의 상단에 위치
- 데이터 저장, 조회 등의 작업은 JPA를 거쳐 JPA의 구현체인 Hibernate ORM을 통해서 이루어짐
- Hibernate ORM은 내부적으로 JDBC API를 이용해서 데이터베이스에 접근
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/02b8be12-6e26-4574-9d75-0c287c234e87" width=300/>

## Persistence

- JPA에서 P는 Persistence를 뜻함.
- ORM은 객체(Object)와 데이터베이스 테이블의 매핑을 통해 엔티티 클래스 객체 안에 포함된 정보를 테이블에 저장하는 기술
- JPA에서는 테이블과 매핑되는 엔티티 객체정보를 영속성 컨텍스트에 보관해서 애플리케이션 내에 오래 지속되도록 함.
- 영속성 컨텍스트에는 1차 캐시와 쓰기지연 SQL 저장소의 영역이 있음.
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/ced9222a-5428-41ec-b16f-4897b25fdb66" width=250/>
    

        

### 영속성 컨텍스트와 EntityManger

1. 엔티티 클래스 생성
    
    ```java
    import lombok.Getter;
    
    import javax.persistence.*;
    
    @Getter
    @Setter
    @NoArgsConstructor
    @Entity //(1) 
    public class Member {
        @Id  // (2)
        @GeneratedValue  // (3)
        private Long memberId;
    
        private String email;
    
        public Member(String email) {
            this.email = email;
        }
    }
    
    ```
    
    (1) JPA 엔티티로 사용→ `members`라는 테이블에 매핑됨
    
    (2) 엔티티의 기본 키
    
    (3)`@GeneratedValue` 어노테이션은 기본 키 값이 자동으로 생성됨
    
2. 영속성 컨텍스트 관련 JPA API
    
    ```java
    @Configuration
    public class JpaBasicConfig {
        private EntityManager em; 
        private EntityTransaction tx;
    
        @Bean
        public CommandLineRunner testJpaBasicRunner(EntityManagerFactory emFactory) {
            this.em = emFactory.createEntityManager();
    
            
            this.tx = em.getTransaction();
            
            return args -> {
            Member member = new Member("hgd@gmail.com");
            em.persist(member); 
            //영속성 컨텍스트에 member 객체의 정보들이 저장
    				tx.commit();
    				
            Member resultMember = em.find(Member.class, 1L);
            System.out.println("Id: " + resultMember.getMemberId() + ", email: " +
                    resultMember.getEmail());
            };
        }
    ```
    
- **EntityManger**
    - JPA의 영속성 컨텍스트는 EntityManager 클래스에 의해서 관리된다.
    - EntityManager 클래스의 객체는 (1)과 같이 **EntityManagerFactory** 객체를 Spring으로부터 DI 받음
- EntityManagerFactory의 **createEntityManager() 메서드**를 이용해서 EntityManager 클래스의 객체를 얻을 수 있다.
    - createEntityManager는 @Entity 붙은 클래스를 데이터베이스 테이블로 자동으로 매핑되고, 애플리케이션 시작 시점에 테이블이 생성시켜줌.
    - [hibernate.hbm2ddl.auto](http://hibernate.hbm2ddl.auto) 속성에 따름. → 예제는 create가 적용되어 있음.
        - `create`: 애플리케이션이 실행될 때 기존 테이블을 삭제하고 새로 생성
        - `create-drop`: `create`와 동일하지만, 애플리케이션이 종료될 때 테이블을 삭제함
        - `update`: 기존 테이블을 유지하며 필요한 경우 테이블 구조를 업데이트 함
        - `validate`: 테이블이 존재하는지 확인 후 매핑 정보가 일치하는지 검증, 변경은 수행하지 않음
        - `none`: 아무런 동작도 하지 않음
- **Transaction을 시작**하기 위해서 **tx.begin() 메서드**를 먼저 호출
- **find() : 조회 기능  , commit 되면 Select 쿼리 날림**
    - 첫 번째 파라미터 : 조회할 엔티티 클래스의 타입
    - 두 번째 파라미터 : 조회할 엔티티 클래스의 식별자 값
- **em.persist()** : 엔티티 객체를 영속성 컨텍스트에 저장
- **em.flush()** : 영속성 컨텍스트의 변경 사항을 테이블에 반영 → tx.commit()이 호출되면 내부적으로 em.flush()가 호출됨.
- **tx.commit()** :  영속성 컨텍스트에 저장되어 있는 member 객체를 데이터베이스의 테이블에 저장

### 1차 캐시와 쓰기지연 SQL 저장소

1. **Entity Create, Insert, Select** 
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/ef9e7ce9-f701-492d-ae3e-7d5e49df7b2c" />
    
    - 애플리케이션이 시작되면서 [hibernate.hbm2ddl.auto](http://hibernate.hbm2ddl.auto) : create 속성에 따라 기존 테이블을 삭제하고 새로 생성함 (@Entity 가 붙은 클래스를 테이블로 만들어준다)
    - 분홍박스의 em.persist(member);  → 여기서 member가 1차 캐시에 저장되고 쓰기지연저장소에 insert쿼리가 저장되고(영속성컨텍스트에 저장)
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c4c9d5a4-2aef-429c-b9e4-8ad361e24333" width=250/>
        
    - 아랫줄에 tx.commit();이 실행되면 쿼리문이 실행되고 DB에 저장됨.
        
        쓰기지연저장소에는 쿼리문이 실행되어 사라지고 1차캐시에 있던 member는 남아있음
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/89aa373c-054a-485a-b287-6140905350f8" width=250/>
        
    - Member result =em.find(Member.class, 1L); 에서 1차 캐시에 member가 존재하기 때문에 select 쿼리를 날리지 않아도 조회 가능 → 콘솔에 출력도 잘 됨.
    - 그러나 member_Id가 2인 객체는 1차 캐시에 없으므로 select 쿼리를 날려서 DB를 조회 하고 없기 때문에 null 이 맞음.
    
2. **엔티티 업데이트**
    
    ```java
    private void example(){
       tx.begin();
       em.persist(new Member("jerry5@gmail.com")); 
        // 1차 캐시에 보관 - snapshot 찍어서 보관 
       tx.commit(); //commit 하는 순간 객체정보가 달라졌는지 비교 함
        // 주소값 공유해서 캐시와 DB가 맞지 않으면 업데이트가 추가 되고 commit시 쿼리가 실행됨.
            
        //다시 transaction 생성
    	 tx.begin();
       Member member1 =em.find(Member.class,1L); 
       //위에서 생성된 멤버를 가져옴. -> 1차 캐시에서 조회
       member1.setEmail("jerry5@yahoo.co.kr"); 
       //불러온 객체(1차캐시에 있는)를 바꿈.
      // 이미 이전에 찍어놓고 변경사항이 있으면 쓰기지연저장소에 쿼리등록
       //update 실행됨.(JPA가 알아서 변경사항을 추적하여 update 쿼리 날려줌)
                    
        }
    ```
    
3. 엔티티 삭제
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d230cd5a-7355-49f3-b694-82bdcfab8bf7" />
    
    - select 쿼리는 1차 캐시에  이미 존재하기 때문에 나가지 않음.
    - remove 하면 1차 캐시를 지우지만 사라졌다는 상태도 보관함.
    - tx.commit()을 실행하면 영속성 컨텍스트의 1차 캐시에 있는 엔티티를 제거하고, 쓰기 지연 SQL 저장소에 등록된 DELETE 쿼리가 실행
    - tx를 커밋할때 플러시를 강제로 발생시킴. ⇒ 커넥션 풀 담당 히카리 풀

## 엔티티매핑

### 엔티티와 테이블

1. **@Entity**
    - 엔티티 클래스와 테이블을 매핑
    - attribute
        - name: 엔티티의 이름 설정 가능, 참고로 이름이 중복되는 클래스가 있으면 절대 안됨!
2. **@Table**
    - 주로 테이블 이름이 클래스 이름과 달라야 할 경우에 추가
    - name attribute로 테이블 이름 설정 가능.
    - `@Table` 애너테이션은 옵션이며, 추가하지 않을 경우 클래스 이름을 테이블 이름으로 사용
3. 주의사항
    - **`@Entity` 애너테이션과 `@Id` 애너테이션은 필수**
    - 파라미터가 없는 **기본 생성자는 필수로 추가**
        
        → 생성자가 없을때는 기본 생성자가 자동으로 생성되지만 그렇지 않은 경우에는 기본 생성자가 필수로 있어야 하므로 습관적으로 기본 생성자 추가하기.
        
    - 테이블 이름을 따로 바꿀 필요가 없는 경우에는 클래스 이름으로 사용하는 것을 권장.

### 기본키 매핑

@ID 애너테이션을 추가한 필드가 기본 키 열이 되는데 JPA에서는 기본 키를 어떤 방식으로 생성해줄지 다양한 전략을 지원

1. **IDENTITY**
    - **@GeneratedValue(strategy = GenerationType.IDENTITY**
    - 데이터베이스에서 기본 키 대신 생성
    - Ex) MySQL의 AUTO_INCREMENT 기능
2. **SEQUENCE**
    - **@GeneratedValue(strategy = GenerationType.SEQUENCE)**
    - 데이터베이스에서 제공하는 시퀀스를 사용해서 기본 키를 생성하는 전략
3. **TABLE**
    - 별도의 키 생성 테이블을 사용하는 전략
4. **AUTO**
    - **@GeneratedValue(strategy = `GenerationType.AUTO`**
    - JPA가 데이터베이스의 Dialect에 따라서 적절한 전략을 자동으로 선택

### 필드(멤버변수)와 Column 간의 매핑

**Annotation**

| annotation | Description |
| --- | --- |
| @Column | 열 매핑 |
| @Temporal | 날짜 타입 매핑
 LocalDateTime 타입일 경우, @Temporal 애너테이션은 생략 가능(LocalDateTime은 열의 TIMESTAMP 타입과 매핑) |
| @Enumerated | enum 타입 매핑 |
| @Lob | BLOB, CLOB  매핑 |
| @Transient | 특정필드를 컬럼에 매핑하지 않음 (매핑 무시)
임시 데이터를 메모리에서 사용하기 위한 용도로 사용 |

**attribute**

| 속성 | 기본값 | Description |
| --- | --- | --- |
| updatable | true | 수정 여부 지정  |
| nullable(DDL) | true | null 값의 허용여부 설정
false 설정시 DDL 생성시에 NOT NULL 제약조건 붙음 |
| columnDefinition(DDL) |  | @Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용 |
| length(DDL) | 255 | 열에 저장할 수 있는 문자 길이 |
| unique | false | 고유한 값만, 중복 불가 |
|  |  |  |

**애트리뷰트가 기본값을 사용할 경우 주의 사항**

- `@Column` 애너테이션이 생략되면 기본적으로 `nullable=true 임.`
- `int price not null`일 경우에는 `@Column(nullable=false)`라고 명시적으로 지정해야 함.

`@Enumerated` 애너테이션은 아래의 두 가지 타입

- `EnumType.ORDINAL` : enum의 순서를 나타내는 숫자를 테이블에 저장 → 순서가 일치하지 않게 되는 문제가 발생
- `EnumType.STRING` : enum의 이름을 테이블에 저장
- **`EnumType.STRING`을 사용하는 것을 권장**
- 예제
    
    ```java
    public enum Day {
        SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY
    }
    
    @Entity
    public class Event {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        @Enumerated(EnumType.ORDINAL)
        private Day day;
    
        // getters and setters
    }
    ```
    
    Enum 상수 변경 
    
    ```java
    public enum Day {
        MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY}
    ```
    
    0은 MONDAY를, 1은 TUESDAY를 의미하게 되어 잘못된 참조가 발생
    

## 엔티티간의 연관관계 매핑

**엔티티 클래스 간의 관계를 만들어주는 것이 바로 연관 관계 매핑**

### 방향 (단방향/양방향)

1. **단방향 연관 관계**
    
    한쪽 클래스만 다른 쪽 클래스의 참조 정보를 가지고 있는 관계를 **단방향 연관 관계**
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/bf036c97-027a-49c1-b7bc-db904eeb40b8"/>
    
    
    - Member 클래스가 Order 객체를 원소로 포함하고 있는 List 객체를 가지고 있으므로, Order를 참조
    - Order 입장에서는 Member 정보를 알 수 없음
    
2. **양방향 연관 관계**
    
    양쪽 클래스가 서로의 참조 정보를 가지고 있는 관계를 **양방향 연관 관계** <br>
    서로 다른 단방향 관계 2개임.

    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/87849c96-b64f-4378-91d8-36558b19a6e5"/>
    
    
    - 두 클래스가 모두 서로의 객체를 참조할 수 있으므로, Member는 Order 정보를 알 수 있고, Order는 Member 정보를 알 수 있음
    
3. JPA는 단방향 연관 관계와 양방향 연관 관계를 모두 지원하는 반면에 Spring Data JDBC는 단방향 연관 관계만 지원

### 다중성(다대일/일대다/일대일/다대다)

### 다대일[N:1] : @ManyToOne

1. **다대일 단방향 :  가장 많이 사용, 다(N)에 해당하는 클래스가 일(1)에 해당하는 객체를 참조할 수 있는 관계**를 의미
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9f035a62-5b00-45f5-ad43-0b7af6c4f427">
    
    - Order만 Member 객체를 참조할 수 있으므로 단방향 관계
    - 예제
        
        ```java
        @NoArgsConstructor
        @Getter
        @Setter
        @Entity(name = "ORDERS")
        public class Order {
        ...
            @ManyToOne   // (1)
            @JoinColumn(name = "MEMBER_ID")  // (2)
            private Member member;
        
            public void addMember(Member member) {
                this.member = member;
            }
        ```
        
        (1) `@ManyToOne` 애너테이션으로 다대일의 관계를 명시
        
        (2) `@JoinColumn` 애너테이션으로 **외래 키에 해당하는 열 이름 작성**
        
        - MEMBER 테이블의 기본키 열 이름이 “`MEMBER_ID`” 이기 때문에 동일하게 작성
            
            
    - 예제) 다대일에서 일대다를 추가해야 하는 경우
        
        ```java
         	   	  tx.begin();
                Member member 
        			        = new Member("hgd@gmail.com", "Hong Gil Dong",
                        "010-1111-1111");
         	   	  
         	   	  em.persist(member);
        
                Order order = new Order();
                order.addMember(member);     // (2)
                em.persist(order);           // (3)
        
                tx.commit();
        
        				// (4)
                Order findOrder = em.find(Order.class, 1L);
        
                // (5) 주문에 해당하는 회원 정보를 가져올 수 있다.
                System.out.println("findOrder: " + findOrder.getMember().getMemberId() +
                                ", " + findOrder.getMember().getEmail());
        ```
        
        (2)  order 객체에 member 객체를 추가, member 객체는 이 MEMBER_ID 같은 외래키의 역할
        
        (3) 에서 주문 정보를 저장
        
        (4) 등록한 회원에 해당하는 주문 정보를 조회
        
        (5)  `findOrder.getMember()`와 같이 주문에 해당하는 회원 정보를 가져와서 출력
        
        ✔️그러나 내가 주문한 주문 정보는 조회할 수 없음 → **일대다 매핑을 추가해 양방향 관계 만들어야 함.**
        
2. 다대일 양방향
    - 양쪽을 서로 참조, 다 에 해당하는 쪽이 객체, 일에 해당하는 쪽이 List
        - @ManyToOne   
         @JoinColumn(name = "MEMBER_ID")
        - @OneToMany(mappedBy = "member")
    - 만약에 새로운 주문이 발생하면 다대일 양방향에서는 서로 참조하고 있으니 서로를 update 해주어야 함.
    - 양쪽에 add  메서드 생성후 commit 하면 StackOverFlow 발생 - 서로가 서로를 호출하기 때문 ⇒ 종료시점 필요
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c3ad47a3-4cb1-4199-9a7a-76b61df00ad7"/>
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e45e6166-d677-4bf4-9ce0-db5bc6eef021" width=250>
        
    
    - 이렇게 해주어야 함!
        
        ```java
        public void addOrder(Order order) {
            orders.add(order); // Order 객체를 orders 리스트에 추가
            
            // Order 객체의 member 필드가 현재 Member 객체(this)를 가리키지 않는 경우
            if (order.getMember() != this) {
                order.addMember(this); 
                // Order 객체의 member 필드를 현재 Member 객체(this)로 설정
            }
        }
            
          public void addMember(Member member){
            this.member = member; 
            // Order 객체의 member 필드를 주어진 Member 객체로 설
        
            // Member 객체의 orders 리스트에 현재 Order 객체(this)가 포함되지 않은 경우
            if (!member.getOrders().contains(this)) {
                member.addOrder(this); 
                // Member 객체의 orders 리스트에 현재 Order 객체(this)를 추가
            }
        }
        ```
        
        새로운 주문을 했는데 주문목록에 멤버가 없다면 현재 멤버를 추가
        
        멤버의 주문목록에 새로운 주문이 등록되지 않았다면 새로운 주문을 추가.
        
        ```java
         tx.begin();
                Member member = new Member("hgd@gmail.com", "Hong Gil Dong",
                        "010-1111-1111");
                Order order = new Order();
        
                member.addOrder(order); // (1)
                order.addMember(member); // (2)
        
                em.persist(member);   // (3) 회원정보 저장
                em.persist(order);    // (4) 주문정보 저장
        
                tx.commit();
        
        				// (5) 1차 캐시에서 조회
                Member findMember = em.find(Member.class, 1L);
        
                // (6) 이제 주문한 회원의 회원 정보를 통해 주문 정보를 가져올 수 있다.
                findMember
                        .getOrders()
                        .stream()
                        .forEach(findOrder -> {
                            System.out.println("findOrder: " +
                                    findOrder.getOrderId() + ", "
                                    + findOrder.getOrderStatus());
                        });
        ```
        
        (1) member에 order를 추가하지 않아도 테이블에 order 자동 저장됨.
        
        But, order를 조회할 수 없음, 1차 캐시에 저장된 member는 order를 가지고 있지 않기 때문
        
        (2) order 객체에 member 객체를 추가,  member가 order의 외래키 역할을 하기 때문에 order 객체 저장 시, 반드시 필요 → 하지 않으면 주문정보의 MEBER_ID 는 Null이 됨.
        

### 일대다[1:N] : @OneToMany

1. 일대다 단방향 연관관계
    
     **일(1)에 해당하는 클래스가 다(N)에 해당하는 객체를 참조할 수 있는 관계를 의미**
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/0d86d5a1-913c-4cde-b850-c9f711cfb432"/>
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/fc265785-5ab0-4665-8d03-654da995a944"/>
    
    - Member만 List<Order> 객체를 참조할 수 있으므로 단방향 관계
    - 엔티티가 관리하는 외래 키가 다른 테이블에 있음
    - Order 클래스의 정보를 테이블에 저장하더라도 외래 키에 해당하는 MEMBER 클래스의 memberId 값이 없는 채로 저장됨 → 이런 문제점 때문에 잘 사용하지 않음.
    - 다대일 단방향 매핑을 먼저 한 후에 필요한 경우, 일대다 단방향 매핑을 추가해서 양방향 연관 관계를 만드는 것이 일반적
    - 예제
        
        ```java
        @Entity
        public class Member{
        ...
        @OneToMany
        private List<Order> orders = new ArrayList<>();
        ...
        }
        ```
        

### 일대일[1:1] : @OneToOne

### 다대다[N:M] : @ManyToMany
<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/40b7fb14-34ca-4864-a7a3-900d0d4094a1">

- 실무에서 거의 사용하지 않으므로 이런게 있다 정도만 참고!

### Comment
    
오늘 코드 따라지면서 Enum을 아무생각없이 enum으로 사용했는데 에러가나서 왜그런가 했더니 <br>
Enum은 자바에서 제공하는 기본 클래스(java.lang.Enum)이며,<br>
모든 enum 타입은 java.lang.Enum 클래스를 상속받으나, extends를 사용하고 있지 않음(사용못함) <br>
자동으로 컴파일러가 extend를 자동으로 추가해주기 때문이다! <br>
그리고 꼭 기억해야 할 것 하나 더는 Order는 Orderby로 인식하니까 Orders로 테이블명 바꿔줘야함.<br>

다른 것 보다 다대일 양방향에서 아직 헷갈려서 더 봐야할 것 같다..!
