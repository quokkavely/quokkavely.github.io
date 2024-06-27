---
layout : single
title : "[Spring] UnitTest"
categories: Spring
tag : [Spring, 실습, JPA, Transaction]
author_profile: true
---

# 단위 테스트(Unit Test)

## TestIntro

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3e61cadc-746f-4ded-84c3-59dab3d005a4" width=200/>

main과 test 가 분리 되어 있는 이유는 test는 bulid 앞에서 test만 하고 실제 패키징에는 포함되지 않음.

test 코드는 꼭 test 내에서만 작성해야 한다~!

----

**테스트는 어떤 대상에 대한 일정 기준을 정해놓고, 그 대상이 정해진 기준에 부합하는지 부합하지 못하는지를 검증하는 과정**

1. 테스트를 해야 하는 이유
    - 이때 동안은 Postman으로 검증하는 비효율적인 요청을 했음.
    - 테스트를 제대로 잘 거쳐서 테스트 대상이 검증 과정에 잘 통과하게 만들어 최대한 더 나은 결과를 얻기 위해서
2. 테스트 케이스(Test Case)란?
    - 테스트를 위한 입력 데이터, 실행 조건, 기대 결과를 표현하기 위한 명세를 의미하는데,
    - 한마디로 메서드 등 하나의 단위를  테스트하기 위해 작성하는 테스트 코드
3. 테스트 분류
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/8418c78e-7df7-4f20-919d-af8c1e914e14" width=500/>
    
    1️⃣ 기능 테스트
    
    - 애플리케이션을 사용하는 사용자 입장에서 애플리케이션이 제공하는 기능이 올바르게 동작하는지를 테스트
    - 일반적으로 테스트 전문 부서(QA 부서) 또는 외부 QA 업체가 담당.
    - API 툴이나 데이터베이스까지 연관되어 있어서 HTTP 통신도 해야 되고, 데이터베이스 연결도 해야 되는 등 연관된 대상이 많아 단위 테스트로 부르기 힘듬.
    
    2️⃣ 통합 테스트
    
    - 애플리케이션을 만든 개발자 또는 개발팀이 테스트의 주체
    - 클라이언트 측 툴 없이 개발자가 짜 놓은 테스트 코드를 실행시켜서 이루어지는 경우가 많음.
    
    3️⃣ 슬라이스 테스트
    
    - 애플리케이션을 특정 계층으로 쪼개어서 하는 테스트를 의미
    - Mock(가짜) 객체를 사용해서 계층별로 끊어서 테스트할 수 있기 때문에 어느 정도 테스트 범위를 좁히는 것이 가능
    - but, 단위테스트 보다는 큰 단위의 테스트이며, 애플리케이션의 일부만 테스트하기 때문에 부분 통합 테스트라고 부르기도 함.
    
    4️⃣ 단위테스트
    
    - 비즈니스 로직에서 사용하는 클래스들이 독립적으로 테스트하기 가장 좋은 대상이기 때문에 단위 테스트라 부르는 경우가 많음
    - **단위 테스트 코드는 메서드 단위로 대부분 작성**
    

### 단위테스트

1. 단위테스트를 해야하는 이유?
    - 코드가 의도한 대로 동작하는지 그 결과를 빠르게 확인가능
    - 문제의 원인을 찾아내는 것보다 상대적으로 더 적은 시간 안에 문제를 찾아낼 가능성 ↑
    - 버그가 발생한 기능의 테스트 케이스를 돌려보면서 문제가 발생한 원인을 단계적으로 찾아가기가 용이
