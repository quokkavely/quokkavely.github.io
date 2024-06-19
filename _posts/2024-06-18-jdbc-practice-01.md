---
layout : single
title : "JDBC-Practice service계층 구현 +Optional"
categories: Spring
tag : [Spring, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}



## Optional Class

[https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

### Optional 클래스의 주요 기능과 사용법

Optional 클래스는  NullPointerException을 방지하고, 명시적으로 값의 존재 여부를 처리할 수 있는 유용한 도구

`Optional` 클래스는 모든 참조타입은 사용 가능.

1. **Optional 객체 생성**
    - Optional.of(T value): value가 null이 아니어야 하며, null이면 NullPointerException이 발생
    - Optional.ofNullable(T value): value가 null일 수도 있고, null이 아니어도 됨
    - Optional.empty(): 빈 Optional 객체를 생성
    
    ```java
    Optional<String> nonEmptyOptional = Optional.of("Hello");
    Optional<String> nullableOptional = Optional.ofNullable(null);
    Optional<String> emptyOptional = Optional.empty();
    
    ```
    
2. **값 존재 여부 확인**
    - isPresent(): 값이 존재하면 true, 존재하지 않으면 false를 반환
    - ifPresent(Consumer<? super T> action): 값이 존재할 때만 지정된 동작을 수행
    
    ```java
    if (nonEmptyOptional.isPresent()) {
        System.out.println("Value is present");
    }
    
    nullableOptional.ifPresent(value -> System.out.println("Value: " + value));
    
    ```
    
3. **값 가져오기**
    - get(): 값이 존재하면 값을 반환하고, 값이 존재하지 않으면 NoSuchElementException 던짐
    - orElse(T other): 값이 존재하면 값을 반환하고, 존재하지 않으면 other를 반환
    - orElseGet(Supplier<? extends T> other): 값이 존재하지 않으면 Supplier를 호출하여 그 값을 반환
    - orElseThrow(Supplier<? extends X> exceptionSupplier): 값이 존재하지 않으면 지정된 예외를 던짐
    
    ```java
    String value1 = nonEmptyOptional.get();
    String value2 = nullableOptional.orElse("Default Value");
    String value3 = emptyOptional.orElseGet(() -> "Default Value");
    
    try {
        String value4 = emptyOptional.orElseThrow(() -> new IllegalArgumentException("Value not present"));
    } catch (IllegalArgumentException e) {
        System.out.println(e.getMessage());
    }
    
    ```
    
4. **값 변환**
    - map(Function<? super T, ? extends U> mapper): 값이 존재하면 주어진 함수를 적용한 결과를 담은 새로운 Optional을 반환
    - flatMap(Function<? super T, Optional<'U'> mapper): 값이 존재하면 주어진 함수를 적용하여 반환된 Optional을 평탄화하여 반환
    
    ```java
    
    Optional<String> upperOptional = nonEmptyOptional.map(String::toUpperCase);
    upperOptional.ifPresent(System.out::println); // "HELLO"
    
    Optional<Integer> lengthOptional = nonEmptyOptional.flatMap(value -> Optional.of(value.length()));
    lengthOptional.ifPresent(System.out::println); // 5
    ```
    
    - `? extends T`
        
        상한 와일드카드이며 매개변수화 된 유형이 ‘T’또는 해당 하위 클래스 중 하나임을 나타내는 데 사용됨, 이는 컬렉션에서 항목을 읽으려는 생산자에게 유용
        
    - `? super T`
        
        하한 경계의 와일드카드이며 컬렉션에 항목을 쓰려는 소비자에게 유용.
        

## 실습 (Service 구현하기)

Repository  활용하여 Service 구현하기.

### MemberService

1. 부족한 부분 + 꼭 익히기
    
    ```java
    public Member updateMember(Member member) {
    		// 수정하기 전에 필요한 작업 - 유효성 검증 필요. - 멤버가 있는지
    	 Member findMember=findVerifiedMember(member.getMemberId());
            if(member.getName()!= null){
                findMember.setName(member.getName());
            }
            if(member.getPhone()!=null){
                findMember.setPhone(member.getPhone());
            }
            return memberRepository.save(findMember);
        }
    ```
    
    - member를 수정하기 전에 필요한 것은 유효성 검증! → member가 존재하는지 검증할 것.
    - Member findMember=findVerifiedMember(member.getMemberId()); 여기서 검증 실패하면 updateMember를 호출한 곳으로 가게 됨. → 그럼 MemberController로 가게되고 globalExcpetionAdvise로 이동해서 예외처리
    - 만약 통과했다면 멤버가 있다는 것이고 Member에게는 수정할 이름 정보나 휴대폰 정보가 필요하게 되는데 하나만 변경하면 null이 들어가게 됨.
    - 나는 if문으로 정리했는데 Optional class를 활용하면 더 좋음.
        
        ```java
         Optional.ofNullable(member.getName())
                        .ifPresent(name -> findMember.setName(name));
                Optional.ofNullable(member.getPhone())
                        .ifPresent(phone -> findMember.setPhone(phone));
        ```
        
        ⚡멤버 변수 값이 null일 경우에는 Optional.of()가 아닌 Optional.ofNullable()을 이용해서 null 값을 허용할 수 있음.
        
        ⚡name 또는 phone 멤버 변수의 값이 null이 아니라면) ifPresent() 메서드 내의 코드가 실행이 되고, 수정할 값이 없다면 (name 또는 phone 멤버 변수의 값이 null이라면) 아무 동작도 하지 않음
        
        🔎 참고로 Spring Data JDBC에서는 @Id 애너테이션이 추가된 엔티티 클래스의 멤버 변수 값이
        
        - 0 또는 null이면 →  신규 데이터라고 판단하여 테이블에 **insert** 쿼리를 전송
        - 0 또는 null이 아니라면  → 이미 테이블에 존재하는 데이터라고 판단하여 테이블에 **update** 쿼리를 전송

