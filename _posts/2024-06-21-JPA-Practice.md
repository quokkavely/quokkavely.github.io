---
layout : single
title : "[JPA] 연관관계 매핑, 기능 추가 실습"
categories: Spring
tag : [Spring, 실습]
author_profile: true
---

# JPA 실습

필수 구현 조건

1. 연관관계 매핑
    - Order 클래스와 Coffee 클래스의 연관 관계 매핑 구현
    - Member 클래스와 Order 클래스의 연관 관계 매핑 구현
    - Member 클래스와 Stamp 클래스의 연관 관계 매핑 구현
2. 주문 기능 추가 구현
    - **주문한 커피의 수량만큼 회원의 스탬프 숫자를 증가**
    - get 요청시 페이지 구현
    - dto생성 및 mapper, service 클래스 새로 구현

## DTO 및 mapper 클래스 구현 - 가장 오래 걸림

아무래도 제일 오래 걸린 이유는 Stream에 능숙하지 않아서 이지 않을 까 생각이 든다…

Order클래스가 member, orderCoffee랑 연결되어 있고 주문 기능이 메인이다보니  MapStruct 기능을 사용하기에 복잡한 로직이라서 커스텀으로 구현해야 했다. 

1. Page 구현을 위해서 MultiResponseDto orderToPageResponseDto(Page<Order>orderPage)를 구현하려고 했는데 **pageInfo 의 타입이 안 맞다고 에러 발생**
    - 아마도 타입이 안맞아서 그런것 같음
      
        ```java
        MultiResponseDto(java.util.List, org.springframework.data.domain.Page)' 
        in 'com.springboot.response.MultiResponseDto' cannot be applied 
        to '(java.util.List<com.springboot.order.entity.Order>, com.springboot.response.PageInfo)’
        ```
        
        <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/a03be664-8dc7-458d-8b21-038842bee222" width=300/>
        
    - MultiResposneDto는 기본 생성자로 *List* '<T>' data, *Page* page 2개의 파라미터가 존재하는데 Page타입과 Pageinfo 타입이 달라서 발생하는 문제인 것 같다.
    - 그래서 강제로 (Page)pageInfo하면 에러가 사라진다.
    - 근데  MultiResponseDto는 전역에서 사용하는 것이므로 자동으로 생성자에서 PageInfo를 생성자로 만들어 활용하기 좋게 만들어  굳이 mapper에서 Method를 만드는 것 보다 data에 해당하는 List '<Order>' orders만 dto로 변환해주는 메서드를 만드는 것이 더 좋을 것 같아서 `*default List*<OrderResponseDto> ordersToOrderResponseDtos(*List*<Order> orders)` 를 구현하고 기존 것은 지웠다
      
        ```java
        //controller
        
        @GetMapping
        public ResponseEntity getOrders(@Positive @RequestParam int page,
                                        @Positive @RequestParam int size){
                Page<Order> pageOrders = orderService.findOrders(page - 1, size);
                List<Order> orders = pageOrders.getContent();
        
                
                return new ResponseEntity<>(
                        new MultiResponseDto<>(
                        mapper.ordersToOrderResponseDtos(orders),pageOrders)
                        ,HttpStatus.OK);
            }
        ```
        
    
