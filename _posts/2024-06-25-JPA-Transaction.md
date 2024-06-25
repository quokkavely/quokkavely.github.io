---
layout : single
title : "[JPA] 영속성 전이,연관관계 매핑(일관성)"
categories: Spring
tag : [Spring, 실습, JPA, Transaction]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## Transaction Intro

- 애플리케이션에서 사용하는 데이터의 무결성을 보장하는 핵심적인 역할
- 이때동안 만든 커피주문 시스템에서 발생할 수 있는 Transaction 사례
    1. 주문 중 네트워크 오류로 결제 실패했는데 DB에 주문이 정상등록 되어 커피 수만큼 스탬프가 찍힌 경우
    2.  결제를 했는데 DB에 정상 등록이 되지 않아 회원이 커피를 받지 못했을 때
    3. 주문은 정상 등록 되었는데 스탬프 횟수만 DB에 업데이트 하는 중에 에러가 발생해서 누적이 되지 않았을 경우

### ACID 원칙

5월에 DB에서 했지만 중요하니까 한번 더 정리해봄!

1. 원자성(**A**tomicity)
    - 트랜잭션에서의 원자성이란 작업을 더 이상 쪼갤 수 없음을 의미
    - 따라서 논리적으로 하나의 작업으로 인식해서 둘 다 성공하든가 둘 다 실패하든가(All or Nothing)중에서 하나로만 처리되는 것이 보장되어야 함.

1. 일관성(**C**onsistency)
    - 일관성은 트랜잭션이 에러 없이 성공적으로 종료될 경우, 비즈니스 로직에서 의도하는 대로 일관성 있게 저장되거나 변경되는 것을 의미
    - 주문한 커피의 수만큼, 스탬프 횟수가 증가한다는 비즈니스 로직에 맞게 저장되거나 변경되어야 함. 만약에 3이 아닌 숫자로 증가한 값이 조회된다면 일관성에 위배되는 것

1. 격리성(**I**solation)
    - 격리성은 여러 개의 트랜잭션이 실행될 경우 각각 독립적으로 실행이 되어야 함을 의미
    - 예를 들어 우리가 컴퓨터에서 워드 작업을 하고 있고, 동시에 뮤직 플레이어로 음악을 듣고 있다면 우리 눈에는 보이지 않지만 CPU는 위 두 가지 프로세스를 아주 빠른 속도로 번갈아가면서 실행을 시키는 것
    - 이처럼 데이터베이스 역시 성능 향상을 목적으로 한 개 이상의 트랜잭션을 번갈아가면서 처리할 수 있는데, 이 경우 각 트랜잭션이 다른 트랜잭션에 영향을 주지 않고 독립적으로 실행이 되어야 한다는 것이 바로 격리성(**I**solation).

1. 지속성(Durability)
    - 트랜잭션이 완료되면 그 결과는 지속되어야 한다는 의미
    - 즉, 지속성은  데이터베이스가 종료되어도 데이터는 물리적인 저장소에 저장되어 지속적으로 유지되어야 한다는 의미

## 트랜잭션 커밋(commit)과 롤백(rollback)

### Commit

- 커밋(commit)은 모든 작업을 최종적으로 데이터베이스에 반영하는 명령어로써 commit 명령을 수행하면 변경된 내용이 데이터베이스에 영구적으로 저장됨.
- 만약 commit 명령을 수행하지 않으면 작업의 결과가 데이터베이스에 최종적으로 반영되지 않음
- commit 명령을 수행하면, 하나의 트랜젝션 과정은 종료

### 롤백(rollback)

- 롤백(rollback)은 작업 중 문제가 발생했을 때, 트랜잭션 내에서 수행된 작업들을 취소한다.
- 따라서 트랜잭션 시작 이 전의 상태로 되돌아 가게 됨.

> 인메모리 DB 특성상 (H2는 인메모리 DB다) , 인메모리 DB란? ↔
> 

### JPA의 tx.commit() 내부

🍒 JPA tx.commit() 호출 과정 이해를 위한 클래스 다이어그램 관계도

