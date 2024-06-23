---
layout : single
title : "JDBC-intro"
categories: Spring
tag : [Spring, DB, 개념정리]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## JDBC란?

1. **JDBC(Java Database Connectivity)**
    - Java 기반 애플리케이션의 코드 레벨에서 사용하는 데이터를 데이터베이스에 저장 및 업데이트하거나 반대로 데이터베이스에 저장된 데이터를 Java 코드 레벨에서 사용할 수 있도록 해주는 Java에서 제공하는 **표준 사양(또는 명세, Specification)**
    - JDBC API를 사용해서 다양한 벤더(Oracle, MS SQL, MySQL 등)의 데이터베이스와 연동가능

### JDBC 동작 흐름

간단하게 말하면 Java 애플리케이션에서 JDBC API를 이용해 적절한 데이터베이스 드라이버를 로딩한 후, Database와 interaction 한다

1. **JDBC Driver**
    - 데이터베이스와의 통신을 담당하는 인터페이스
    - Oracle이나 MS SQL, MySQL 같은 다양한 벤더에서는 해당 벤더에 맞는 JDBC 드라이버를 구현해서 제공을  함,
    - 우리는 이 JDBC 드라이버의 구현체를 이용해서 특정 벤더의 데이터베이스에 액세스 할 수 잇음
2. **JDBC API**
    - **JDBC API 사용 흐름**
    
      <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/a2685f8e-f019-4ed6-b9ca-6f70f7261714" width=500/>
    
    예전에 DB할 때 마지막에 JDBC에서 배웠던 것.
    
    자세히 알 필요는 없지만 간단히 보고 가기
    
    1. JDBC 드라이버 로딩
        - 사용하고자 하는 JDBC 드라이버를 로딩
        - JDBC 드라이버는 DriverManager라는 클래스를 통해서 로딩됨
    2. Connection 객체 생성
        - JDBC 드라이버가 정상적으로 로딩되면  DriverManager를 통해 데이터베이스와 연결되는 세션(Session)인 Connection 객체를 생성
    3. Statement 객체 생성
        - Statement 객체는 작성된 SQL 쿼리문을 실행하기 위한 객체로써 객체 생성 후에 정적인 SQL 쿼리 문자열을 입력으로 가짐
    4. Query 실행
        - 생성된 Statement 객체를 이용해서 입력한 SQL 쿼리를 실행합니다.
    5. ResultSet 객체로부터 데이터 조회
        - 실행된 SQL 쿼리문에 대한 결과 데이터 셋입니다.
    6. ResultSet 객체 Close, Statement 객체 Close, Connection 객체 Close
    - JDBC API를 통해 사용된 객체들은 사용 이후에 사용한 순서의 역순으로 차례로 Close 해야함.
3. **Connection Pool**
    - 데이터베이스 Connection 객체를 미리 만들어서 보관하고 애플리케이션이 필요할 때 이 Connection을 제공해 주는 역할을 하는 Connection 관리자
    - DB와의 연결을 위해 Connection객체 생성하하는 작업은 비용이 많이 드니 미리 만들어서 보관하고 필요할 때 제공함.
    - 원래 Apache Commons DBCP(Database Connection Pool, DBCP)를 주로 사용했지만
    - Spring Boot 2.0부터는 성능면에서 더 나은 이점을 가지고 있는 HikariCP를 기본 DBCP로 채택
    
      <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/4eb12e74-371d-4e4c-a9a7-4f2aeb22a24d" width=500/>
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/03aee5a9-a4fa-4cfd-b525-0f68327e2293" width=500/>

## Spring Data JDBC란?

### SQL 중심 기술

- 앞에서 언급한 **mybatis**와 **Spring JDBC**는 대표적인 SQL 중심 기술
- Spring JDBC의 JdbcTemplate 사용 예
  
    ```java
    Member member = this.jdbcTemplate.queryForObject(
        "select * from member where member_id=?", 1, Member.class);
    
    ```
    
- Java 진영에서는 SQL 중심의 기술에서 객체(Object) 중심의 기술로 지속적으로 이전을 하고 있는 추세

### 객체(Object) 중심 기술

- 객체(Object) 중심 기술은 데이터베이스에 접근하기 위해서 **SQL 쿼리문을 직접적으로 작성하기보다는** 데이터베이스의 테이블에 데이터를 저장하거나 조회할 경우, Java 객체(Object)를 이용해 애플리케이션 내부에서 이 Java 객체(Object)를 SQL 쿼리문으로 자동 변환 한 후에 데이터베이스의 테이블에 접근함.
- 객체(Object) 중심의 데이터 액세스 기술을 ORM(Object-Relational Mapping)
- Java에서 대표적인 ORM 기술이 바로 JPA(Java Persistence API)

