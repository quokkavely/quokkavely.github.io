---
layout : single
title : "[JAVA] final-Program-배달텍스트프로그램"
categories: JAVA-Practice
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## 배달텍스트프로그램_V2
### 1번째 시도
아주 Nullpoint Exception 지옥이다.

1. **안되는 부분 -1**
    1. 빈배열 찾고 해당 인덱스 반환하는 메서드 먼저 사용할지, 아니면 등록된 shop여부를 찾고 빈배열을 찾을지 우선순위를 구해야하는데
        
        ```java
        int shopIdx =isValidShop(shops,SHOP_MAX,shopName);
            int currentIdx;
            //가게가 있다면
              if(shopIdx!=-1){
                  shops[shopIdx]=new Shop(shopName);
                  shop.addFood(signatureMenu,menuPrice);
                  System.out.printf("[안내] %s에 음식(%s, %d) 추가되었습니다.", shopName, signatureMenu, menuPrice);
                  System.out.println("[시스템] 가게 등록이 정상처리 되었습니다.");
              }else{ //가게가 없다면
                    currentIdx=isValidIndex(shops);
                    if(currentIdx!=-1){
                      //가게 배열이 비었다면 등록
                      shops[currentIdx]=new Shop(shopName);
                      shop.addFood(signatureMenu,menuPrice);
                        System.out.printf("[안내] %s에 음식(%s, %d) 추가되었습니다.", shopName, signatureMenu, menuPrice);
                        System.out.println("[시스템] 가게 등록이 정상처리 되었습니다.");
                    }else{
                      System.out.println("등록할 수 있는 가맹점 수를 초과하엿습니다.");
        
        ```
        
        1. 가게 등록여부를 확인하고 배열이 비어있는지 여부를 확인하려하는데 NullPointerException 오류가 남... ⇒  `because "shop[i]" is null`
        2. 그래서 왜 그럴까 고민했더니...동일 shop인지 검증할 때 사용하는 메서드에는
        3. shop배열과 사용자가 입력한 shop이름만 일치하는지 일치한다면 해당 배열의 인덱스를 반환하기로 되어있는데
        4. 동일shop인지 검증할때 빈배열이면 null밖에 존재하지 않으니 null도 함께 검증해야한다
        5. 그래서 결국엔 null인지 먼저 검증 후 shop이 일치하는지 여부를 찾거나 이미 등록된 shop인지 검증하는 메서드에서 null값을 걸러내야하는데, 그것보단 전자가 나을 것 같다.!
        
        <img src="https://github.com/user-attachments/assets/ab12612c-b015-4eb1-9943-04707437f363" width=500 />
        
        

1. **안되는 부분 -2 /1번과 내용 비슷**
    1. isValidShop에서 for문 돌때 shop.length 사용 안하고  길이 따로 넣는 이유
        
        <img src="https://github.com/user-attachments/assets/56b24131-4933-4dd9-adb0-66ed4a584703" width=500/>
   


    2. 등록된 가게가 있는지 찾을때 사용하는 메서드인데 등록된 가게만큼만 돌게 해야된다.
    3. 왜냐면 빈배열일경우 null값이 존재하기때문에 nullpoint exception이 발생하기 때문
    <br/>
    4. 그래서 빈배열을 다 “”빈 문자열로 초기화 하거나 , null이 존재하지 않게 배열의 길이를 따로 구해서
    <br/>

    5. 매개인자로 넣어줘야함.
        <img src="https://github.com/user-attachments/assets/8520e089-f0b2-4102-a89c-f8bc2bd208a1" width=300/>
    
    <br/>

    6. 여기서는 빈배열이 존재하는지 만약 존재한다면 몇번째 인덱스에 존재하는지 찾는 메서드를 이용하여 maxlength를 구한 후
    7. 매개인자로 넣어주고 있다.