2. **OrderPostDto → Order**
    1. 처음 작성한 코드
       
        ```java
         default Order orderPostDtoToOrder(OrderPostDto orderPostDto) {
                  orderPostDto.getMember().getOrders()
                        .stream()
                        .forEach(order-> {
                            new Order(
                                order.setOrderId(),
                                order.getOrderStatus(),
                                order.getCreatedAt(),
                                order.getModifiedAt(),
                                order.getMember(),
                                order.getOrderCoffees());
                        });
            }
        ```
        
        - orderPostDto에서 Order를 가져오면 되니까 이렇게 구현했는데 이게 아니라 주문 요청이 들어오면 새로운 주문을 만들어 줘야 한다.
        - Order가 가지고 있는 멤버변수로  orderId는 자동으로 생성되고, orderStatus도 자동으로 주문요청으로 초기화 값 생김
        - 그리고 createdAt, modifiedAt도 newOrder가 생성되는 시간으로 초기화되기 때문에 내가 넣을 필요는 없음.
        - 그럼 최종적으로  Order 변환해줄 값은 **Member와 List<OrderCoffee>** 뿐이다.
    2. 다시 작성한 코드
       
        ```java
        default Order orderPostDtoToOrder(OrderPostDto orderPostDto) {
                    Order order = new Order();
                    Member member = orderPostDto.getMember();
                    order.setMember(member);
                    List<OrderCoffee>orderCoffees=orderPostDto.getOrderCoffees()
                                    .stream()
                                    .map(orderCoffeeDto -> {
                                        OrderCoffee orderCoffee = new OrderCoffee();
                                        orderCoffee.setQuantity(orderCoffeeDto.getQuantity());
                                        Coffee coffee = new Coffee();
                                        coffee.setCoffeeId(orderCoffeeDto.getCoffeeId());
                                        orderCoffee.setCoffee(coffee);
                                        orderCoffee.setOrder(order);
                                        return orderCoffee;
                                    }).collect(Collectors.toList());
        
                    order.setOrderCoffees(orderCoffees);
                return order;
            }
        ```
        
        - 처음에 주문이 시작되려면 회원이 주문을 할 것이고 커피와 수량이 포함되어 있는 List<OrderCofee> 가 존재한다.  ⇒ 이것들은 OrderPostDto가 가지고 있는 멤버 변수이고, 여기서 member정보와 List를 order에 넣어주면 됨!
        - 새로운 주문에 해당하는 Order order = new Order()을 생성한 후
        - orderPostDto에서 getMember로  회원을 가져와서  order에 setter로 넣어줌
        - 그다음 List<OrderCoffeeDto> → List<orderCoffee>로 Stream을 이용하여 변환 후 order에 setter로 넣어준 후 order 리턴.하면 된다.
    3. 내가 Stream 할 때 가장 헷갈리는 부분은 .map으로 변환해서 { } 안에서 return 하거나 , forEach는 return이 안되니까 map을 사용해야 되는데 map은 최종 연산자로 어떤 걸 사용해야 할 지 모르겠다.
    
3. order → List<OrderCoffeeResponseDto>
   
    ```java
    default List<OrderCoffeeResponseDto> orderToOrderCoffeeResponseDtos(Order order) {
    
            List<OrderCoffeeResponseDto> result = order.getOrderCoffees()
                    .stream()
                    .map(orderCoffee ->
            {
                return new OrderCoffeeResponseDto(
                        orderCoffee.getCoffee().getCoffeeId(),
                        orderCoffee.getCoffee().getKorName(),
                        orderCoffee.getCoffee().getEngName(),
                        orderCoffee.getCoffee().getPrice(),
                        orderCoffee.getQuantity());
            }).collect(Collectors.toList());
            return result;
        } 
    ```
    
    - return 안 써서 계속 빨간 줄인채로 나두다가 나중에 아 맞다….하고 해결했던 부분, { } 있을때는 return문을 꼭 넣어줘야 함. (람다식에서 return 문이 있으면 중괄호를 생략할 수 없음.)

## POST 요청 에러

mapper랑 service 그리고 controller 까지 완료하고 stamp 구현만 안 한 상태에서 Member와 Coffee 모두 등록 후 order가 잘 등록되는지 확인하려고 postman에서 post 요청했는데 400 에러 발생

1. 주문요청
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/856c5df4-75c7-491f-ae67-e0747c19268e" width=300/>
    
2. H2 확인
   
    Member와 Coffee는 모두 잘 들어와 있고 , Order는 없음
    
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/36ca045b-8327-4b7d-a4fe-b6b193ff29da" width=300/>
    
1. Body 입력 실수
   
    애초에 Postman에서 Required request body is missing이니까 body입력을 잘못했나..생각했는데 커피를 여러 개 등록하고 Order에서 Post 요청하다 보니 coffeeCode에 coffeeId 입력함…!  
    
2. 해결 → 다시 입력하니까 잘 들어옴!
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d918ff51-6218-4b2e-8dcb-cd957a40e0ee" width=300/>
    
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/56321960-33a5-453d-894a-c151f198e56d" width=300/>

## Stamp 구현

### 관계 설정 에러 - 일대일 매핑

Stamp와 Member 간의 관계는 1:1이고 Member가 Stamp를 가지고 있어야 주문요청 때  Stamp 증가 기능을 활용할 수 있으니 Member에서 필드에 Stamp를 가지게 함.

1. 영속성 전이 추가
   
    ```java
    Caused by: java.lang.IllegalStateException: 
    org.hibernate.TransientPropertyValueException: 
    object references an unsaved transient instance 
    - save the transient instance before flushing : 
    com.springboot.member.entity.Member.stamp -> com.springboot.member.entity.Stamp
    ```
    
    transient가 계속 나오길래 stamp는  member가 있을 때만 존재하는 값이니 member가 생길 때 1차 캐시에 같이 저장되게끔 추가 함.
    
