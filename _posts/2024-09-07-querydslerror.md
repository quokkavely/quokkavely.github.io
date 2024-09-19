---
layout : single
title : "[Project] QueryDSL 문제해결"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

### 에러 1

1. QueryDSL 을 적용하고 나서 바로 겪은 문제…!
    
    <img src= "https://github.com/user-attachments/assets/5339461e-993e-4f8b-b4ce-2b78c007f979" width=500/>
    
    <img src= "https://github.com/user-attachments/assets/4f4a0ea9-fc5a-41a4-9c59-ffed85548c37" width=500/>
    
    왜 그런가 하고 찾아보다가 혹시 순서가 중요한가 해서 수정했더니 해결되었다.
    
    아무리 해도 안되더니 메서드 순서대로 매개변수 순서를 조정하니까 해결됨…!
    

### 에러2

1. 문제 
    - 이제 된 줄 알았는데 status/ buyerCode/ itemCode 중에 하나라도 안 넣으면 데이터가 나오지 않음
        
         <img src= "https://github.com/user-attachments/assets/78e565f6-bd7e-4b37-b363-a78c228be362" width=500/>
        
    - 조건을 모두 다 넣으면 잘만나온다
        
         <img src= "https://github.com/user-attachments/assets/2f3b7f3e-8d8d-4924-8ab1-7a9e7f78513d" width=500/>
        
    - null 이 문제라고 생각되어서 requestParam으로 받던 것을 DTO 클래스로 변경해서 받았지만 그래도 해결되지 않았다.,
        
         <img src= "https://github.com/user-attachments/assets/c9d52f5b-d72b-4b65-9dad-93948ca895ea" width=500/>
        
2. 해결
    - 원인
        
        기존에 만들었던 OrderHeadersRepository 인터페이스에서 JPA가 자동으로 메서드를 생성하려고 시도하는 `findByRequestDateBetweenAndOrderStatusAndBuyer_BuyerCdAndOrderItems_ItemCD` 메서드를 제거하고 `OrderQueryRepositoryImpl` 클래스에서 QueryDSL로 쿼리를 구현하는 방식으로 처리했더니 해결되었다.
        
    - 결국 OrderHeadersRepository를 Custom에서 상속받아 사용햇는데 이를 제거했더니 null 이 들어와도 동적으로 잘 처리된다.


두가지 문제 모두 기존에 있던 OrderHeadersRepository를 상속받아 구현하였을때 발생한 문제인데 굳이 상속을 받지 않고 custom interface와 이를 구현하는 impl클래스만 있으면 모두 해결 된다.