![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled.png)

- 추상화되어있어서 구현체를 쭉 지나가다보면 마지막에 Command가 있음.
- 우리가 JPA API를 사용해서 commit을 수행하는 작업은 너무나도 간단한 작업인데, 내부적으로는 아주 복잡한 과정을 거쳐서 최종적으로 commit 명령이 데이터베이스에 전달

**하나의 작업 단위로 묶어서 처리해야 되는 상황에서는 어떤 식으로 트랜잭션을 적용하면 좋을지에 대해서 꼭 고민해보기**

## 선언형 방식의 트랜잭션

🔑 요약

✔️ Class level과 Method level에 설정 가능, 둘 다 설정했을 경우에는 Method가 우선순위를 가지게 된다

✔️ ****Checked exception는 **@Transactional** 애너테이션만 추가해서는 rollback이 되지 않음 → 속성에 rollbackFor로 지정해 주어야 함

Ex ) @Transactional(rollbackFor = {SQLException.class, DataFormatException.class}) 

 ✔️

### 애너테이션 방식 @Transactional

1. class level에 적용
    
    ```java
    ...
    **import org.springframework.transaction.annotation.Transactional;**
    
    @Service
    **@Transactional**   // (1)
    public class MemberService {
        private final MemberRepository memberRepository;
    
        public MemberService(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        public Member createMember(Member member) {
            verifyExistsEmail(member.getEmail());
    
            return memberRepository.save(member);
        }
    		...
    }
    ```
    
    - MemberRepository의 기능을 이용하는 모든 메서드에 트랜잭션이 적용
    
2. JPA 로그레벨 설정
    
    ```yaml
    spring:
      h2:
        console:
          enabled: true
          path: /h2
      datasource:
        url: jdbc:h2:mem:test
      jpa:
        hibernate:
    ...
    ...
    
    logging:         # (1) 로그 레벨 설정
      level:
        org:
          springframework:
            orm:
              jpa: DEBUG
    ```
    
3. 콘솔 log 확인
    - application 실행하여 Postman으로 회원 등록 후 log 확인
    
    ```yaml
    ...
    2024-06-25 22:19:55.876 DEBUG 46820 --- [nio-8080-exec-4] o.s.orm.jpa.JpaTransactionManager        
    : Found thread-bound EntityManager [SessionImpl(1398140194<open>)] for JPA transactio      
      //(1)
    : **Creating new transaction with name** [org.springframework.data.jpa.repository.support.SimpleJpaRepository.save]
    
    : PROPAGATION_REQUIRED,ISOLATION_DEFAULT
      //(2)
    2024-06-25 22:19:55.887 DEBUG 46820 --- [nio-8080-exec-4] o.s.orm.jpa.JpaTransactionManager        
    : Initiating transaction commit
    2024-06-25 22:19:55.887 DEBUG 46820 --- [nio-8080-exec-4] o.s.orm.jpa.JpaTransactionManager        
    : Committing JPA transaction on EntityManager [SessionImpl(1398140194<open>)]
    2024-06-25 22:19:55.899 DEBUG 46820 --- [nio-8080-exec-4] o.s.orm.jpa.JpaTransactionManager        
    : Not closing pre-bound JPA EntityManager after transaction   //(3)
      //(4)
    2024-06-25 22:19:55.910 DEBUG 46820 --- [nio-8080-exec-4] o.j.s.OpenEntityManagerInViewInterceptor 
    : Closing JPA EntityManager in OpenEntityManagerInViewIntercepto
    ```
    
    - (1) :  CoffeeService의 createCoffee() 메서드가 호출되면서 새로운 트랜잭션이 생성
    - (2) : 트랜잭션에서 commit이 일어나고 있음
    - (3) : 트랜잭션이 종료됨
    - (4) : JPA의 EntityManager를 종료