1. **안되는 부분 -3**
    1. **`상점이 있는데 new Shop으로 한것. >>  new Shop이 아니라 기존 shop에 addFood해야함.`**
        
        ```java
        int shopIdx = isValidIndex(shops);
        int currentIdx;
        // 빈 배열이 있다면
        if (shopIdx != -1) {
            // 배열 내 상점이 존재하는지?
            currentIdx = isValidShop(shops, shops.length, shopName);
            if (currentIdx != -1) {
                **shops[currentIdx] = new Shop(shopName);**
                if (shops[currentIdx].addFood(signatureMenu, menuPrice)) {
                    System.out.printf("[안내] %s에 음식(%s, %d) 추가되었습니다.", shopName, signatureMenu, menuPrice);
                    System.out.println("[시스템] 가게 등록이 정상처리 되었습니다.");
                }
            } else { // 배열 내 상점이 없다면 빈 배열에 상점 저장
                shops[shopIdx] = new Shop(shopName);
                if (shops[shopIdx].addFood(signatureMenu, menuPrice)) {
                    System.out.printf("[안내] %s에 음식(%s, %d) 추가되었습니다.", shopName, signatureMenu, menuPrice);
                    System.out.println("[시스템] 가게 등록이 정상처리 되었습니다.");
                }
            }
        } else { // 빈 배열이 없다면
            currentIdx = isValidShop(shops, shops.length, shopName);
            if (currentIdx != -1) {
                // 상점이 존재
                **shops[currentIdx] = new Shop(shopName);**
                if (shops[currentIdx].addFood(signatureMenu, menuPrice)) {
                    System.out.printf("[안내] %s에 음식(%s, %d) 추가되었습니다.", shopName, signatureMenu, menuPrice);
                    System.out.println("[시스템] 가게 등록이 정상처리 되었습니다.");
                }
            } else { // 상점이 없음
                System.out.println("등록할 수 있는 가맹점 수를 초과하였습니다.");
            }
        }
        ```
        
    2. **`shop에서 menu,price등록하는 메서드를 boolean type으로 만들었을 때 메서드에서 true, false만 반환하고 메서드 내에서 출력해야하는 건 안됨,`** 
        1. 그래서 if절에서 메서드를 추가하면 안되고 따로 메서드 실행 해야함.
            
            <img src="https://github.com/user-attachments/assets/be82ce00-4710-4388-8c95-c6e02bf371ab" width=500 />
            
            1. 그래서 addFood메서드를 활용해서
            2. addFood가 성공했을때만  출력문이 발생하게끔함.
            3. addFood가 성공하면 isAddFood는 true를 유지하고
            4. 실패하면 false로 바뀐다.
    3. addfood 메서드에서도 nullpoint Exception 발생했는데 메뉴나 price 에서 빈 배열의 경우 null값이 들어가기 때문이다, 그래서 초기화할때 빈 배열은 모두 “”빈문자열로 변경해줘야함.
        
        <img src="https://github.com/user-attachments/assets/4706c09e-6cfb-431c-be2f-ce15dd9d1777" width = 500 />
        
        <img src="https://github.com/user-attachments/assets/392ff127-25c6-487f-997f-d3d9f207b4d3" width = 500/>
        
    

### 2번째 시도

1. 이미 배열내에 존재하면
    1. registerSuccess = shops[shopValidIdx].addFood(menuName, menuPrice);
2. 배열 내에 존재하지 않을 때 -신규등록
    1. Shop shop = new Shop(shopName);
                shops[shopEmptyIdx] = shop;
3. 1번 음식점+메뉴등록시
    1. NPE 발생
        
        <img src="https://github.com/user-attachments/assets/1da0084c-74e7-46eb-b2de-3e2388b1772e" width=400/>
        
        1. 이유-1 메인에서 생성자에서 초기화 안함(1)
            
            기껏 initValues 메서드 만들어 놓고 생성자에서 초기화 안했음….
            
            <img src="https://github.com/user-attachments/assets/323d2f70-9431-48e4-baec-7778ec784cad" width=400/>
            
        2. 이유-2 Shop 클래스 생성자에서 초기화 안함(2)
            
            기껏 initalMenuAndPrice 메서드로 초기화 만들어 놓고 생성자에서 초기화 안했음….
            
            <img src="https://github.com/user-attachments/assets/824884d1-7466-40b2-9f13-2efaa3ed060a" width = 400/>
            
4. 리뷰등록 할 때 1-5점만 입력하게 하고 싶을 때
    1. 메서드 하나 더 만들어서 등록
        
        <img src="https://github.com/user-attachments/assets/66c96acb-4c15-4011-8b9a-11065a8cae53" width=500 />
        
    