### Spring Data JDBC

Spring Data JDBC는 JPA처럼 ORM 기술을 사용하지만 JPA의 기술적 복잡도를 낮춘 기술

1. **라이브러리 추가**
   
    ```groovy
    dependencies {
    	...
    	...
    	implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'
    	runtimeOnly 'com.h2database:h2'
    }
    ```
    
    - **인메모리(In-memory) DB**인 **H2**를 사용하기 위해 의존 라이브러리 설정에 추가
      
        > 인메모리 DB란?
        **메모리 안에 데이터를 저장하는 데이터베이스**
        
        ✔️ 인메모리(In-memory) DB를 사용하는 이유
        > 
        > - 로컬 개발 환경에서는 인메모리(In-memory) DB를 주로 사용
        > - 왜 ? 애플리케이션의 테스트 측면에서 많은 이점
        > - 테스트가 끝나고 나면 데이터베이스의 테이블에 남아있는 데이터는 깨끗이 비워져 있는 것이 좋음

1. **H2 관련 설정 추가**
   
    ```yaml
    spring:
      h2:
        console:
          enabled: true
    
    ```
    
    - 웹 브라우저 상(H2 콘솔)에서 H2 DB에 접속한 후, 데이터베이스를 관리가능
    - yml 파일에서는 indent를 주어서 depth를 설정할 때에는 Tab(탭) 키를 사용해서 일관성을 유지하기
2. **H2 Database에 접속** 
    1. localhost:8080/h2-console
    2. 애플리케이션 로그에 출력되는 JDBC URL을 확인하여 JDBC URL에 입력 후 Connect
    3. **H2 DB 설정 추가 →** JDBC URL이 랜덤으로 바뀌는 불편함 해결하기 위함
       
        ```yaml
        spring:
          h2:
            console:
              enabled: true
              path: /h2     # (1) Context path 변경
          datasource:
            url: jdbc:h2:mem:test     # (2) JDBC URL 변경
        
        ```
        
        - H2 콘솔의 접속 URL Context path를 조금 더 간결하게 ‘`/h2`’로 설정
        - JDBC URL이 매번 랜덤하게 바뀌지 않도록 ‘`jdbc:h2:mem:test`’로 설정
3. **클래스 구현 [새로 배운 것만 정리함.]**
    - MessageDto(DTO 클래스)
    - MessageController
    - MessageMapper
    - **MessageService**
      
        ```java
        package com.springboot.hello_world.service;
        
        import com.springboot.hello_world.entity.Message;
        import com.springboot.hello_world.repository.MessageRepository;
        import org.springframework.stereotype.Service;
        
        @Service
        public class MessageService {
            private final MessageRepository messageRepository;
        
            public MessageService(MessageRepository messageRepository) {
                this.messageRepository = messageRepository;
            }
        
            public Message createMessage(Message message){
                return messageRepository.save(message); 
            }
        }
        
        ```
        
        1. MessageRepository 인터페이스는 MessageService 클래스에서 DI를 통해 주입받음
        2. messageRepository.save(message)와 같이 save() 메서드를 사용
           
            1)  Sava 메서드는 `CrudRepository`가 이 작업을 대신해 주는 역할
            
            2)  쿼리문으로 데이터 저장할 때 기능 (==insert into)
            
        
    - **Message** (Entity 클래스)
      
        ```java
        import lombok.Getter;
        import lombok.Setter;
        import org.springframework.data.annotation.Id;
        
        @Getter
        @Setter
        public class Message {  // (1)
            @Id    // (2)
            private long messageId;
            private String message;
        }
        
        ```
        
        1)  Message라는 클래스 명은 데이터베이스의 테이블 명
        
        2)   @Id애너테이션을 추가한 멤버 변수는 해당 엔티티의 **고유 식별자 역할 =primary key**
        
    - **MessageRepository**
      
        ```java
        import org.springframework.data.repository.CrudRepository;
        
        public interface MessageRepository extends CrudRepository<Message, Long> {
        }
        
        ```
        
        1. 데이터 액세스 계층에서 데이터베이스와의 연동을 담당하는 Repository
        2. **CrudRepository**
            1. CrudRepository라는 인터페이스를 상속, 제너릭 타입이 `<Message, Long>`으로 선언
            2. 데이터베이스에 CRUD(데이터 생성, 조회, 수정, 삭제) 작업을 진행하기 위해 Spring에서 지원해 주는 인터페이스
            3. CrudRepository를 상속함으로써 query를 따로 작성할 일도 없고 container에 bean을 등록하지 않아도 CrudRepository가 가지고있어서 괜찮음.
               
                <img src="![image-20240618101334129](../assets/images/image-20240618101334129.png)" width=500 />
    
    1. application. yml 추가 설정 - **H2 DB에 MESSAGE 테이블 생성**
       
        ```xml
        spring:
          h2:
            console:
              enabled: true
              path: /h2
          datasource:
            url: jdbc:h2:mem:test
          **sql:
            init:
              schema-locations: classpath*:db/h2/schema.sql**
          jpa:
            hibernate:
              ddl-auto: create  # (1) 스키마 자동 생성
            show-sql: true      # (2) SQL 쿼리 출력
        
        ```
        
        schema.sql
        
        ```java
        CREATE TABLE IF NOT EXISTS MESSAGE(
            message_id BIGINT NOT NULL AUTO_INCREMENT,
            message varchar(100) NOT NULL,
            PRIMARY KEY (message_id)
        );
        ```
        
        - **인메모리 DB를 사용할 경우**, 애플리케이션이 실행될 때마다 schema.sql 파일의 **스크립트가 매번 실행된다는 것 기억하기.**
        
    2. H2 콘솔에 접속해서 MESSAGE 테이블 확인
       
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e449adc4-4e8a-41d2-a0b1-5f228b12347d" width=500 />
        
    3. POSTRUN
       
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3992c40a-4e00-4e85-b3f9-d67353926afe" width=500 />
        
        POST요청으로   message만 조금 변경함
        
    4. MESSAGE table 조회
       
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3e6e2393-d04e-4982-8d02-90199ed8935a" width=500 />
        
        POST 요청이 잘 되었는지 Message table 조회
        