2. **F.I.R.S.T 원칙**
    
    ✔️<span style="color:Blue">**Fast(빠르게)**</span>
    
    일반적으로 작성한 테스트 케이스는 빨라야 한다는 의미
    
    ✔️<span style="color:Blue">**Independent(독립적으로)**</span>
    
    test1 () → test2 () 를 통과했는데 반대로는 통과하지 않는 경우에는 독립적이지 않음, 영향을 끼치면 안됨<br/> ➡️ 롤백 기능 추가 또는 ~~등 독립적으로 통과할 수 있도록 해야 함. 
    
    ✔️<span style="color:Blue">**Repeatable(반복 가능하도록)**</span>
    
    어떤 환경에서도 반복해서 실행이 가능해야 된다는 의미
    
    ✔️<span style="color:Blue">**Self-validating(셀프 검증이 되도록)**</span>
    
    단위 테스트는 성공 또는 실패라는 자체 검증 결과를 보여주어야 한다는 의미
    
    ✔️**<span style="color:Blue">Timely(시기적절하게)</span>**
    
    단위 테스트는 테스트하려는 기능 구현을 하기 직전에 작성해야 한다
    
    많이 안짜봤거나 프로젝트에서 이걸 보통 안 지키는 경우가 많음 
    
    ⇒ 반례를 떠오르지 못하기 때문에 테스트를 통과하기 위한 코드를 작성하는 경우가 생김
    
    개발 끝나고 테스트 코드를 짜면 안됨 → 코드를 짜면서 테스트를 짜야 함. (커밋 내역으로 확인 가능하니 숨길 수 없음)
    
    **구현하고자 하는 기능을 단계적으로 조금씩 업그레이드하면서 그때그때 테스트 케이스 역시 단계적으로 업그레이드하는 방식이 더 낫다!!**
    
    <br/>

## JUnit 없이 단위테스트 적용

### 유틸리티 클래스

- 단위 테스트를 제일 쉽고 빠르게 적용할 수 있는 부분은 바로 헬퍼(helper) 클래스 또는 유틸리티(utility) 클래스
- Spring Framework에서 StringUtils, BeanUtils 같은 유틸리티 클래스를 지원
  
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3ca7c665-8533-41ad-b749-fbb0283b1f2d" width=500/>
    
    - static이 아닌이유는 하나임⇒  제네릭이라서 !, 타입 매개변수를 받아야 하기 때문에 , 아니라면 정적으로 사용 → 왜냐면 전역에서 사용하는 것이기 때문
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/0162c944-12ba-452c-9040-3b09798d9994" width=700/>
    
    - static 사용 시 에러 발생
    
    - 커피 주문 프로그램의 MemberService에 구현할 경우 DI를 받아서 사용해야 함
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c753b942-262d-4dfd-aa9b-a84746a9c3e0"  width=500/>
    
    - BeanUtils의 기능은 업데이트 할 때 원본이랑  비교 후  변경해주는 것
    - 3줄을(Optional) →   한줄로 사용 가능,코드를 줄일 수 있다는 것이 큰 장점. -코드의 중복을 줄여줌

### Given-When-Then 표현 스타일

<span style="color:OrangeRed">`given - when - then`</span>이라는 용어는 BDD(Behavior Driven Development)라는 테스트 방식에서 사용하는 용어

1. <span style="color:OrangeRed">given</span>
    
    - 테스트를 위한 준비 과정을 명시
    - 테스트에 필요한 전제 조건들이 포함
    - 테스트 대상에 전달되는 입력 값(테스트 데이터) 역시 Given에 포함
2. <span style="color:OrangeRed">when</span>
    
    - 테스트할 동작(대상)을 지정
    - 일반적으로 메서드 호출을 통해 테스트를 진행
3. <span style="color:OrangeRed">then</span>
    
    - 테스트의 결과를 검증하는 영역
    - 일반적으로 예상하는 값(**expected**)과 테스트 대상 메서드의 동작 수행 결과(**actual**) 값을 비교해서 기대한 대로 동작을 수행하는지 검증(**Assertion**)하는 코드들이 포함된다.
