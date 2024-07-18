---
layout : single
title : "Java StringBuilder "
categories: Programmers
tag : [CodingTest, 개념, java]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


### StringBuilder

프로그래머스에서 문제 풀다가 지난 번 StringBuilder를 많이 사용하길래
아직 생소해서 사용하지는 못했었는데 찾아보니 유용한 기능이 많아서(특히 코테풀때!!!) 정리해보았다😸

관련문서 : [https://docs.oracle.com/javase/tutorial/java/data/buffers.html](https://docs.oracle.com/javase/tutorial/java/data/buffers.html)

1. java.lang.StringBuilder; 
2. 예제 및 설명 포함
    
    ```java
    public class Test {
        public static void main(String[] args) {
            StringBuilder strBuildTest = new StringBuilder();
    
            //문자열 추가
    
            strBuildTest = strBuildTest.append("kyungbumIsAngry");
            //타입이 String이 아닌 객체임 (StringBuilder)
            //String abc = strbuildTest.append("kyungbumIsAngry"); 는 안됨.
            System.out.println("Test1:" +strBuildTest);
    
            //숫자 추가
            int appendNumber =9879864;
            strBuildTest.append(appendNumber);
            System.out.println("Test2:" +strBuildTest);
    
            //문자열 삽입
            String str = "smart";
            strBuildTest.insert(10,str); // index
            System.out.println("Test3:" +strBuildTest);
    
            //문자열 치환, 문자열 교체
            strBuildTest.replace(15,20,""); //교체 문자 시작, 교체 문자 종료, 교체할 문장
            System.out.println("Test4:" + strBuildTest);
    
            //문자열 자르기
            //strBuildTest.substring(15,21); -> 원본이 변경되지 않음
            //strBuildTest.subSequence(15,21); -> 원본이 변경되지 않음
            StringBuilder strBuildTest2 = new StringBuilder(strBuildTest.substring(15, 22));
            System.out.println("Test5:" + strBuildTest2); //Test5:987986 출력됨.
    
            //원본을 저장하고 싶을 때는 Delete사용
            strBuildTest.delete(15,22);
            System.out.println("Test6:" + strBuildTest);
    
            //문자 대체(교환) ,char type
            strBuildTest.setCharAt(10,'m');
            System.out.println("Test7:" + strBuildTest);
    
            //문자열 길이 조절
            strBuildTest.setLength(8);
            System.out.println("Test8:" + strBuildTest);
    
            //문자열 뒤집기
            strBuildTest.reverse();
            System.out.println("Test9:" + strBuildTest);
    
            //문자열 변환
            strBuildTest.toString();
        }
    
    }
    
    ```
    

- 예제 출력
    
    ```java
    Test1:kyungbumIsAngry
    Test2:kyungbumIsAngry9879864
    Test3:kyungbumIssmartAngry9879864
    Test4:kyungbumIssmart9879864
    Test5:9879864
    Test6:kyungbumIssmart
    Test7:kyungbumIsmmart
    Test8:kyungbum
    Test9:mubgnuyk
    
    ```