## Spring Data JDBC - Entity, table설계

### DDD(Domain Driven Design) : 도메인 위주의 설계 기법

1. Domain이란
    1.  **도메인이란 용어 자체는 한 마디로 우리가 실제로 현실 세계에서 접하는 업무의 한 영역**
    2. 비즈니스 업무 영역을 의미하는 도메인 지식(Domain Knowledge)들을 서비스 계층에서 비즈니스 로직으로 구현해야 한다.
2. Aggreagate
    1. 비슷한 업무 도메인들의 묶음
    2. **애그리거트(Aggregate)는 한마디로 비슷한 범주의 연관된 업무들을 하나로 그룹화해놓은 그룹이라고 생각하기**
3. Aggregate Root
    1. 각각의 애그리거트에는 해당 애그리거트를 대표하는 도메인이 존재함
    2. 하나의 애그리거트를 대표하는 도메인을 DDD에서는 애그리거트 루트(Aggregate Root)
    3. 어떤 특정 집단이나 그룹의 대표라고 생각하기
    4. 애그리거트의 선정기준 ⇒ 도메인들과 직간접적으로 연관되어 있는 도메인
    5. **데이터베이스의 테이블 간 관계로 보자면, 애그리거트 루트는 부모 테이블이 되고, 애그리거트 루트가 아닌 다른 도메인들은 자식 테이블이 되는 것과 비슷..**
    6. 배달주문 앱을 세분화 하면
       
        <img src="" width=500 />
        
        - **회원 애그리거트, 주문 애그리거트, 음식 애그리거트, 결제 애그리거트**라 할 수 있음.

### 엔티티 구현

1. **엔티티 설계 확인 - 커피 주문 샘플 애플리케이션**
   
    <img src="" width=500 />
    
    - Mebmer : Order =1: N
        - Order에 member_Id 외래키를 추가하여 조인.
    - Order : Coffee =N:M →JoinTable로 OrderCoffee
        - OrderCoffee에 order_Id와 coffee_Id 외래키 추가하여 조인
    - DB에서는 다에 해당하는 테이블이 참조키를 가졌다면 클래스에서는 1에 해당하는 클래스에서 객체 참조 리스트를 가지게 됨, (List<T>)