4. 예제
    - Test 생성
      
        StampCalculator가 구현되어 있는 클래스에서 Alt+Insert 누르면 Test를 생성할 수 있음 → 이렇게 하면  Test 폴더에 기존에 있던 클래스의 패키지 아래에 클래스를 생성시켜줌.
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c3d66a4a-003a-44a8-8b2a-8b846f90aefc"  width=400/>
        
    - 테스트 코드 작성
    
    ```java
    package com.codestates.helper;
    
    public class StampCalculatorTestWithoutJUnit {
        public static void main(String[] args) {
            calculateStampCountTest();
        }
    
        private static void calculateStampCountTest() {
            // given , parameter 준비
            int nowCount = 5;
            int earned = 3;
    
            // when, 검증해야 될 Test를 실행
            int actual = StampCalculator.calculateStampCount(nowCount, earned);
    
            int expected = 7;
    
            // then 검증 
            System.out.println(expected == actual);
        }
    	
        private static void calculateEarnedStampCountTest() {
            // given
            Order order = new Order();
            OrderCoffee orderCoffee1 = new OrderCoffee();
            orderCoffee1.setQuantity(3);
    
            OrderCoffee orderCoffee2 = new OrderCoffee();
            orderCoffee2.setQuantity(5);
    
            order.setOrderCoffees(List.of(orderCoffee1, orderCoffee2));
    
            int expected = orderCoffee1.getQuantity() + orderCoffee2.getQuantity();
    
            // when
            int actual = StampCalculator.calculateEarnedStampCount(order);
    
            // then
            System.out.println(expected == actual);
        }	
    }
    ```
    
    - **given**
      
        주문한 커피의 수량이 필요하기 때문에 Order와 OrderCoffee 객체를 직접 만들어서 테스트에 필요한 데이터를 생성
        
    - **when**
        - StampCalculator.calculateEarnedStampCount()에 given에서 생성한 테스트 데이터를 입력값으로 전달 →  StampCalculator.calculateEarnedStampCount() 메서드가 잘 동작하는지를 확인하기 위함.
    - **then**
      
        주문한 커피 수량만큼의 스탬프가 계산되는지를 **Assertion**
        

## JUnit으로 단위테스트 적용

1. JUnit이란?
    - Unit은 Java 언어로 만들어진 애플리케이션을 테스트하기 위한 오픈 소스 테스트 프레임워크
2. 테스트 케이스의 기본 구조
   
    ```java
    import org.junit.jupiter.api.Test;
    
    public class JunitDefaultStructure {
    		// (1)
        @Test
        public void test1() {
            // 테스트하고자 하는 대상에 대한 테스트 로직 작성
        }
    
    		// (2)
        @Test
        public void test2() {
            // 테스트하고자 하는 대상에 대한 테스트 로직 작성
        }
    
    		// (3)
        @Test
        public void test3() {
            // 테스트하고자 하는 대상에 대한 테스트 로직 작성
        }
    }
    
    ```
    

### **Assertion 메서드**

Assertion은  ‘**예상하는 결과 값이 참(true)이길 바라는 논리적인 표현**’

1. <span style="color:OrangeRed">**`assertNotNull()`**</span> : Null 여부 테스트
2. <span style="color:OrangeRed">**`assertThrows()`**</span> : 예외(Exception) 테스트
    
    - **assertThrows()**의 첫 번째 파라미터에는 발생이 기대되는 예외 클래스를 입력하고, 두 번째 파라미터인 람다 표현식에서는 테스트 대상 메서드를 호출하면 된다.
    
    ```java
    //...
    public class AssertionNotNullTest {
        @Test
        @DisplayName("AssertionNull() Test")
        public void assertNotNullTest(){
            //given,when
            String currencyName = CryptoCurrency.map.get("BTT");
            //then
           
            Assertions.assertNotNull(currencyName);
            //(1)
            Assertions.assertThrows(NullPointerException.class,()-> currencyName.toUpperCase());
            //(2)
            Assertions.assertThrows(Exception.class,()->currencyName.toUpperCase());
            //(3)
        }
    }
    ```
    
    - (1) Map에서는 없는 키를 가져올 때는 자동으로 null 이 들어오게 된다.
    - (2) assertTrows는 예외가 발생하는지 그리고 실제로 NPE가 발생하는지 봄 → fail이 나오면 예외가 발생하지 않은 것임
    - (3) Exception은 NPE의 상위 클래스이므로 동일하게 작동함. → **예외 클래스의 상속 관계를 이해한 상태에서 테스트 실행 결과를 예상해야 된다,**
    
