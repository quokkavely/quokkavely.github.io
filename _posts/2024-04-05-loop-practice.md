---
layout : single
title : "[JAVA] 조건문 반복문 연습문제"
categories: JAVA-Practice
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: true
---

# 조건문 연습문제

### 15번

1. 시, 분, 초를 입력받아 1초를 더한 결과값을 특정 형태의 메시지로 리턴

2. 출력

   1. `String` 타입을 리턴
   2. `1초 뒤에 {hour}시 {minute}분 {second}초 입니다` 형식으로 리턴

3. 입출력예시

   ```java
   String output = addOneSecond(14, 17, 59);
   System.out.println(output); // --> "1초 뒤에 14시 18분 0초 입니다"
   
   output = addOneSecond(1, 59, 59);
   System.out.println(output); // --> "1초 뒤에 2시 0분 0초 입니다"
   
   output = addOneSecond(3, 24, 29);
   System.out.println(output); // --> "1초 뒤에 3시 24분 30초가 입니다"
   
   output = addOneSecond(23, 59, 59);
   System.out.println(output); // --> "1초 뒤에 0시 0분 0초 입니다"
   ```

   

4. 나의 오답

   ```java
       public String addOneSec(int hour, int minute, int second) {
           String result;
           second++;
           if (second >= 60) {
               result = String.format("1초 뒤에 %d시 %d분 %d초 입니다", hour, minute, second - 60);
               {
                 if (minute >= 60) {
                 result = String.format("1초 뒤에 %d시 %d분 %d초 입니다", hour, minute - 60, second - 60);
                 }
                 if (hour >= 24) {
                 result = String.format("1초 뒤에 %d시 %d분 %d초 입니다", hour - 24, minute - 60, second - 60);
                 }
               }
           } else {
               result = result = String.format("1초 뒤에 %d시 %d분 %d초 입니다", hour, minute, second);
           }
           return result;
       }
   ```

- 문제점

   1. 1초가 증가했기 때문에 60 이상일 경우를 포함 할 필요가 없음 60일 경우에만 거르면 됨.

   2. 경우의 수마다 하나하나 String.format으로 결과를 정해주지 않아도 됨…! 조건문 나온 결과를 가지고 hour, minute, second 정해지면 그때만 String.format으로 리턴값 지정해줘도 충분

   3. 결정적으로는 1초가 증가했는데 60을 빼줬으면 minute나 hour값도 올려줘야 함> 자동으로 60초라고 minute이나 hour이 증가하지 않기 때문,  99가 1증가 했다고 100이 되는 건 당연하지만 시간 시스템은 내가 하나하나 지정해줘야만 가능!!

      

5. 다시 풀어본 풀이

   ```
   public String addOneSecond(int hour, int minute, int second) {
   
           second++;
           //1초 증가할때 메세지
   
           //1. 1초 증가하면 14시 17분 59초일 경우
           //2.  14시 59분 59초일 경우
           //3. 23시 59분 59초일 경우
   
           if (second == 60) {
               minute++;
               second = 0;
               if (minute == 60) {
                   minute = 0;
                   hour++;
                   if (hour == 24) {
                       hour = 0;
                   }
               }
           }
           return String.format("1초 뒤에 %d시 %d분 %d초 입니다", hour, minute, second);
       }
   ```

   1. 조건문을 이해하고 나서는 그래도 쉽게 다시 풀 수 있었던 것 같다.
   2. 이제는 왜 저렇게 생각했을까 싶음.

****

<br/>
<br/>

# 반복문 연습문제

### 2번

1. 수를 입력받아 홀수인지 여부를 리턴

2. 출력 : boolean 타입을 리턴

3. 조건

   1. 반복문(`while`)문을 사용
   2. `for`문 사용은 금지.
   3. 나눗셈(`/`), 나머지(`%`) 연산자 사용은 금지됩니다.
   4. 0은 짝수로 간주

4. 입출력 예시

   ```java
   boolean output = isOdd(17);
   System.out.println(output); // --> true
   
   output = isOdd(-8);
   System.out.println(output); // --> false
   ```

   