### CoffeeService

1. 헤맸던 구간
    
    ```java
    public Coffee updateCoffee(Coffee coffee) {
        Coffee findCoffee = verifyExistCoffee(coffee.getCoffeeId());
    
        Optional.ofNullable(coffee.getCoffeeCode())
                .isPresent(code -> findCoffee.setCoffeeCode(code));
        Optional.ofNullable(coffee.getKorName())
                .isPresent(name -> findCoffee.setKorName(name));
    
        return coffeeRepository.save(findCoffee);
    }
    
    ```
    
    - 이 부분에 code랑 name이 자꾸 빨간줄 떠서 왜그런가 했는데 isPresent가 아니라 ifPresent 를 사용해야하는데 잘못 사용함,

1. 가장 어려웠던 부분
    
    ```java
        // 주문에 해당하는 커피 정보 조회
        public List<Coffee> findOrderedCoffees(Order order) {
    //        Set<OrderCoffee>orderCoffees = order.getOrderCoffees();
    //        List<Coffee> coffees=new ArrayList<>();
    //        for(OrderCoffee ordercoffee: orderCoffees){
    //            Coffee findCoffee = verifyExistCoffee(ordercoffee.getCoffeeId());
    //            coffees.add(findCoffee);
    //        }
            return order.getOrderCoffees()
                    .stream()
                    .map(orderCoffee -> verifyExistCoffee(orderCoffee.getCoffeeId()))
                    .collect(Collectors.toList());
    
        }
    ```
    
    - 처음에 for문으로 한 번 구현하고 그 다음 Stream으로 구현해봄
    - 한 번에 Stream은 아직 불가능해서 for문 먼저 만들어봄,, 사실 for문도 조금 헤맸기 때문에,,
    - 그래도 한 번 구현하니 Stream은 금방 구현함!!!!

### OrderService

1. **보완할 사항** 
    
    ```java
       public Order createOrder(Order order) {
            //멤버가 존재하는지
            memberService.findMember(order.getMemberId());
            //커피가 존재하는지
            //List<Coffee>orderCoffees=coffeeService.findOrderedCoffees(order);
            for(Coffee coffee: coffeeService.findOrderedCoffees(order)){
                coffeeService.verifyExistCoffee(coffee.getCoffeeId());
            }
          return orderRepository.save(order);
        }
    ```
    
    - 설명듣다가 orderCoffee가 jointable이니까 orderCoffee는 Coffee와 Order를 다 참조할 수 있음. → order가 Set<OrderCoffee>ordercoffees를 멤버 변수로 가지고 있음.
        
        ```java
         for(OrderCoffee orderCoffee:order.getOrderCoffees()){
                    coffeeService.verifyExistCoffee(orderCoffee.getCoffeeId());
             }
        ```
        
    - Stream
        
        ```java
        order.getOrderCoffees()
        										.stream()
        		                .forEach(orderCoffee -> 
        		                coffeeService.verifyExistCoffee(orderCoffee.getCoffeeId()));
        ```
        
    
2. **틀린 부분**
    
    ```java
        public void cancelOrder(long orderId) {
            //유효한 주문인지 찾고 삭제 +주문상태확인
           Order findOrder = verifiedOrder(orderId);
           int orderStatus = findOrder.getOrderStatus().getStepNumber();
           if(orderStatus>=2){
               throw new BusinessLogicException(ExceptionCode.CANNOT_CHANGE_ORDER);
           }
           findOrder.setOrderStatus(Order.OrderStatus.ORDER_CANCLE);
           orderRepository.save(findOrder);//(!)
        }
    ```
    
    - (!) 여기 부분을 orderRepository.delete(findOrder); 로 하면 안됨. → 실무에서는 어떠한 데이터도 지우지 않음.

### Stream에서 foreach와 map의 차이

forEach는 최종연산자이고 반환값이 void

map은 중간연산자임.

### Comment

혼자서 코드 작성할 때는 아 이렇게 하는 건가 하면서 워낙 인텔리제이가 잘되있으니까

. 치면 다 나오니까 연결지어서 했는데 막상 내가 친 코드를 남에게 설명하려고 하니 말이 안나와서 당황했다.. 그래서 옆에 있는 사람에게 이 코드를 왜 이렇게 쳐야하는지 이걸 왜불러와야하는지 설명까지 잘해야 내가 그 코드를 완벽하게 이해한 것이 아닐까?