2. **애그리거트 객체 매핑 규칙**
    - Aggregate : 관련된 객체를 모아둔 하나의 단위로 Value Object와 Entity로 구성
    - Aggregate 중심에는 Aggregate Root가 존재
    
    (1) **모든 엔티티 객체의 상태는 애그리거트 루트를 통해서만 변경가능**
    
    → **어떤 식으로든 애그리거트 루트가 나머지 모든 엔티티에 대한 객체를 직간접적으로 참조할 수 있다는 의미**
    
    <img src="" width=500 />
    
    (2) 하나의 동일한 애그리거트 내에서의 엔티티 객체 참조
    
    - 동일한 하나의 애그리거트 내에서는 엔티티 간에 객체로 참조
    
    (3) 애그리거트 루트 대 애그리거트 루트 간의 엔티티 객체 참조
    
    - **애그리거트 루트 간의 참조는 객체 참조 대신에 ID로 참조한다.**
    - 1대1 또는 1대N 관계일 때 테이블 간의 외래키 방식과 동일하다.
    - N대 N 관계일 때는 외래키 방식인 ID 참조와 객체 참조 방식이 함께 사용됨

### 엔티티 구현 - 예제

1. Order
   
    ```java
    package com.springboot.order.entity;
    
    import lombok.Getter;
    import lombok.Setter;
    import org.springframework.data.annotation.Id;
    import org.springframework.data.relational.core.mapping.MappedCollection;
    import org.springframework.data.relational.core.mapping.Table;
    
    import java.time.LocalDateTime;
    import java.util.LinkedHashSet;
    import java.util.Set;
    
    @Getter
    @Setter
    @Table("ORDERS")
    public class Order {
        @Id
        private long orderId;
    
        // 테이블 외래키처럼 memberId를 추가한다.
        private long memberId;
    
        // (1)
        @MappedCollection(idColumn = "ORDER_ID")
        private Set<OrderCoffee> orderCoffees = new LinkedHashSet<>();
    
    		...
    		...
    }
    ```
    
    1. Order 클래스는 애그리거트 루트
    2. long memberId로 외래키를 표현했는데 Spring Data JDBC에서는 AggregateReference 라는 클래스를 이용해 아래 코드와 같이 외래키 표현 가능
       
        ```java
        private AggregateReference<Member, Long> memberId;
        ```
        
    3. ✅ @MappedCollection 애너테이션의 역할
        - 엔티티 클래스 간에 연관 관계를 맺어주는 정보를 의미
        - ⭐ i**dColumn** 애트리뷰트는 자식 테이블에 추가되는 **외래키에 해당되는 열명**을 지정
        - ⭐ **keyColumn** 애트리뷰트는 **외래키를 포함하고 있는 테이블의 기본키 열명**을 지정
            - ex) ORDERS 테이블의 자식 테이블인 ORDER_COFFEE 테이블의 기본키는 **ORDER_COFFEE_ID** 이므로, `keyColumn`의 값이 ‘ORDER_COFFEE_ID
    
2. OrderCoffee - 주문 애그리거트 내에 있는 엔티티 클래스
   
    ```java
    package com.springboot.order.entity;
    
    import lombok.Builder;
    import lombok.Getter;
    import org.springframework.data.annotation.Id;
    import org.springframework.data.relational.core.mapping.Table;
    
    @Getter
    @Builder
    @Table("ORDER_COFFEE") // (1)
    public class OrderCoffee {
        @Id
        private long orderCoffeeId;
        private long coffeeId; // (2)
        private int quantity; // (3)
    }
    
    ```
    
    1) @Table
    
    - @Table 애너테이션을 추가하지 않으면 기본적으로 클래스명이 테이블의 이름과 매핑됨
    - `@Table("ORDER_COFFEE")`와 같이 테이블 이름을 변경
    
    2)  coffeeId를 외래키처럼 추가
    
    3) @Builder -> 내일 업로드 하기!
    
