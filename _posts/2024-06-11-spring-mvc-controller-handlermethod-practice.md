---
layout : single
title : "[Spring MVC] controller handlerMethod 실습"
categories: Spring
tag : [MVC,실습,Spring]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 핸들러메서드 구현

## MemberController 핸들러 메서드

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.annotation.PostConstruct;
import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/v1/members")
public class MemberController {
    private final Map<Long, Map<String, Object>> members = new HashMap<>();

    @PostConstruct
    public void init() {
        Map<String, Object> member1 = new HashMap<>();
        long memberId = 1L;
        member1.put("memberId", memberId);
        member1.put("email", "hgd@gmail.com");
        member1.put("name", "홍길동");
        member1.put("phone", "010-1234-5678");

        members.put(memberId, member1);
    }
```

1. 회원정보 수정을 위한 핸들러 메서드 구현
    
    1) 처음 작성 코드
    
    ```java
        @PatchMapping("/{member-id}")
        public ResponseEntity editMember(@PathVariable("member-id") long memberId){
            Map<String, Object> member1 = new HashMap<>();  
            if (members.get(1)==member1) {
                member1.put("phone","010-1111-2222");
                return new ResponseEntity<>(HttpStatus.OK);
            }else{
                return new ResponseEntity<>(HttpStatus.NOT_FOUND);
            }
        }
    ```
    
    - member id가 1일 때 수정하는 거니까 key가 1(memberId=1)일때  값으로 member1에 해당되면 member1의 phone에 해당하는 전화번호를 010-1111-2222로 수정한 후 ResponseEntity로 리턴.
    - 근데 자꾸 HTTP상태코드 400번대만 뜨고 다른 방법을 모르겠어서 강사님께 도움 요청
    
    2) 다시 푼 코드
    
      강사님 조언듣고 생각한 코드 1) 의 문제점.
    
    - 매개인자로 @RequestParam 없음 → 클라이언트에서 쿼리전송하면 서버에서 전달받을때 사용
    - member가 1이라는 것은 요청에 의한 것(매개인자에 오는 값)이므로 해당 key가 존재하는지를 확인해서 있으면 번호 수정.
    - 번호 수정은 쿼리에서 진행하니까 매개인자 RequestParam의 변수로 정해줄 것.
    - ResposeEntity에 바디도 같이 리턴해야 함 → 요구 조건에 명시되어있음.
    
    ```java
     @PatchMapping("/{member-id}")
        public ResponseEntity editMember(@PathVariable("member-id") Long memberId,@RequestParam("phone") String phone){
            Map<String, Object> member1 = new HashMap<>();
            if(members.containsKey(memberId)) {
                member1 = members.get(memberId);
                member1.put("phone",phone);
                return new ResponseEntity<>(member1,HttpStatus.OK);
            }else{
                return new ResponseEntity<>(member1,HttpStatus.NOT_FOUND);
            }
        }
    ```
    
    - memberId가 존재한다면 값에 해당하는 객체를 member1 변수로 지정
    - member1의 변수에 phone이라는 키의 값에 해당하는 phone을 수정.
        
        (값은 중복이 안되서 put할 경우 기존의 값에서 새로운 값으로 변경됨)
        
    - ResposeEntity에 body를 포함해야하므로 member1과 상태코드를 함께 반환.
2. 회원정보 삭제를 위한 핸들러 메서드 구현
    
    1) 처음 작성한 코드
    
    ```java
     @DeleteMapping("/{memberId}")
        public ResponseEntity deleteMember(@PathVariable("member-id") long memberId){
            if (members.containsKey(memberId)) {
    
                members.remove(memberId);
                return new ResponseEntity<>(null,HttpStatus.NO_CONTENT);
            }else{
                return new ResponseEntity<>(HttpStatus.NOT_FOUND);
            }
        }
    ```
    
    자꾸 500번 상태코드뜨면서 안되길래 콘솔보니 
    
    ```java
    Resolved [org.springframework.web.bind.MissingPathVariableException: Required URI template variable 'member-id' for method parameter type long is not present]
    ```
    
    알고보니 DeleteMapping의 경로와 pathVariable의 경로가 일치하지 않아서 나는 오류 임.  둘의 문자열은 같아야 한다! 잊지말기
    

## 핸들러 메서드 구현 2

1. coffeeId가 1인 커피 정보 중에서 아래 정보를 수정하는 핸들러 메서드
    
    ```java
        // 1. 커피 정보 수정을 위한 핸들러 메서드 구현
        @PatchMapping("/{coffeeId}")
        public ResponseEntity editCoffee(@PathVariable ("coffeeId")long coffeeId,
                                         @RequestParam("korName") String korName,
                                         @RequestParam("price") int price){
            Map<String,Object> coffee1=new HashMap<>();
            if(coffees.containsKey(coffeeId)){
                coffee1=coffees.get(coffeeId);
                coffee1.put("korName",korName);
                coffee1.put("price",price);
                return new ResponseEntity<>(coffee1, HttpStatus.OK);
            }else{
                return new ResponseEntity<>(HttpStatus.NOT_FOUND);
            }
        }
    ```
    
2. coffeeId가 1인 커피 정보를 삭제하는 핸들러 메서드
    
    ```java
     // 2. 커피 정보 삭제를 위한 핸들러 서드 구현
        @DeleteMapping("/{coffeeId}")
        public ResponseEntity deleteCoffee(@PathVariable("coffeeId") long coffeeId){
            if(coffees.containsKey(coffeeId)){
                coffees.remove(coffeeId);
                return new ResponseEntity<>(null,HttpStatus.NO_CONTENT);
            }else{
                return new ResponseEntity<>(HttpStatus.NOT_FOUND);
            }
        }
    ```
    
<br/>

 ### comment

1-1번에서 한번 감 잡고나니까 대충 어떤 식으로 돌아가는지 이해가 간다.
처음에 힘들었던 이유는 annotation도 많고 왜 사용하는 건지 이해도 못했는데
직접 코드치고 annotation은 공식문서나 다른 블로그도 읽어보고 하니
왜 사용하는지 어떻게 쓰는 건지 대략 감이 와서 다행이었다.
dto 들어가면 더 어렵다는데 ~~ 부디 따라잡을 수 있기를!



<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>