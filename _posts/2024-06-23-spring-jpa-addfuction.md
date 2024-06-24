---
layout : single
title : "[JPA] 영속성 전이,연관관계 매핑(일관성)"
categories: Spring
tag : [Spring, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## 영속성 전이

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용
- 커피주문시스템에서 회원과 스탬프(포인트)의 관계를 생각해보면 회원이 있어야 stamp가 발생되고 회원이 없으면 stamp는 사라지게 됨.
- 영속성 전이를 해주지 않으면 stamp의 경우 1차 캐시에 저장되지 않아 에러 발생할 수도 있음.
- Cascade 속성을 넣어주면 됨. @OneToMany(mappedBy = "order", cascade = CascadeType.PERSIST)

### CascadeType 종류

- **ALL : 모두 적용**
- **PERSIST : 영속화 될 때 함께 영속화 됨.**
- **REMOVE : 삭제될 때 함께 삭제**
- MERGE : 병합될 때 함께 병합
- REFRESH :새로고침될 때 함께 새로고침 됨
- DETACH : DETACH

### 예제

```java
 @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "STAMP_ID")
    private Stamp stamp = new Stamp();
```

## 양방향 연관관계 - 일관성유지 필요

- 만약에 새로운 주문이 발생하면 다대일 양방향에서는 서로 참조하고 있으니 서로를 update 해주어야 함.
- 양쪽에 add 메서드 생성후 commit 하면 StackOverFlow 발생 - 서로가 서로를 호출하기 때문 ⇒ 종료시점 필요

### Order 와 Member

1. Order와 Member = N : 1 
    
    Order에 @ManyToOne, Member 는 OneToMany 로 다대일 양방향으로 구현.
    
    ```java
        
        //Member 클래스
        @OneToMany(mappedBy = "member")
        private List<Order> orders = new ArrayList<>();
       
        public void setOrder(Order order){
            orders.add(order);
            if(order.getMember()!=this){
                order.setMember(this);
            }
        }
        
        //order 클래스
        @ManyToOne
        @JoinColumn(name = "MEMBER_ID")
        private Member member;
    
        public void addMember(Member member) {
            this.member = member;
            if(!member.getOrders().contains(this)){
                member.getOrders().add(this);
            }
        }
    ```
    

1. 일관성 유지 위해 Setter와 비슷한 기능을 하는 Method 추가
    - 양방향 연관관계에서는 서로를 참조하고 있기 때문에 Member에 새로운 객체가 생성되었다면 Order도 그 객체를 가져야하고 반대의 경우도 동일
    - Member나 Order에 새로운 객체가 생성될 경우 이미 있다면 추가하지 않고 없으면 추가하도록 조건문으로 설정
    
    ![Untitled](%5BJPA%5D%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC,%20%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%8B%E1%85%B5%20cb3293b2c4ce43529aa6702ddec5f6bb/Untitled%2017.png)
    
2. 2번 처럼 종료시점을 정하지 않으면 무한루프에 빠지게 됨 - **StackOverflow 발생**
    
    ![Untitled](%5BJPA%5D%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC,%20%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%8B%E1%85%B5%20cb3293b2c4ce43529aa6702ddec5f6bb/Untitled%2018.png)
    

### Order 와 OrderCoffee

1. OrderCoffee는 Join table로 , Order - OrderCoffee 관계는 1:N
    
    ```java
    //OrderCoffee 클래스
       @ManyToOne
       @JoinColumn(name="ORDER_ID")
       private Order order;
       public void setOrder(Order order){
            this.order=order;
            if(this.order.getOrderCoffeeList().contains(this)){
                if(!order.getOrderCoffeeList().add(this)){
    
                }
            }
        }
        
    //Order 클래스
    
        @OneToMany(mappedBy = "order", cascade = CascadeType.PERSIST)
        private List<OrderCoffee> orderCoffees=new ArrayList<>();
    
        public void setOrderCoffee(OrderCoffee orderCoffee){
            orderCoffees.add(orderCoffee);
            if(orderCoffee.getOrder()!=this){
                orderCoffee.setOrder(this);
            }
        }
    ```
    
2. 마찬가지로 다대일 양방향이라서 일관성 유지를 위해, Setter 기능과 비슷한 역할을 하는 메서드를 추가하였음.


## Comment

중요한 것 같은데 뭔가 아리쏭해서 다시한번 정리해봄.
일관성유지를 위해 method 를 추가하는 것에 대해서는 따로 뭐라고 부를 명칭이 없어서<br>
검색해봐도 일관성까지만 나오고 별 다른 설명은 없는듯...!
여기가 조금 헷갈리니 다시 봐야지