4. roolback 동작 유무 확인
    - createMember() 메서드에서 회원 정보 저장하고 메서드가 종료되기 전에 강제로 예외 발생시키기.
        
        ![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled%201.png)
        
         롤백 후 예외 던짐.
        
        ![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled%202.png)
        
        ![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled%203.png)
        
    
    ### Checked Exception
    
    Checked 예외로 할려면 실행조차 되지 않기 때문에 throw로 던져야 실행할 수 잇음.
    
    이 예외도 롤백해주세요 라고 attribute 넣어야 함 →  안 넣으면 롤백 안됨!
    
    ![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled%204.png)
    

### Class Level 과  Method Level에 적용시

1. 클래스 레벨과 메서드 레벨의 트랜잭선 적용 순서
    - 클래스 레벨에만 `@Transactional`이 적용된 경우
        - 클래스 레벨의 `@Transactional` 애너테이션이 메서드에 일괄 적용됨
    - 클래스 레벨과 메서드 레벨에 함께 적용된 경우
        - 메서드 레벨의 `@Transactional` 애너테이션이 적용됨
        - 만약 메서드 레벨에 `@Transactional` 애너테이션이 적용되지 않았을 경우, 클래스 레벨의 `@Transactional` 애너테이션이 적용
2. 메서드레벨에서 @Transactional(readOnly = true) 적용
    
    → 읽기 전용으로 설정 하는 것 :
    
    → 이유 : commit이 되지 않아 영속성 컨텍스트에 flush도 하지 않고 스냅샷을 생성하지 않으므로 불필요한 추가동작을 줄일 수 있음.
    
    → 즉 조회 메서드에는 readOnly=true 로 설정하여 JPA가 자체적으로 성능 최적화 과정을 거치도록 하는 것을 권장!
    

### 여러 작업이 하나의 트랜잭션으로 묶일 경우

![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled%205.png)

- OrderService에서 createOrder() 메서드를 호출할 경우, 내부에서 **주문 정보 저장을 위한 트랜잭션이 하나 시작**되며, 다음으로 memberService.updateStamp() 메서드 호출을 통해서 MemberService에서 **스탬프 업데이트를 위한 트랜잭션이 하나 더 시작됨**
- 만약 독립적으로 실행된다면 updateStamp() 동작에서 예외가 발생할 경우 스탬프 숫자는 업데이트되지 않았는데 주문 정보는 저장되는 원치 않는 상황이 발생함

![Untitled](%5BJPA%5D%20@Transactional%20ec791fadcff3441fb6875ba0120bbdb6/Untitled%206.png)

- 트랜잭션이 하나로 묶이면 `MemberService`의 `updateStamp(`) 메서드 작업을 처리하는 도중에 예외가 발생해도 두 클래스에서 작업을 처리하는 메서드들이 모두 하나의 트랜잭션 경계 내에 있으므로 모두 rollbacK 됨

### Attribute 설정으로 Transaction 전파와 격리가 가능.

**🔎 트랜잭션 전파 (Transaction Propagation)**

트랜잭션의 경계에서 진행 중인 트랜잭션이 존재할 때 또는 존재하지 않을 때, 어떻게 동작할 것인지 결정하는 방식을 의미

1. **@Transactional(propagation = Propagation.REQUIRED)**
    - 현재 진행 중인 트랜잭션이 존재하면 해당 트랜잭션을 사용하고, 존재하지 않으면 새 트랜잭션을 생성하도록 해줌
    - 일반적으로 가장 많이 사용되는 propagation 유형의 디폴트 값
2. **@Transactional(Propagation.REQUIRES_NEW)**
    - 이미 진행 중인 트랜잭션과 무관하게 새로운 트랜잭션이 시작
    - 기존에 진행 중이던 트랜잭션은 새로 시작된 트랜잭션이 종료할 때까지 중지됨
3. **@Transactional(Propagation.MANDATORY)**
    - 진행 중인 트랜잭션이 없으면 예외를 발생시킴
4. **@Transactional(Propagation.NOT_SUPPORTED)**
    - 트랜잭션을 필요로 하지 않음을 의미
    - 진행 중인 트랜잭션이 있으면 메서드 실행이 종료될 때까지 진행 중인 트랜잭션은 중지되며, 메서드 실행이 종료되면 트랜잭션을 계속 진행
