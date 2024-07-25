---
layout : single
title : "[JAVA]Lv1_ 카카오인턴십 실패율, 비밀지도 "
categories: Programmers
tag : [CodingTest, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

### 프로그래머스 lv. 1_실패율/ 카카오인턴십

```java
import java.util.*;

class Solution {
    public int[] solution(int N, int[] stages) {
 
        //실패율 key : stage, value : 동일 stage에 있는 플레이어 수
        Map <Integer, Integer> countStage = new HashMap<>();

        for (int stage : stages) {
            countStage.put(stage,countStage.getOrDefault(stage,0)+1);
        }
				
				//key만 list 로 담기
        List <Integer> keys = new ArrayList<>();
        for(int i = 1; i <= N ; i++) {
            keys.add(i);
        }
       
       // key : stage, value : 실패율
        Map<Integer,Double> failPercent = new HashMap<>();
       
        int total = stages.length;
        for(int i = 1; i<=N; i++) {
            if(total > 0) {
                int failPlayer = countStage.getOrDefault(i,0);
                failPercent.put(i,(double) failPlayer/total);
                total-=failPlayer;
            } else {
                failPercent.put(i,0.0);
            }
            
            if(i==N){
                Collections.sort(keys, (o1, o2) 
                -> Double.compare (failPercent.get(o2), failPercent.get(o1) ) );
            }
        }

        int[] answer = new int[N];
        
        for(int i = 0; i < N ; i++) {
            answer[i] = keys.get(i);
        } 
            
        return answer;
        
    }
}
```

- double로 안하면 엉망으로 내림차순 함 (소수를 버리기 때문에 …)
- getOrDefault(key, defalut)는 map에서 key가 있으면 value 를 반환, key가 없으면 value = default
- 내림차순 하는 법은 따로 찾아봄
    1. Double.compare
        
        <img src="https://github.com/user-attachments/assets/5990b0f3-aa8b-45dc-ba98-8ee9794f7def" width=500/>
        
        > [**Double.compare](http://Double.compare)**
        > 
        > 
        > `d1`>`d2`보다 크면 1 반환
        > 
        > `d1` <`d2`보다 작으면 -1 반환
        > 
        > `d1` == `d2`와 같으면 0을 반환
        > 
        > **아래의 원시형 타입도 동일한 메서드가 있다.**
        > 
        > - `Float.compare(float f1, float f2)`
        > - `Integer.compare(int x, int y)`
        > - `Long.compare(long x, long y)`
    2. `Collections.sort`
        - Parameter
            - Collections.sort(List<T> list)
                - 리스트의 요소들이 `Comparable` 인터페이스를 구현하고 있으면 넣어주지 않아도 된다
            - Collections.sort(List<T> list, Comparator<? super T> c)
                - 주어진 리스트를 지정된 `Comparator`를 사용하여 정렬
                
        - 정렬방법
            - 리스트의 요소들을 비교하여 다음을 수행
            - 비교 결과가 양수이면 두 요소의 순서를 바꾼다 (즉, o2가 o1보다 앞에 오도록).
            - 비교 결과가 음수이면 순서 유지 (즉, o1이 o2보다 앞에 있도록 유지).
            - 비교 결과가 0이면 두 요소의 순서를 유지합니다 (즉, o1과 o2의 순서를 바꾸지 않는다)
        - 리스트를 정렬할 때 Timsort 알고리즘을 사용
            - timesort
                - Timsort는 합병 정렬(Merge Sort)와 삽입 정렬(Insertion Sort)의 하이브리드 알고리즘
                - 최선의 경우 시간 복잡도는 O(n)
                - 최악의 경우 시간 복잡도는 O(n log n)
        

### 프로그래머스 lv.1_비밀지도/카카오인턴십

```jsx
class Solution {
    public String[] solution(int n, int[] arr1, int[] arr2) {
        String[] answer = new String[n];
        
       String[] answer1 = intToBinary(n, arr1);
       String[] answer2 = intToBinary(n, arr2);
       
        
        for(int i = 0; i < n ; i++) {
             String str = "";
            for(int j =0 ; j < n ; j++) {
            if(answer1[i].charAt(j) == '1' || answer2[i].charAt(j) == '1' ) {
                str += "#";
            }else{
                str+=" ";
            }
        }
            answer[i] = str;
        }
        
        return answer;
    }
    
        public String[] intToBinary(int n, int [] arr) {
            String answer[] = new String[n];
            
                for(int i =0; i < n ;i++) {
                
                String back = Integer.toBinaryString(arr[i]);
                String front = "";
                
                if(back.length() < n) {
                    front = "0".repeat(n-back.length());
                }
                    answer[i] = front + back;
            }
            return answer;
        }
	}
```

- repeat이 생각안나서 헤맨 문제, 길이만큼 같은 문자열을 반복해서 넣고 싶을때 **repeat** 사용하면 된다.
- 그리고 한번 더 헤맸던건 charAt(i) ==1 이라고 했는데 자꾸 공백만 반환해서 왜지왜지??하다가 결국 인텔리제이 열어서 디버깅했더니 작은 따옴표 안넣어서 그런거였다. 이런 실수를 하다니…!

