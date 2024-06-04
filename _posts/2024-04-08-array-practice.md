---
layout : single
title : "[JAVA] 배열 연습문제"
categories: JAVA-Learn
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: true
---


# [JAVA] 배열 연습문제

- 문제 풀이 시 Method나 for문을 많이 사용
- 빈 배열 조건을 설정할 때 : strArray=new String[0] 또는 new String[0]

### **오답1.**

1. **Q 배열과 요소를 입력받아 주어진 요소를 배열의 맨 앞에 추가하고 해당 배열을 리턴** 
    1. **ex)[arr[0], arr[1], ..., arr[n-1], el]**
2. ***Answer 1) System.arraycopy() 로 푸는 방법***
    1. 해당 방법을 시도했는데 출력 말고 return을 몰라서 헤맸던 문제
        
        ```java
        int[]result = new int[arr.length+1]; 
        //el의 요소가 추가되어야 하므로 배열 길이는 +1
        result[arr.length]=el; //배열의 맨 마지막이 el이라고 지정. 
        System.arraycopy(arr,0,result,0,arr.length);
        return result;
        ```
        
3. ***Answer 2) System.arraycopy() 로 푸는 방법***
    
    ```java
    int[]result = new int[arr.length+1]; 
    result[arr.length]=el;
    for(int i=0; i<arr.length; i++){
    	result[i]=arr[i];
    	}
    
    return result;
    ```
    

### **오답2.**

1. **int 타입를 요소로 갖는 배열을 입력받아 짝수만을 요소로 갖는 배열 리턴**
2. 입출력 예시
    
    ```java
    int[] output = getEvenNumbers(new int[]{1, 2, 3, 4});
    System.out.println(output); // --> [2, 4]
    ```
    
3. 내가 푼 풀이
    
    ```java
    public class I_GetEvenNumbers {
        public int[] getEvenNumbers(int[] arr) {
            // TODO:
            int leng;
    
            if(arr.length%2==0){
                leng=arr.length/2;
            }else{
                leng=arr.length/2+1;
            }
            
            for(int i=0; i<leng; i++){
                if(arr[i]%2==0){
                    result+=arr[i];
                }
            }
            return result;
        }
    }
    
    ```
    
    - 문제를 잘못 해석 함 입력이 1,2,3,4 이길래 연속적인 숫자로 된 배열이 들어온다 생각해서 배열 길이를 잘못 계산
    - 배열은 추가할 수 없음, 다르게 풀어야 함, 아직 안배운 list로 푸는 방법이 있긴 하다고 함..
    - 일단 배운걸로만 풀면
    
    ```java
    public class I_GetEvenNumbers {
        public int[] getEvenNumbers(int[] arr) {
            // TODO:
            int leng=0;
            for(int i =0; i< arr.length; i++){
                if(arr[i]%2==0){
                    leng++;
                }
            }
    
            int[]result=new int[leng];
            int count= 0;
            for(int i=0; i<arr.length; i++){
                if(arr[i]%2==0){
                    result[count]=arr[i];
                    count++;
                }
            }
            return result;
        }
    }
    
    ```
    
    먼저 짝수의 개수를 구한 다음에 새로운 배열의 길이를 정해주고
    
    이중 반복문 안하고 짝수만 들어올 수 있게 구함.
    

### **오답3.**

1. 숫자 배열을 문자열 전화번호 형태로 리턴
2. **입출력 예시**
    
    ```java
    String output = createPhoneNumber(new int[]{0, 1, 0, 1, 2, 3, 4, 5, 6, 7, 8});
    System.out.println(output); // --> '(010)1234-5678'
    ```
    

1. **내가 푼 풀이**
    
    ```java
    int[] first=new int[3];
            int[] mid = new int[4];
            int[] last = new int[4];
    
            if(arr.length==8) {
                for (int i = 0; i < 4; i++) {
                    mid[i] += arr[i];
                }
                for (int i = 4; i < 8; i++) {
                    last[i]+=arr[i];
                }
                result = "(010)-" + mid + "-" + last;
            }
            if(arr.length==11) {
                for (int i = 0; i < 3; i++) {
                    mid[i] += arr[i];
                }
                for (int i = 3; i < 7; i++) {
                    mid[i] += arr[i];
                }
                for (int i = 7; i < 11; i++) {
                    last[i]+=arr[i];
                }
                result = "("+first+")-" + mid + "-" + last;
            }
    ```
    
- 처음에 풀었던 방법이 왜 틀렸는지 이해가 가지 않아서 계속 고민했는데
- Index 4 out of Bounds for Length 4 오류가 떴고 index 값이 잘못 되었다는 것을 알게 됨.

1. **다시 고친 풀이**
    
    ```java
    public class R_CreatePhoneNumber {
        public String createPhoneNumber(int[] arr) {
            // TODO:
            String result="";
            int[] first=new int[3];
            int[] mid = new int[4];
            int[] last = new int[4];
    
            if(arr.length==8) {
                for (int i = 0; i < arr.length-4; i++) {
                    mid[i] += arr[i];
                }
                for (int i = 4; i < arr.length; i++) {
                    last[i-4]+=arr[i];
                }
                result = "(010)-"+ Arrays.toString(mid) + "-" + Arrays.toString(last);
            }
            if(arr.length==11) {
                for (int i = 0; i<arr.length-8; i++) {
                    first[i] += arr[i];
                }
                for (int i = 3; i < arr.length-4; i++) {
                    mid[i-3] += arr[i];
                }
                for (int i = 7; i < arr.length; i++) {
                    last[i-7]+=arr[i];
                }
                result = "("+Arrays.toString(first)+")-" + Arrays.toString(mid) + "-" + Arrays.toString(last);;
            }
    
            return result;
        }
    }
    ```
    
    - 그러나 String 배열 형태로 반환 되는데, ["([0, 7, 0])-[9, 8, 7, 6]-[3, 2, 1, 4]"
    - int 배열을 String으로 변환하는 방법이 따로 있는지 찾아보거나
    - [ ,], 를 replaceAll 을 사용하여 바꿔야하는지?
    
2. 위 방법을 찾다가 너무 복잡해 져서 차라리 if 구문을 하나 더 넣어서 돌리는 것이 좋을 것 같음
    
    즉, length가 8일때는 index가 3일때 length가 11일 때는 index가 0,2,6일때 조건문을 넣어서
    
    **for 문 돌리는 방법으로 다시 풀어봄.**
    
    ```java
    String result = "";
            if(arr.length == 8) {
                result = "(010)";
    
                for(int i = 0; i < arr.length; i++) {
                    if(i == 3) {
                        result = result + arr[i] + "-";
                    } else {
                        result = result + arr[i];
                    }
                }
            } else { 
                for(int i = 0; i < arr.length; i++) {
                    if(i == 0) {
                        result = "(" + result + arr[i];
                    } else if(i == 2) {
                        result = result + arr[i] + ")";
                    } else if(i == 6) {
                        result = result + arr[i] + "-";
                    }else {
                        result = result + arr[i];
                    }
                }
            }
    ```

<br>
<br>
<br>
<br>

---