2. 일대일 → 양쪽에서 참조 에러 발생.
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/58548e8d-1970-40a5-99ba-51c0b9b9908c" width=300/>
    
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/100ef8f1-f3f8-4179-9a4b-3705a04d33c6" width=300/>
    
3.  Stamp 양쪽을 참조하게 관계를 설정하고 Stamp 갯수가 증가되면 Member에서도 Stamp가 추가되도록 구현했는데 생각해보니 Stamp가 새로 생성되는 것이 아니고 갯수만 증가하는 것이라서 addStamp기능을 제외하였음
4. 일대일 매핑할 때 양쪽에서도 서로 참조하는게 가능한 걸로 알고 있는데 아래와 같이 에러 발생함.
   
    ```java
    Caused by: org.hibernate.AnnotationException
    : Unknown mappedBy in: com.springboot.member.entity.Stamp.members, 
    referenced property unknown: java.util.List.stamp
    ```
    
5. 정확한 원인은 모르겠지만,,,생각해보니 stamp가 member를 가질 이유는 딱히 없는 것 같아서 stamp는 member를 참조하지 않고 member만 stamp 참조하게끔 구현하였음. 

### OrderService에서 POST 요청 시 Stamp 추가.

Stamp는 내가 뭘해도 자꾸 null이 들어옴,,,

<img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/7771320b-59df-4d71-8a48-a7de2264c6e2" width=300/>

1. Stamp 매핑 에러 
   
    커피 주문이 발생되면 Stamp가 들어와야하는데 안들어와서 ResponseDto에 아직 Stamp가 없어서 생성함
    
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/1a45e276-764e-4345-a650-68a7635fa8ae" width=300/>
    
    이에 맞게 mapper 도 함께 수정
    
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3b6112ef-5502-4687-bdee-53fca05a91db" width=300/>
    
2. 주문 시 마다 Stamp가 새로 생성되는 문제 발생
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5f48f3c5-2785-4766-b63e-3743743b2a01" width=300/>
    
    똑같은 member가 주문했는데 stamp가 2개 발생 됨.
    
    아무래도 stamp생성시 Stamp stamp = new Stamp(), 로 생성하는 것이 문제 인 것 같음
    
    member가 가지고 있는 스탬프를 가져와야 할듯.
    
3. NPE 발생
   
    member가 처음 주문이면 member.getStamp를 해도 Null이니 가져올 수 가 없기 때문에 NPE 발생하기 때문에 if문 추가 하였음, 혹시 Optional로 사용해볼까 했는데 더 복잡해지는 것 같아서 if문 사용해서 
    
    만약 null이면 new Stamp로 하고, null이 아니면 member.getStamp한 후 수량을 set으로 변경함.
    
    ```java
    Stamp stamp;
           if(member.getStamp()!=null){
               stamp= member.getStamp();
           }else{
               stamp=new Stamp();
           }
           stamp.setCoffeeStamp(quantity);
           member.setStamp(stamp);
    ```
    

## 필수구현은 완료!

구현해야 할 필수 조건은 다 완성 함.

### 연관관계 매핑 및 DB 구현

1. Member
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6fac1e7d-d89b-468e-bd53-b67ae1f8473b" width=300/>
    
2. Order
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b6b79ab7-b4b8-45b1-9897-eb56956e48c0" width=300/>
    
3. Coffee
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/45eb6caf-f71d-4b5b-98f1-e7a5c5986bc7" width=300/>
    
4. OrderCoffee
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c92525af-9701-4d4f-bbf3-6d436cb13349" width=300/>
    
    

### Paginaiton 구현

1. getOrder - member Id가 1인 회원의 주문목록 조회.
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c92525af-9701-4d4f-bbf3-6d436cb13349" width=300/>
    
2. getOrders
   
    <img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/58adff84-5de9-444f-8868-8b405026aaac" width=300/>

### Stamp 구현

<img src ="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/1ef9b084-5bec-42c8-b5b6-ae808087de3f" width=300/>

<br>

<br>

## 개선포인트 확인 후 개선.

### Stamp 생성 후 적립시 시간 변경 안됨.