3. <span style="color:OrangeRed">**`Assertions.assertTrue` :**</span>
4. <span style="color:OrangeRed">**`assertEquals()`:**</span> 기대하는 값과 실제 결과 값이 같은 지를 검증
   
    ```java
    package com.springboot.helper;
    
    import com.springboot.order.entity.Order;
    import com.springboot.order.entity.OrderCoffee;
    import org.junit.jupiter.api.Assertions;
    import org.junit.jupiter.api.Test;
    
    import java.util.List;
    
    import static org.junit.jupiter.api.Assertions.*;
    
    class StampCalculatorTest {
    
        @Test
        void calculateStampCount() {
            //given
            int nowCount = 5;
            int earned = 3;
            int expected = 7;
    
            //when
            int actual = StampCalculator.calculateStampCount(nowCount,earned);
    
            //then
            Assertions.assertTrue(expected==actual);
            //밑줄 에러 발생 하는 이유 ->  String일 때는 비교는 .equals로 해야 함.
            //Assertions.assertTrue("str".equals("str"))
            Assertions.assertEquals(expected, actual);
    				Assertions.assertEquals(new int[]{1,2}, new int[]{1,2});
    
        @Test
        void calculateEarnedStampCount() {
            //given
            Order order = new Order();
            OrderCoffee orderCoffee1 = new OrderCoffee();
            orderCoffee1.setQuantity(3);
            OrderCoffee orderCoffee2 = new OrderCoffee();
            orderCoffee2.setQuantity(5);
    
            order.setOrderCoffees(List.of(orderCoffee1,orderCoffee2));
            int expected = orderCoffee1.getQuantity()+orderCoffee2.getQuantity();
            //when
            int actual = StampCalculator.calculateEarnedStampCount(order);
    
            //then
            Assertions.assertEquals(expected, actual);
        }
    }
    ```
    
    - Assertions.assertTrue는 직관적이지 않음, 두 개의 값을 비교할 때는 무조건 assertEquals

### **Assumption을 이용한 조건부 테스트**

(Assertions.assertTrue랑 헷갈리지 않기)

<span style="color:OrangeRed">**assumeTrue()** </span> 메서드는 파라미터로 입력된 값이<u> **`true`**이면 나머지 아래 로직들을 실행</u>

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

public class AssumptionTest {
    @DisplayName("Assumption Test")
    @Test
    public void assumptionTest() {
        // (1)
        assumeTrue(System.getProperty("os.name").startsWith("Windows"));
//        assumeTrue(System.getProperty("os.name").startsWith("Linux")); // (2)
        System.out.println("execute?");
        assertTrue(processOnlyWindowsTask());
    }

    private boolean processOnlyWindowsTask() {
        return true;
    }
}

```

- PC의 운영체제(OS)가 윈도우(Windows)라면 `assumeTrue()` 메서드의 파라미터 값이 `true`가 될 것이므로 `assumeTrue()` 아래 나머지 로직들이 실행이 될 것이고,  PC 운영체제(OS)가 윈도우(Windows)가 아니라면 `assumeTrue()` 아래 나머지 로직들이 실행되지 않음!
- 아래 사진에 대한 코드는 윈도우에서  빨간박스 부분이 실행되지 않는다

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/eda94ab3-12eb-4a6d-ba17-917775ca72bb"  width=500/>

### Test케이스 실행 전 전처리

1. <span style="color:OrangeRed">**@BeforeEach**</span>
   
    테스트 케이스가 각각 실행될 때마다 테스트 케이스 실행 직전에 먼저 실행되어 초기화 작업 등을 진행
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/af597036-f880-41e1-a0fb-b80841d63fb2"  width=400/>
    
    - Test_1은 passed, Test_2는 Failed
    - Test case1에서 map에 **“XRP”를 추가했다 하더라도 추가한 “XRP”는 Test case2 실행 전에 init() 메서드가 다시 호출되면서 map이 초기화되기 때문에 초기화된 상태로 되돌아간다.**
2. <span style="color:OrangeRed">**@BeforeAll**</span>
   
    클래스 레벨에서 테스트 케이스를 한꺼번에 실행시키면 테스트 케이스가 실행되기 전에 딱 한 번만 초기화 작업을 할 수 있도록 해주는 애너테이션
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/19e217d1-7465-4059-bea0-e8494b171e7b"  width=400/>
    
    - @BeforeAll로 실행할 수 있는 메서드는 static 메서드이다, 상태가 항상 공유되어야 함.
3. <span style="color:OrangeRed">@AfterEach</span> - 하나 끝날때마다 실행
4. <span style="color:OrangeRed">@AfterAll</span> - 제일 마지막에 한번만 실행