3. Order
   
    ```java
    import com.springboot.member.entity.Member;
    import lombok.Getter;
    import lombok.Setter;
    import org.springframework.data.annotation.Id;
    import org.springframework.data.relational.core.mapping.MappedCollection;
    import org.springframework.data.relational.core.mapping.Table;
    
    import java.time.LocalDateTime;
    import java.util.LinkedHashSet;
    import java.util.Set;
    
    @Getter
    @Setter
    @Table("ORDERS")
    public class Order {
        @Id
        private long orderId;
    
        // 테이블 외래키처럼 memberId를 추가한다.
        private long memberId;
    
        @MappedCollection(idColumn = "ORDER_ID")
        private Set<OrderCoffee> orderCoffees = new LinkedHashSet<>();
    
        // (1)
        private OrderStatus orderStatus = OrderStatus.ORDER_REQUEST;
    
        // (2)
        private LocalDateTime createdAt = LocalDateTime.now();
    
        // (3)
        public enum OrderStatus {
            ORDER_REQUEST(1, "주문 요청"),
            ORDER_CONFIRM(2, "주문 확정"),
            ORDER_COMPLETE(3, "주문 완료"),
            ORDER_CANCEL(4, "주문 취소");
    
            @Getter
            private int stepNumber;
    
            @Getter
            private String stepDescription;
    
            OrderStatus(int stepNumber, String stepDescription) {
                this.stepNumber = stepNumber;
                this.stepDescription = stepDescription;
            }
        }
    }
    
    ```
    
    - 주문 정보가 저장될 때 기본 값은 ORDER_REQUEST (주문 요청)
    - 주문이 등록되는 시간 정보를 나타내는 멤버 변수이며, LocalDateTime 타입
        - [https://docs.oracle.com/javase/8/docs/api/java/time/LocalDate.html](https://docs.oracle.com/javase/8/docs/api/java/time/LocalDate.html)
    - OrderStatus enum
        - OrderStatus enum이 Order 클래스의 멤버로 포함이 되어 있는 이유는 OrderStatus는 주문을 위한 전용 상태 값으로 사용할 수 있기 때문
        - 다른 기능에서도 사용할 가능성이 있다면 클래스 외부로 분리시킬 수 있겠지만 현재로서는 특별히 그럴 이유가 없음
    - coffee와 order은 연결하지 않음, 사용하지 않을것이라서.
      
    
4.  `src/main/resources/db/h2/schema.sql` 파일에 테이블 생성 스크립트 추가하여 애플리케이션 실행시 테이블 생성되도록 하기.

## Spring Data JDBC - Service, Repository

### Repository

1. Spring Data JDBC, Spring Data JPA에서는 데이터 액세스 계층에서 데이터베이스와 상호작용하는 역할을 하는 인터페이스를 **리포지토리(Repository)라고 함.**
   
    > **리포지토리(Repository)**라는 용어는 DDD(Domain Driven Design, 도메인 주도 설계)에서 사용하는 용어
    > 
2. **쿼리 메서드(Query Method)**
    - Spring Data JDBC에서는 쿼리 메서드를 이용해서 SQL 쿼리문을 사용하지 않고 데이터베이스에 질의가능
    - 사용법 ⇒ ‘find + By + SQL 쿼리문에서 WHERE 절의 열명 + (WHERE 절 열의 조건이 되는 데이터) ’ 형식
    - WHERE 절의 조건 열을 여러 개 지정하고 싶다면 ‘And’를 사용
    - Ex) findByEmailAndName(String email, String name)
    - 반드시 엔티티 클래스의 멤버 변수명을 적어주어야 함.
    - 약 Member 엔티티 클래스에 firstName이라는 멤버 변수, 테이블에 있는 FIRST_NAME이라는 열명과 매핑이 된다고 가정하면  findByFirstName이 되어야 함.
3. 예제
   
    ```java
    import com.springboot.coffee.entity.Coffee;
    import org.springframework.data.jdbc.repository.query.Query;
    import org.springframework.data.repository.CrudRepository;
    
    import java.util.Optional;
    
    public interface CoffeeRepository extends CrudRepository<Coffee, Long> {
    		//(1)
        Optional<Coffee> findByCoffeeCode(String coffeeCode);
    		//(2)
        @Query("SELECT * FROM COFFEE WHERE COFFEE_ID = :coffeeId")
        Optional<Coffee> findByCoffee(Long coffeeId);
    }
    
    ```
    
    - CrudRepository를 상속 → CrudRepository 인터페이스를 통해서 편리하게 데이터를 데이터베이스의 테이블에 저장, 조회, 수정, 삭제 가능
    - Coffee 엔티티 클래스, `Long`은 `Member` 엔티티 클래스에서 `@Id` 애너테이션이 붙은 멤버 변수의 타입
    - Spring Data JDBC에서는 **Optional**을 지원하기 때문에 리턴 값을 Optional로 래핑 가능
    - (1)은 WHERE 절에서 COFFEE_CODE를 조건으로 질의하게 해주는 쿼리 메서드
    - (2) @Query 애너테이션 :  COFFEE 테이블에 질의하기 위함,
        - 실제로는  CrudRepository 인터페이스에 내장되어 있는 findById(ID id)를 사용하는 것이 좋음.
        - findById(ID id)는 테이블에서 기본키를 WHERE절의 조건으로 지정해 데이터를 조회할 수 있는 편리한 쿼리메서드
        - findBy~ 이렇게 작성하면 알아서 찾아줌-> 카멜케이스로 작성 필요


<br>

### COMMENT
 뭔가 훅훅 지나가서 따라가고 이해하느라 시간이 훅훅 지나간다..
 부디 잘 따라가서 버벅거리지 말자.