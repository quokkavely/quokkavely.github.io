---
layout : single
title : "[JAVA] 계산기 프로그램"
categories: JAVA-Learn
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: trueㅎ
---

## 최소 기능 구현

1. 사용자의 입력으로 첫 번째 숫자, 연산자, 두 번째 숫자를 받아야 함
2. 연산자의 종류는 +, -, *, / 네 가지
3. 소수점 연산을 수행 가능해야 함.
4. 연산 결과를 콘솔에 출력.



## 내가 만든 계산기

```java
    

	System.out.println("계산을 원한다면 아무 숫자나 눌러주세요")
    String repeat="";

      do {
        String last = scanner.nextLine();
        int l = Integer.parseInt(last);

        System.out.print("첫번째 숫자를 입력하세요:");
        String inputOne = scanner.nextLine();
        int first = Integer.parseInt(inputOne);

        System.out.print("연산자를 입력하세요, 연산자의 종류는 ' + ',' - ',' * ',' / '입니다.:");
        String op = scanner.nextLine();

        System.out.print("두번째 숫자를 입력하세요:");
        String inputTwo = scanner.nextLine();
        int second = Integer.parseInt(inputTwo);

        double result = 0;

        if (op.equals("+")) {
          result = (double) first + (double) second;
        } else if (op.equals("-")) {
          result = (double) first - (double) second;
        } else if (op.equals("*")) {
          result = (double) first * (double) second;
        } else if (op.equals("/")) {
          result = (double) first / (double) second;
        } else {
          System.out.println("연산자가 잘못되었습니다. 프로그램이 종료됩니다.");
          break;
        }

        System.out.println("결과는 " + result + "입니다.");
        System.out.println("다시 계산하기를 원하십니까? 아니라면 n을 입력해주세요.");
        repeat=scanner.nextLine().toLowerCase();

      } while (repeat.equals("N"));
    scanner.close();
    }
  }
```

1. **부족했던 점**

   1. **스캐너**

      사실 자바의 정석 예제에서 스캐너 몇번 사용 했기 때문에 스캐너 사용하는 것에는 크게 부담은 없었는데 이렇게 처음부터 만들어본 것은 처음이라서

        Scanner sc = new Scanner(System.in); 를 계속 만듬. 입력받을 때마다 스캐너 객체 생성해서 사용했는데 다 한 사람 검사받으래서 손들었는데 스캐너 왤케 많이 만들었냐고 하심..

   2. **do-while문 사용**

      처음에는 기초 기능만 구현하고 다시 반복하시겠냐는 기능은 못넣었는데

      생각보다 내가 빨리 풀어서 강사님이 심화기능 구현해보라고 하심..

      심화기능은 잘모르겠어서 구글링 해보니 do-while이나 try-catch문 사용하라고 나옴

   3. **scope 문제**

      do-while 쓰는 것은 알겠는데 중괄호내에 변수가 있으면 밖에서 사용 못하니까 답답했음.

      안에서 할당해야하는데 (while문에 사용된 repeat 같은 경우) 첨부터 n으로 할당하면 자꾸 원하는대로 출력이 안되서 어려웠음

   4. **validate 기능**

      do-while 문에 시간을 많이써서 추가 검증 기능은 구현 하지 못함.

      사실 메서드 만드는 것이 뭔지도 모르겠고 문제풀때마다 return이나 이런 것도 모르겠는데

      내가 모르는데 어떻게 만드나 싶었음..ㅠㅠㅠㅠ

      구현해야 하는 것은 아래와 같음

      -  사용자 입력 값이 숫자인지 아닌지?
      - 연산자가 + , / , - , * 4개 외에 다른 문자나 숫자가 아닌지?
      -  나누기 할 때 두번째 숫자는 0이 아니어야 함!



## 강사님 풀이 듣고 다시 풀어본 계산기

```java
package com.java.calculator;
import javax.swing.*;
import java.util.Scanner;

public class Calculator {
    public static void main(String[] args) {
        System.out.println("===Java Calculator===");
        Scanner sc = new Scanner(System.in);
        String replyCal="";
        do {
            System.out.println("첫번째 숫자를 입력해주세요: ");
            String firstNum = sc.nextLine();
            System.out.println("연산자를 입력해주세요: ");
            String op = sc.nextLine();
            System.out.println("두번째 숫자를 입력해주세요");
            String secondNum = sc.nextLine();
            if (isValid(op)){
                if(isZero(secondNum)){
                    System.out.printf("계산된 결과는 %.2f입니다.",
               		calculate(Double.parseDouble(firstNum),op,Double.parseDouble(secondNum)));
                }else{
                    System.out.println("0으로 나눌 수 없습니다. 프로그램을 종료하겠습니다.");
                }

            }else{
                System.out.println("잘못된 연산자 입니다. 프로그램을 종료하겠습니다.");
            }

             System.out.println("계산을 다시 시작하겠습니까? 중단하고 싶으면 키보드에서 n키를 눌러주세요");
            replyCal = sc.nextLine().toLowerCase().toUpperCase();

        }while(replyCal.equals("n"));
        System.out.println("계산기가 종료되었습니다.");
   	 }

      public static boolean isValid(String oper){
            String op="+-*/";
            int opCount=0;
            for(char c: oper.toCharArray()){
                if(op.indexOf(c)!=-1){
                    opCount++;
                }else{
                    return false;
                }
            }
            return opCount==1;
        }

        public static boolean isZero(String number){
             String twoNum = "0.";
             int dotcount=0;
             for(char c:number.toCharArray()){
                 if(twoNum.indexOf(c)!=-1){
                     return false;
                 }
                 if(c=='.'){
                     dotcount++;
                 }
             }
             return dotcount<=1;
        }

        
     public static double calculate(double num1, String op, double num2){
            double result=0.0;

            switch(op){
                case"+":
                    result = num1+ num2;
                  break;
                case"-":
                    result = num1-num2;
                    break;
                case"*":
                    result=num1*num2;
                    break;
                case"/":
                    result=num1/num2;
                    break;

            }
            return result;
        }

    }
```

​	사실 검증 기능 중에  메서드로 따로 만든 숫자인지 검증하는 것과 opeator 검증하는 것은 향상된 for문이라 좀 헷갈리기도 하고 사용	하기가 까다로운 건 아니라서 그냥 외우고 사용한 것 같다. 물론 이해도 되지만 약간 수박 겉핥기 식으로 이해해서 또 안보면 잊어버	릴 것 같은데..앞으로 자주 사용하게 될 것 같아서 쉬는날에 계산기를 한 번 더 만들어봐야 할 것 같다!

​	

### Comment

이때동안 조건문, 반복문, 배열 연습문제 풀면서 당황하고 남들보다 뒤쳐지는 것 같아서 많이 힘들었는데

계산기는 얼핏 심화기능도 구현해서 뿌듯했다. 프로그램 만들어보면서 메서드나 중괄호 내에 선언된 변수는 사용하지 못한다거나

이런 거에대해서 더 알게 되었고 이론보다 프로그램이 더 재밌는 고 많이 배우는 것 같기도 하다! 

아직 더 많은 기능을 구현해내기에는 실력이 많이 모자라니 더더더 공부해야지

<br/>

---