5. **@Transactional(Propagation.NEVER)**
    - 트랜잭션을 필요로 하지 않음을 의미
    - 진행 중인 트랜잭션이 존재할 경우에는 예외를 발생시킴\

**🔎 트랜잭션 격리 레벨(Isolation Level)**

트랜잭션은 다른 트랜잭션에 영향을 주지 않고, 독립적으로 실행되어야 하는 격리성이 보장되어야 하는데 Spring은 이러한 격리성을 조정할 수 있는 옵션을 `@Transactional` 애너테이션의 isolation 애트리뷰트를 통해 제공함

1. **@Transactional(Isolation.DEFAULT)**
    - 데이터베이스에서 제공하는 기본 값
2. **@Transactional(Isolation.READ_UNCOMMITTED)**
다른 트랜잭션에서 커밋하지 않은 데이터를 읽는 것을 허용함
3. **@Transactional(Isolation.READ_COMMITTED)**
다른 트랜잭션에 의해 커밋된 데이터를 읽는 것을 허용함
4. **@Transactional(Isolation.REPEATABLE_READ)**
트랜잭션 내에서 한 번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회되도록 함
5. **@Transactional(Isolation.SERIALIZABLE)**
동일한 데이터에 대해서 동시에 두 개 이상의 트랜잭션이 수행되지 못하도록 함
    
    트랜잭션의 격리 레벨은 일반적으로 데이터베이스나 데이터소스에 설정된 격리 레벨을 따르는 것이 권장되므로, 이러한 격리 레벨이 있다고 이해하면 된다.
    

### AOP 방식의 트랜잭션 적용

비즈니스 로직에 적용하지 않고, 트랜잭션을 적용하는 방법

```java
import org.springframework.aop.Advisor;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionManager;
import org.springframework.transaction.interceptor.*;
import java.util.HashMap;
import java.util.Map;

// (1)
@Configuration
public class TxConfig {
    private final TransactionManager transactionManager;

		// (2)
    public TxConfig(TransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    @Bean
    public TransactionInterceptor txAdvice() {
        NameMatchTransactionAttributeSource txAttributeSource =
                                    new NameMatchTransactionAttributeSource();

				// (3)
        RuleBasedTransactionAttribute txAttribute =
                                        new RuleBasedTransactionAttribute();
        txAttribute.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

				// (4)
        RuleBasedTransactionAttribute txFindAttribute =
                                        new RuleBasedTransactionAttribute();
        txFindAttribute.setPropagationBehavior(
                                        TransactionDefinition.PROPAGATION_REQUIRED);
        txFindAttribute.setReadOnly(true);

				// (5)
        Map<String, TransactionAttribute> txMethods = new HashMap<>();
        txMethods.put("find*", txFindAttribute);
        txMethods.put("*", txAttribute);

				// (6)
        txAttributeSource.setNameMap(txMethods);

				// (7)
        return new TransactionInterceptor(transactionManager, txAttributeSource);
    }

    @Bean
    public Advisor txAdvisor() {
				// (8)
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.codestates.coffee.service." +
                "CoffeeService.*(..))");

        return new DefaultPointcutAdvisor(pointcut, txAdvice());  // (9)
    }
}

```

**AOP 방식으로 트랜잭션을 적용하는 순서**

1. **AOP 방식으로 트랜잭션을 적용하기 위한 Configuration 클래스 정의**
    
    (1)과 같이 @Configuration 애너테이션을 추가하며 Configuration 클래스를 정의
    
2. TransactionManager DI
    
    애플리케이션에 트랜잭션을 적용하기 위해서는 TransactionManager  객체가 필요 → (2)와 같이 TransactionManager  객체를 DI 받습니다.
    