## 배달텍스트프로그램_V3

        
1. 안되는 부분
    1. 처음 등록된 shop(shops의 index=0)을 제외한 새로운 가게의 첫번째 메뉴는 등록안하고, 두번째 등록부터만 등록됨
        - 출력예시
            
            
            ```java
            [시스템] 무엇을 도와드릴까요?
            >>>1
            aaa
            {ccc=10000, bbb=10000}
            
            [안내] 반갑습니다. 가맹주님!
            [안내] 음식점 상호는 무엇인가요?
            >>>def
            [안내] 대표 메뉴 이름은 무엇인가요?
            >>>aaa
            [안내] 해당 메뉴 가격은 얼마인가요?
            >>>10000
            [안내] def에 음식(aaa,10000) 추가되었습니다.[시스템] 가게 등록이 정상 처리되었습니다.
            ----------------------------------------
            1) [사장님 용] 음식점 등록하기
            2) [고객님과 사장님 용] 음식점 별점 조회하기
            3) [고객님 용] 음식 주문하기
            4) [고객님 용] 별점 등록하기
            5) 프로그램 종료하기
            ----------------------------------------
            [시스템] 무엇을 도와드릴까요?
            >>>1
            def
            {}
            aaa
            {aaa=10000, ccc=10000, bbb=10000}
            [안내] 반갑습니다. 가맹주님!
            [안내] 음식점 상호는 무엇인가요?
            >>>def
            [안내] 대표 메뉴 이름은 무엇인가요?
            >>>aaa
            [안내] 해당 메뉴 가격은 얼마인가요?
            >>>10000
            [안내] def에 음식(aaa,10000) 추가되었습니다.
            [시스템] 가게 등록이 정상 처리되었습니다.
            ----------------------------------------
            1) [사장님 용] 음식점 등록하기
            2) [고객님과 사장님 용] 음식점 별점 조회하기
            3) [고객님 용] 음식 주문하기
            4) [고객님 용] 별점 등록하기
            5) 프로그램 종료하기
            ----------------------------------------
            [시스템] 무엇을 도와드릴까요?
            >>>1
            def
            {aaa=10000}
            aaa
            {aaa=10000, ccc=10000, bbb=10000}
            [안내] 반갑습니다. 가맹주님!
            [안내] 음식점 상호는 무엇인가요?
            >>>def
            [안내] 대표 메뉴 이름은 무엇인가요?
            >>>bbb
            [안내] 해당 메뉴 가격은 얼마인가요?
            >>>10000
            [안내] def에 음식(bbb,10000) 추가되었습니다.
            [시스템] 가게 등록이 정상 처리되었습니다.
            ----------------------------------------
            1) [사장님 용] 음식점 등록하기
            2) [고객님과 사장님 용] 음식점 별점 조회하기
            3) [고객님 용] 음식 주문하기
            4) [고객님 용] 별점 등록하기
            5) 프로그램 종료하기
            ----------------------------------------
            [시스템] 무엇을 도와드릴까요?
            >>>1
            def
            {aaa=10000, bbb=10000}
            aaa
            {aaa=10000, ccc=10000, bbb=10000}
            [안내] 반갑습니다. 가맹주님!
            [안내] 음식점 상호는 무엇인가요?
            >>>
            ```
            
        
        ➡️ 가게 등록시에 등록된 shop의 이름과 등록된 menu와 가격을 볼 수 있게 해둠
        처음으로 등록된 가게 aaa에 메뉴를 ccc와 bbb를 등록할 때는 문제 없이 등록이 잘 되었음
        <br/>

        ➡️ 그런데 두번째로 등록할 가게인 가게 이름 : def 에 음식 aaa와 가격 10000원을 추가했더니 aaa라는 가게에 메뉴aaa가 등록된 것을 볼 수 있음
        
        <br/>
        def라는 가게는 생성되었으나 비어있는 arraylist를 가지고 있음 <Br/>
        -> 그래서 다시 aaa를 추가했더니 등록된 메뉴라는 말도 안뜨고 정상적으로 추가됨
        <br/>
        그래서 추가로 def에 bbb라는 메뉴를 추가했고 정상처리가 되었다.
        결국 1번째 메뉴는 다른 shop에 등록되고, 2번째와 3번째 메뉴는 정상등록됨
        <Br/> 

    2. 아무래도 노란박스로 넘어오는듯…
        
        <img src="https://github.com/user-attachments/assets/3d30afe6-4875-4a48-b294-b104fa52b0d6" width=500 />
        
    3. def 가게에 aaa가 def에 등록되어야하는데, 기존에 있던 aaa에 등록됨.
    4. 노란박스부분에 수정해줌.
        
        <img src="https://github.com/user-attachments/assets/fb4f8464-18d0-491b-982b-d23479dd9b45" width=400/>
         
        1. Arraylist의 마지막 배열에 추가 하고싶을 때 shops.add(size()-1,object)이 아니라 shops.add(object)또는 shops.add(shops.size(),object)로 추가해야 함. 
    5. 결국 노란박스 부분만 살리면 되고 위에  빨간 박스 부분은 삭제하면 됨 
        1. arraylist에서 index 를 꼭 알아야 add할 수 있다고 생각했는데 index몰라도 add하면 어짜피 맨 마지막 index에 추가 됨
        
        <img src="https://github.com/user-attachments/assets/b06d46c6-7f59-43cd-a1b0-93d285459782" width=400/>
        

1. 안되는 부분 -2,  NPE 발생
    
    <img src="https://github.com/user-attachments/assets/cf210bd5-acf9-4405-b910-448dd76e1d32" width=400 />
    
    1. NPE발생
        
        ```sql
        Exception in thread "main" java.lang.NullPointerException: 
        Cannot invoke "java.util.ArrayList.add(Object)" because "this.orders" is null
        	at jungmin.kdelivery.KDeliveryMain.deliveryOrder(KDeliveryMain.java:117)
        	at jungmin.kdelivery.KDeliveryMain.main(KDeliveryMain.java:209)
        
        Process finished with exit code 1
        
        ```
        
        >이유  orders  초기화 하는 거 깜 박 함…!
        
        <img src="https://github.com/user-attachments/assets/ca3bd28d-41fe-476c-8e5a-f1f77c48b7ac" witdh =400/>
        
2. Reference와 나의 차이점 
    1. Arraylist를 초기화 할 때 강사님 : 필드에 VS 나 : 생성자
        1. 둘 다 객체를 생성해도 동일한 초기값을 가짐
        2. 필드에서 할 경우
            1. 모든 생성자에서 동일한 초기값을 사용하게 됨
            2. 코드가 간단하고 명확하며
            3. 가본값을 항상 일관되게 설정 할 수 있음
        3. 생성자 에서 할 경우
            1. 생성자는 생성자 오버로딩이 가능하므로 유연하게 사용가능
            2. 즉, 매개변수에 따라 초기화가 달라져야할 때 유용