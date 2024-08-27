---
layout : single
title : "[Project] 탈퇴 및 계정 정지 회원의 로그인 차단"
categories: Project
tag : [project1, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---

## 탈퇴 및 계정 정지 회원의 로그인 차단

프로젝트에서 회원을 삭제해도 DB에서는 상태만 변경되고, 실제 데이터는 남아있게 처리하고 있다. 문제는 이렇게 삭제된 회원이 여전히 로그인을 시도할 수 있고, 로그인 이후에도 닉네임 변경이나 게시글 작성 같은 기능들을 사용할 수 있다는 점이었다. 이런 상황을 방치하면 보안상 문제가 될 수 있어서, 삭제된 회원의 로그인을 차단하고, 접근도 제한하는 방법을 고민하게 됐다.

### 기존 문제점: 인증되지 않은 회원의 권한 남용

현재 구조에서는 삭제된 회원이라도 인증되지 않은 상태로 로그인 후, 다양한 활동을 할 수 있었다. 예를 들어:

- **닉네임 변경 가능**: 삭제된 회원이 로그인 후 닉네임을 변경할 수 있다.
- **게시글 작성 가능**: 인증되지 않은 상태로도 게시글을 올릴 수 있다.

이런 문제가 발생하는 이유는 서비스 계층에서만 회원 상태를 확인하고 권한을 주다 보니, 보안 설정(Security)을 적용한 의미가 퇴색된다는 것이다. 각각의 서비스 메서드에서 일일이 회원 상태를 확인하고 예외를 던지거나 권한을 제한하는 것은 비효율적일 뿐 아니라, 코드가 복잡해지고 실수할 가능성도 높아진다.

### 해결책: Security에서 회원 상태 확인 후 차단

이 문제를 해결하기 위해 Security에서 회원 상태를 확인한 뒤, 문제가 있는 경우 바로 차단할 방법을 찾았다. 

구체적으로는 `Authentication`이 성공했을 때, 회원의 상태를 체크하고, 상태가 '정지'나 '탈퇴'인 경우 예외를 던지도록 했다. 이렇게 하면 불필요하게 서비스 계층에서 일일이 확인할 필요 없이, 로그인 단계에서 차단할 수 있어 더 안전하다.

### 구현: MemberAuthenticationSuccessHandler 클래스 활용

이 로직을 구현하기 위해 `MemberAuthenticationSuccessHandler`클래스를 활용했다. 이 클래스는 `Authentication`이 성공했을 때 호출되는 핸들러인데, 여기서 로그인한 `member`의 상태를 체크했다.

`*MemberAuthenticationSuccessHandler` 클래스에서 member 정보를 찾은 다음에 계정 상태를 보고  정지나 탈퇴는 예외를 던진다.*

<img src = "https://github.com/user-attachments/assets/6be1970e-799c-4ede-8c16-434f242d4575" width =500/>

1. **Member 정보 가져오기**: `Authentication`이 성공했을 때, 해당 회원의 정보를 가져왔다.
2. **회원 상태 확인**: 가져온 `member`의 상태를 확인하고, '정지'나 '탈퇴' 상태인 경우 바로 예외를 던지도록 설정했다.
3. **예외 처리**: 예외가 발생하면 해당 회원은 로그인할 수 없고, 접근도 제한된다. 이를 통해 삭제된 회원이 더 이상 시스템에 접근할 수 없도록 했다.
    
    
    <img src = "https://github.com/user-attachments/assets/a2588621-66a1-4c20-b7d7-554a4e3f78ff" width =500/>
    
    - 또한, 콘솔에 예외가 계속 터지는 걸 보고 싶지 않아서, 예외 처리를 통해 로그를 줄였다. 이렇게 하면 불필요한 콘솔 로그를 피하면서도, 보안 설정이 제대로 적용되는 걸 확인할 수 있다.
    - 그런데 이렇게 했더니 메세지를 명확하게 출력하고 싶은데 너무 간단하게 나와서 생각해보니까`*FailureHandler` 클래스의 메서드에서*  ErrorResponse가 이렇게 구현되어 있기 때문이다.
        
        ```java
        ErrorResponse errorResponse = ErrorResponse.of(HttpStatus.valueOf(HttpStatus.UNAUTHORIZED.value()), exception.getMessage());
        
        ```
        

1. `*MemberAuthenticationFailureHandler` 클래스에서 ErrorResponse를 활용하여 예외를 예쁘게 포장한다.*
    
    
    <img src = "https://github.com/user-attachments/assets/93611d67-74e0-4fa6-b4b5-f1774d441e96" width =500/>
    

### 결과: 깔끔하고 안전한 접근 제한


<img src = "https://github.com/user-attachments/assets/dc3157f9-b43d-4f5f-8fe4-36cd2e56f892" width =500/>

이제 삭제된 회원은 로그인 시도부터 차단되고, 시스템에 접근할 수 없게 되었다. 

이렇게 Security에서 회원 상태를 한 번에 확인하고 차단하는 방식은 코드의 복잡성을 줄이면서도 보안을 강화하는 데 큰 도움이 됐다. 더 이상 서비스 계층에서 불필요한 상태 확인을 하지 않아도 되고, 전체적인 코드가 훨씬 깔끔해졌다.

예외도 예쁘게 잘 나오고 처리도 잘된다.


<img src = "https://github.com/user-attachments/assets/a10bbbd8-169e-4898-a0a7-e4781ff571e2" width =500/>