3. **트랜잭션 어드바이스용 TransactionInterceptor 빈 등록**
    
    Spring에서는 TransactionInterceptor를 이용해서 대상 클래스 또는 인터페이스에 트랜잭션 경계를 설정하고 트랜잭션을 적용할 수 있다.
    
    - 트랜잭션 애트리뷰트 지정
        - 트랜잭션 애트리뷰트는 메서드 이름 패턴에 따라 구분해서 적용 가능하기 때문에 (3), (4)와 같이 트랜잭션 애트리뷰트를 설정할 수 있다.
        - (3)은 조회 메서드를 제외한 공통 트랜잭션 애트리뷰트이고, (4)는 조회 메서드에 적용하기 위한 트랜잭션 애트리뷰트

- 트랜잭션을 적용할 메서드에 트랜잭션 애트리뷰트 매핑
    - 설정한 트랜잭션 애트리뷰트는 (5)와 같이 Map에 추가하는데, Map의 key를 메서드 이름 패턴으로 지정해서 각각의 트랜잭션 애트리뷰트를 추가
    - 트랜잭션 애트리뷰트를 추가한 Map 객체를 (6)과 같이 txAttributeSource.setNameMap(txMethods)으로 넘겨줌

- TransactionInterceptor 객체 생성
    - (7)과 같이 TransactionInterceptor의 생성자 파라미터로 transactionManager와 txAttributeSource를 전달

1. **Advisor 빈 등록**
    - 포인트 컷 지정
        - 이제 트랜잭션 어드바이스인 `TransactionInterceptor`를 타깃 클래스에 적용하기 위해 포인트 컷을 지정
        - (8)과 같이 AspectJExpressionPointcut 객체를 생성한 후, 포인트 컷 표현식으로 CoffeeService 클래스를 타깃 클래스로 지정
    
    - Advisor 객체 생성
        
        마지막으로 (9)와 같이 `DefaultPointcutAdvisor`의 생성자 파라미터로 포인트컷과 어드바이스를 전달해 줍니다.
        

AOP가 적용되어있다는 것은 프록시패턴이 적용되어있다는것.

프록시객체는 원본객체와 달리 부가기능이 포함되어잇음.

SPRING AOP는 가상의 프록시객체를 만들어서 중간에 낚아 챈 후  프록시객체가 실행됨 → 얘를 미리 만들어 놓지 않고 요청이 왔을 때 동적으로 만듬. 가상의 인위의 객체를 만들고 다쓰면 지워버림

내부적으로 GCLIB라는 기술을 사용,,

AOP를 써서 TRANSACTION을 사용하고 있다는 것을 알아야 함. (Spring AOP ⇒ SPRING Transaction)

프록시는 앞단에 먼저 처리하는 과정을 처리하는 가상의 서버

로그를 남기는 것도 프록시 - AOP가 적용된 것.

### (참고) Springboot을 사용하지 않을 경우

```java
@Configuration
@EnableTransactionManagement
public class JpaConfig{

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(){
        final LocalContainerEntityManagerFactoryBean em =
				new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource());
        ...
        ...

        return em;
    }

		// (1)
    @Bean
    public DataSource dataSource() {
        final DriverManagerDataSource dataSource = new DriverManagerDataSource();

				...
				...

        return dataSource;
    }

    @Bean
    public PlatformTransactionManager transactionManager(){
        JpaTransactionManager transactionManager
                = new JpaTransactionManager();    // (2)
        transactionManager.setEntityManagerFactory(
                entityManagerFactoryBean().getObject() );
        return transactionManager;
    }
}

```

- (1)과 같이 데이터베이스 커넥션 정보를 포함하고 있는 Datasource가 기본적으로 필요
- Spring에서 트랜잭션은 기본적으로 `PlatformTransactionManager`에 의해 관리되며, `PlatformTransactionManager` 인터페이스를 구현해서 해당 데이터 액세스 기술에 맞게 유연하게 트랜잭션을 적용할 수 있도록 추상화되어 있음
- 데이터 액세스 기술이 JPA이기 때문에 (2)와 같이 `PlatformTransactionManager`의 구현 클래스인 `JpaTransactionManager`를 사용

동기와 비동기 차이

throw 와 throws 차이