1. Stamp 추가 등록시 modifiedAt에서 시간 수정안됨. 

   ⇒  stamp 갯수는 변경되는데 CREATED_At의 시간과 LAST_MODIFIED_AT의 시간과 동일함.

   <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/0be68bed-7d2a-4c9e-88cf-9997cea535d9" width=400/>

   CreateOrder에서 Stamp 갯수 적립하고 있으니 여기서 수정하면 된다.

2. OrderService의 CreateOrder에서 변경 완료

   ```sql
    Stamp stamp;
          if(member.getStamp()!=null){
              stamp= member.getStamp();
          }else{
              stamp=new Stamp();
          }
          stamp.setCoffeeStamp(quantity);
           stamp.setModifiedAt(LocalDateTime.now()); //추가
          member.setStamp(stamp);
   ```

### CreateOrder에 너무 많은 기능 →  메서드로 분리하기

1. CreateOrder에서 Stamp를 초기화 하지 않고 Member에서 초기화 하기.

   ```java
   Stamp stamp;
          if(member.getStamp()!=null){
              stamp= member.getStamp();
          }else{
              stamp=new Stamp();
          }
          stamp.setCoffeeStamp(quantity);
          member.setStamp(stamp);
   ```

   변경후

2. 기능분리 1 → Member와 Coffee를 검증하는 기능

   ```java
   private Order preOrderValidation(Order order){
           // 회원이 존재하는지 확인
           memberService.findVerifiedMember(order.getMember().getMemberId());
           //커피가 존재하는지 확인
           order.getOrderCoffees()
                   .stream()
                   .forEach(orderCoffee -> {
                       coffeeService.findVerifiedCoffee(orderCoffee.getCoffee().getCoffeeId());
                   });
           return order;
       }
   ```

3. 기능분리2 → Stamp 적립 기능

   ```java
     private void saveUpStamp(Order order){
   			  Member member = order.getMember()
           Stamp stamp = member.getStamp();
           int quantity = order.getOrderCoffees()
                   .stream()
                   .mapToInt(orderCoffee -> orderCoffee.getQuantity()).sum();
           stamp.setCoffeeStamp(quantity);
           stamp.setModifiedAt(LocalDateTime.now());
           order.getMember().setStamp(stamp);
       }
   ```

   - 주문했는데 Stamp 갯수가 안 들어옴

     <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e92c6448-daed-47fb-beb6-448e13086a6e" width=400/>

   - Stamp 갯수와는 상관없을 것 같긴한데 Persist보다는 All이 맞는 것 같아서 변경

     ```java
     //Member 클래스
     
     @OneToOne(cascade = CascadeType.ALL)
         @JoinColumn(name = "STAMP_ID")
         private Stamp stamp = new Stamp();
     ```

   - 디버깅 해봄

     <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/ee6ea134-630f-473f-92b0-7f6c7bd515b3" width=400/>

     - stampCount는 잘 들어오는 것 같은데
     - 원래 잘 되던 메서드가 분리했다고 안되는 것이 이상해서 아마 검증하던 메서드가 없어서 발생하던 일 같았다.
     - member를 memberId로 불러와서  getStamp를 해야되는게 맞는 것 같아서 변경했더니 해결되었다.

   - OrderService 클래스에서 스탬프 적립하는 메서드 새로 수정

     <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e40c4004-af5d-4d89-8d4f-469f2e4aca92" width=400/>

     그리고 하나 놓친것이 또 있는데 여기에 추가로 Stamp를 적립했으면 DB에 명시적으로 저장하는 것이 맞다고 한다. →  표에는 잘들어오더라도 여기선 되다가 문제가 발생할 수 있으니 명시적으로 꼭 업데이트하기!

     ```java
       //명시적으로 꼭 업데이트 하기.
             memberService.updateMember(findMember);
     ```





## Comment

stamp는 기존에 없던 기능이라 오래 걸릴 줄 알았는데 (물론 오래걸리긴 했다만은….)

dto 생성하고 mapper 구현하는 시간이  stamp 구현하는 시간보다 더 오래 걸렸다.

구현할게 생각보다 많아서 당황스러웠고,,,예전에 자바로 텍스트프로그램 하는 것보다 오래걸리는 듯… 이걸 다시 백지부터 해보고 싶은데 할 수 있으려나 모르겠다.. 

정말 어려운 것은 실제로 코드로 구현해내는 것 보다 이 프로그램에 대해 다양한 관계와 기능 그리고 로직을 생각해야해서 더 어렵고 복잡한 것 같다