5. 내가 푼 풀이

   ```java
   public boolean isOdd1(int num) {
       // TODO:
   
   boolean result;
       while(num>=2){
           num-=2;
       }
   if(num==0){
       result = true;
   }else{
       result =false;
   }
   return result;
   }
   ```

   1. 문제점
      1. 이 문제 엄청 오래잡고 있어서 강사님께 도움 요청해서 힌트 얻고 이렇게 풀긴 했는데
      2. 중요한 건 음수는 못거름. 반복문은 2 이상만 오고  if절에서 음수는 모두 false로 리턴됨.

6. 다시 푼 풀이

   ```
   public class B_IsOdd {
       public boolean isOdd(int num) {
           // TODO:
   
           if(num<0){
               num=-num;
           }
   
   		while(num>2) {
      		 num -= 2;
   		}
   		if(num==1){
       		return true;
   		}
   	return false;
   
       }
   }
   ```

   1. 처음부터 음수가 들어오면 -를 붙여서 양수로 바꿔주면 음수들어와도 괜찮음
   2. 절대값 계산할 때는 절대값 계산시 ⇒ Math.abs() 또는 num= num*-1; 이렇게 고쳐줄 수 있다.


****

### 14번

1. 문자열을 입력받아 각 문자(letter) 뒤에 해당 문자의 인덱스가 추가된 문자열을 리턴

2. `String` 타입을 리턴해야 합니다.

3. 조건

   - 반복문(`for`)문을 사용

   - 빈 문자열을 입력받은 경우, 빈 문자열을 리턴

4. 내가 푼 풀이

   ```java
       public int getMaxNumberFromString1(String str) {
           // TODO:
       int max=0;
        for(int i=0;i<str.length()-2;i++){
            max = str.charAt(i) >= str.charAt(i+1)? str.charAt(i): str.charAt(i+1);
        }
        return max;
       }
   }
   ```

   1. str.charAt(i)는 문자로 저장되어 비교를 할 수 없어서 틀린건지?

      숫자도 유니코드로 저장하고,,,,,아!, 유니코드로 반환되어 결과가 틀린건지?

      `Character.getNumericValue()` 메소드는 Unicode의 값을 int value로 반환한다.

5. 다시 푼 풀이

   ```java
       public int getMaxNumberFromString(String str) {
           // TODO:
           int max = 0;
   
           for (int i = 0; i < str.length(); i++) {
               char a = str.charAt(i);
               int num = Character.getNumericValue(a);
   
               if (max < num) {
                   max = num;
               }
           }
           return max;
       }
   ```

****

### 17번

1. 1 이상의 자연수를 입력받아 소수(prime number)인지 여부를 리턴

2. 입출력예시

   ```JAVA
   boolean output = isPrime(2);
   System.out.println(output); // --> true
   
   output = isPrime(6);
   System.out.println(output); // --> false
   
   output = isPrime(17);
   System.out.println(output); // --> true
   ```

   

3. 내가 푼 풀이

   ```
       public boolean isPrime(int num) {
           // TODO:
           boolean result= false;
           if(num==2){
               result =true;
           }
           for (int i=2; i < num ; i++){
               if(num% i !=0){
                   result =true;
               }
           }
       return result;
       }
   ```

   1. 불필요한 수는 조건을 걸어 제외하기

      - 1은 소수가 아니고 2를 제외한 짝수는 전부 소수가 안됨,
      - 그리고 3부터 반복을 하지만, 짝수를 제외하고 반복 걸어서 나머지가 없는 경우를 찾기
      - 모든 조건을 다 제외해도 남았을 경우 그 숫자는 소수.

   2. 루트를 씌우면 더 쉽게 구할 수 있음

      예를 들어 45를 루트 씌우면 6.xxx임, 6까지 봤을 때 1과 2 그리고 짝수를 걸러내면 3만 보면 됨.  //제곱근 구하는 방법 : Math.squrt() - 실수는 int처리하면 소수점 다 내림 처리됨.

4. 다시 푼 풀이

   ```java
   public class Q_IsPrime {
       public boolean isPrime(int num) {
           // TODO:
           if(num==1){
               return false;
           }
           if (num == 2) {
               return true;
           }
   
           if (num > 2) {
               for (int i = 2; i < num; i++) {
                   if (num % i == 0) {
                       return false;
                   }
               }
           }
           return true;
       }
   }
   ```

   

