---
layout : single
title : "[JAVA] 컬렉션프레임워크, 예외처리"
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

## 컬렉션 연습문제

손도 못대서 못 푼 문제는 19번, 21번.

24번은 조금 오래 걸렸음

꼭 다시 풀어보기!

### 19

1. 입출력예시
    
    19번은 당췌 무슨소린지 모르겠다고 생각했는데
    
    ```java
    HashMap<String, String> hashMap = new HashMap<String, String>(){{
    	put("firstName", "김");
    	put("lastName", "코딩");
    }};
    HashMap<String, String> output =  addFullNameEntry(hashMap);
    System.out.println(output); // --> {firstName=김, fullName=김코딩, lastName=코딩}
    ```
    
    입출력예시인데 사실 봐도 모르겠어서  map은 key랑 value밖에없는데 어떻게 3개나 들어가지 했는데 애초에 새로 만들어진  hashmap은 이름만 들어가는 거 였음.
    
2. 내 풀이
    
    ```java
    public class S_addFullNameEntry {
        public HashMap<String, String> addFullNameEntry(HashMap<String, String> hashMap) {
        
           HashMap<String,String> fullName =new HashMap<>();
           String a = hashMap.get("firstName");
           String b = hashMap.get("lastName");
    
             fullName.put("firstName",a);
             fullName.put("fullName",a+b);
             fullName.put("lastName",b);
    
             return fullName;
        }
    }
    ```
    
    - 문제를 이해하고 나서는 쉽게 풀었음,

### 22

1. username과 password로 회원이 맞는지 확인하기. / containsValue 사용금지
2. 입출력예시
    
    ```java
    HashMap<String, String> member = new HashMap<String, String>(){{
    	put("kimcoding", "1234");
    	put("parkhacker", "qwer");
    }};
    boolean output = isMember(member, "parkhacker", "qwer");
    System.out.println(output); // --> true
    ```
    
3. 처음 푼 풀이
    
    ```java
    public class V_isMember {
       public boolean isMember(HashMap<String, String> member,
    												   String username, String password){
    		
    		HashMap<String,String> namePass=new HashMap<>();
    		namePass.put(username,password);
    		 if((namePass.entrySet()).equals(member)) {
    	        return true;
    		  }
    		     return result;
       }
    ```
    
    - 이제와서 보니 result를 왜 할당도 안해놓고 리턴하는지?? 모르겠음..풀다가 포기 했는 듯
    - 의미 없는 풀이를 했음 ⇒ hashmap에 name이랑 password를 넣어두고는 그 값을 다시 비교하고 있기 때문에, 의미 없음
    - 결국 못 풀고 집에서 다시 풀었음.
4. 다시 푼 풀이
    
    ```java
    import java.util.HashMap;
    
    public class V_isMember {
        public boolean isMember(HashMap<String, String> member, String username, String password) {
            for(String user: member.keySet()){
                if(user.equals(username)){
                    if(member.get(user).equals(password)){
                        return true;
                    }
                }
            }
        return false;
        }
    }
    
    ```
    
    - for문 돌려서 회원정보가 들어있는 map의 key들만 모아서 key와 username을 비교한 후 같다면 get으로 키를 넣어  값과 password비교해서 같으면  true, 이외의 조건은 모두 false로 받음.
    - 이렇게 풀이를 보면 쉬운데 왜 코드로 칠때는 힘든지…ㅠㅠㅠ 많이 쳐보고 생각많이 해보기
    

### 24

1. key : str의 letter, value : letter의 갯수를 가지는 HashMap 반환
2. 입출력예시
    
    ```java
    HashMap<Character, Integer> output = countAllCharacter("banana");
    System.out.println(output); // --> {b=1, a=3, n=2}
    ```
    
3. 내가 푼 풀이
    
    ```java
    import java.util.HashMap;
    
    public class X_countAllCharacter {
        public HashMap<Character, Integer> countAllCharacter(String str) {
            if(str.isEmpty()){
                return null;
            }
            HashMap<Character,Integer>letter = new HashMap<>();
            for(int i=0; i<str.length();i++){
                Character key = str.charAt(i);
                if(letter.containsKey(key)){
                  int value = letter.get(key);
                  letter.put(key,value+1);
                }else{
                    letter.put(key,1);
                }
            }
            return letter;
        }
    }
    ```
    
    - 이거 구현하다가 코드가 살짝 복잡하고 반은 맞고 반은 틀리게 나오길래 인텔리제이 창 하나 더 열어서 콘솔 출력하면서 풀었음…ㅎ
    - key가 존재한다면 해당 키의 기존값 +1, 존재하지 않으면 1로 할당.
    - 근데 어짜피 다 순회돌아야해서 향상된 for문 사용하는 것이 좋음,,
        
        ```java
             for(char key:arr){
                if(result.containsKey(key)){
                    int value = result.get(key);
                    result.put(key,value+1);
                }else{
                    result.put(key,1);
                   }
                }
        
        ```