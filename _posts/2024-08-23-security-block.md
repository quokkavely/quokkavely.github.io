---
layout : single
title : "[Project] spring security 탈퇴 및 계정 정지 회원의 로그인 차단"
categories: Project
tag : [project1, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

신고 기능을 구현한 다음, 계정 정지된 회원과 탈퇴된 회원은 여전히 로그인도 되고 활동도 가능하다.<br> 예전에 게시판 할 때는 회원의 상태를 보고 서비스 계층에서 회원의 상태에 따라 접근을 막았는데 <br>  지금은 그때처럼 관리자냐 회원이냐가 아니라 로그인을 막는 것이 상황이 가장 적절하기도 해서 Security를 개선하기로 했다.

`Authentication`이 성공했을 때, 회원의 상태를 체크하고, 상태가 '정지'나 '탈퇴'인 경우 예외를 던지도록 했다.<br>  이렇게 하면 불필요하게 서비스 계층에서 일일이 확인할 필요 없이, <br> 로그인 단계에서 차단할 수 있다.
<br> 

## Exception class 추가

추상화 된  `AuthenticationException`  를 상속하여 예외 클래스를 생성하였다.

<img src="https://github.com/user-attachments/assets/301108b8-0bb8-43cd-ab39-f07a7fded533" width=500/>

1. `SuspendedAccountException`
    
    <img src="https://github.com/user-attachments/assets/90ef1b1d-294f-449c-80eb-a9db8240aadd" width=500/>

    <br> 
    
2. `WithdrawnAccountException`
    
    <img src="https://github.com/user-attachments/assets/46418b52-9958-4da5-88bf-93c59a164391" width=500/>
    

 
<br> 

## MemberAuthenticationSuccessHandler 클래스 에 기능 추가

그 다음 탈퇴나 계정 정지된 회원이 로그인 할 경우 위에서 만든 예외를 발생시킨다. 

<img src="https://github.com/user-attachments/assets/886036d4-fc0e-47f1-b92a-a52566232691"width=500/>

1. `Authentication`이 성공했을 때 호출되는 핸들러에서 로그인한 `member`의 상태를 체크한다.
2. MemberStatus를 확인 후 정지상태나 탈퇴상태의 회원일 경우 예외를 던지도록 구현했다.
3. 이렇게 하면 Auauthorized 만 나오기 때문에 로그인 실패와 동일한 에러가 발생한다. 조금 더 명확하게 메세지를 보내주는 것이 맞을 것 같아서  추가로 ErrorResponse 클래스를 활용하여 예외를 예쁘게 포장하기로 했다.
    
    <img src="https://github.com/user-attachments/assets/f8f81312-f778-4ee6-8aaf-29418582322e" width=500/>
    <br> 

## 적용 결과

### 정지 회원

1. 신고 승인
    
    <img src="https://github.com/user-attachments/assets/3ff89adb-c9e9-49cb-a7ce-2cf62802b7ae" width=500/>
    
2. 계정 정지 후 로그인 
    
    <img src="https://github.com/user-attachments/assets/e5fcfcca-55f5-4f89-a5ad-44f4ec411370" width=500/>
    

### 탈퇴 회원

1. 탈퇴 전 로그인
    
    <img src="https://github.com/user-attachments/assets/a9dde2ca-54f8-4787-84d2-fe2ea14b8839" width=500/>
    
2. 탈퇴
    
    <img src="https://github.com/user-attachments/assets/b4d543f4-78a3-4d27-878d-811be0997821" width=500/>
    
3. 탈퇴 후 로그인
    
    <img src="https://github.com/user-attachments/assets/3a052992-4d1c-42cc-aef7-a6ce4152cbea" width=500/>
    

이제 삭제된 회원은 로그인 시도부터 차단되고, 시스템에 접근할 수 없고 예외도 예쁘게 잘 나오고 처리도 잘된다.

