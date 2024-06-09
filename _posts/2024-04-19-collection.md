---
layout : single
title : "[JAVA] 컬렉션프레임워크, 예외처리"
categories: JAVA-Learn
tag : [JAVA, 실습개념]
toc : true
toc_sticky : true
author_profile: true
---

# 컬렉션프레임웤

1. 컬렉션 : 여러 객체를 모아 놓은 것을 의미
2. 프레임웤 : 표준화, 정형화된 체계적인 프로그램 양식
3. 컬렉션 프레임웤 
    1. 컬렉션(다수의 객체)을 다루기 위한 표준화된 프로그래밍 방식
    2. 컬렉션을 쉽고 편리하게 다룰 수 있는 다양한 클래스를 제공
    3. java.util 패키지에 포함
       
         
       
    
    컬렉션프레임워크의 핵심 인터페이스 : 크게 List와 Set 그리고 Map으로 구분
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/fa1caada-12ae-4766-b3d8-f1d4eb7c7dbd" width =500 />    
        

| List                         | Set                                     | Map                                                          |
| ---------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| 순서가 있는 <br />데이터집합 | 순서를 유지하지 않는<br />데이터의 집합 | key 와 value의 쌍으로<br />이루어진 집합<br />순서 유지하지 않음 |
| 데이터 중복 허용 O           | 데이터 중복허용 X                       | 키는 중복 불가<br />Value는 중복 가능                        |
| 대기자 명단                  | 양의 정수집합<br />소수의 집합          | 우편번호<br />지역번호(전화번호)                             |
| ArrayList                    | Hashset                                 | HashMap                                                      |
| LinkedList                   | TreeSet                                 | TreeMap                                                      |
| Stack                        |                                         | Hashtable                                                    |
| Vector                       |                                         | Properties                                                   |



# List

| 기능 | 리턴 타입 | 메서드 | 설명 |
| --- | --- | --- | --- |
| 객체 추가 | void | add(int index, Object element) | 주어진 인덱스에 객체를 추가 |
|  | boolean | addAll(int index, Collection c) | 주어진 인덱스에 컬렉션을 추가 |
| 수정 | Object | set(int index, Object element) | 주어진 위치에 객체를 저장 |
| 객체 검색 | Object | get(int index) | 주어진 인덱스에 저장된 객체를 반환 |
|  | int | indexOf(Object o) / lastIndexOf(Object o) | 순방향 / 역방향으로 탐색하여 주어진 객체의 위치를 반환 |
|  | ListIterator | listIterator() / listIterator(int index) | List의 객체를 탐색할 수 있는 ListIterator 반환 / 주어진 index부터 탐색할 수 있는 ListIterator 반환 |
|  | List | subList(int fromIndex, int toIndex) | fromIndex부터 toIndex에 있는 객체를 반환 |
| 객체 삭제 | Object | remove(int index) | 주어진 인덱스에 저장된 객체를 삭제하고 삭제된 객체를 반환 |
|  | boolean | remove(Object o) | 주어진 객체를 삭제 |
| 객체 정렬 | void | sort(Comparator c) | 주어진 비교자(comparator)로 List를 정렬 |

## **ArrayList**,

1. 배열로 되어있음.초기 용량은 정하지 않고 초과하면 자동으로 늘어남. 
   
    ArrayList 기본 생성자를 통해 사용 하고 있을 경우를 생각 해보면 element 배열 크기가 0 이기 때문에
    첫 element 를 add 할때 배열의 resize 가 발생하고 배열 크기는 10 으로 설정 된다.
    이후 ArrayList 에 10개의 데이터가 있고 데이터를 추가 하려고 하면 resize 가 발생하여 15 가 된다.
    
2. List 인터페이스를 구현. 컬렉션프레임워크에서 가장 많이 사용﻿

```java
ArrayList<타입 매개변수> 객체명 = new ArrayList<타입 매개변수>(초기 저장 용량)
```

배열(arr)을 ArrayList로 변경할때

>>  new ArrayList<>(Arrays.asList(arr));

장점: 배열은 구조가 간단하고 데이터를 읽는데 걸리는 시간(접근시간)이 짧다

단점 

1) 크기변경불가???

크기변경시 새로운 배열 생성 후 데이터를 복사해야함.

크기변경을 피하기 위해 충분히 큰 배열을 생성하면 메모리가 낭비됨.

2) 비순차적인 데이터의 추가,삭제에 시간이 많이 걸림

데이터를 추가하거나 삭제하기 위해 다른 데이터를 옮겨야함

그러나 순차적인 데이터 추가와 삭제는 빠르다 (끝에서 시작하는) 

## **LinkedList**,

조회 삭제 수정이 빈번할 때 사용

LinkedList의 구조는 각 요소들이 자신과 연결된 다음요소에 대한 주소값과 데이터로 구성되어 있음.

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/df8b616f-7e0d-4032-8058-b16fce90ec7f" width=500/>

그렇기 때문에 중간 데이터를 추가 / 삭제 시 LinkedList가 빠름
but, 데이터의 개수가 많아질수록 데이터를 읽어오는 시간(접근 시간)이 길어짐.

배열의 경우
인덱스가 n인 요소의 값을 얻고자하면
"n번째 데이터의 주소 = 배열의 주소+(n-1)*데이터 타입의 크기"
수식으로 계산하여 간단히 데이터를 읽어 올 수 있지만
LinkedList는 불연속적으로 위치한 각 요소들이 서로 연결되어 처음부터 n번째 데이터 까지 차례대로 따라가야 값을 얻을 수 있음 > 접근시간 길어짐

(ArrayList가 더 나은 경우
+순차적으로 추가, 삭제는 ArrayList가 더 빠름
+마지막 데이터 부터 삭제할 경우 재배치 필요가 없어서 상당히 빠름
+데이터 개수가 변하지 않는다면 )

## **Iterator**

| 리턴타입 | 메서드 | 설명 |
| --- | --- | --- |
| boolean | hasNext() | 읽어올 객체가 남아 있으면 true를 리턴하고, 없으면 false를 리턴 |
| Object | next() | 컬렉션에서 하나의 객체를 읽어옵니다. 이때, next()를 호출하기 전에 hasNext()를 통해 읽어올 다음 요소가 있는지 먼저 확인해야 함 |
| void | remove() | next()를 통해 읽어온 객체를 삭제합니다. next()를 호출한 다음에 remove()를 호출해야 함 |

컬렉션에 저장된 요소들을 순차적으로 읽어오는 역할


1. Iterator 사용

```java
**public class Example1 {
    public static void main(String[] args) {
        LinkedList<String>list=new LinkedList<>();
        list.add("I am");
        list.add("a");
        list.add("sick");
        Iterator<String> iterator= list.iterator();
        while(iterator.hasNext()){
            String str= iterator.next();
            if(str.equals("sick")){
                iterator.remove();
            }
        }
        System.out.println(list);
    }
}
//출력 : [I am, a]**
```

1. Iterator 사용안하고 for 문 돌릴때

```java
public class Example02 {
    public static void main(String[] args) {
        LinkedList<String> list=new LinkedList<>();
        list.add("I am");
        list.add("a");
        list.add("sick");
        for(int i=0; i<list.size();i++){
            if(list.equals("sick")) {
                list.remove();
            }
        }
        System.out.println(list);
    }
}
//출력 : [I am, a]
```

# Set

요소의 중복을 허용하지 않고, 저장 순서를 유지하지 않는 컬렉션

인터페이스로 구현되어 있어 Set하나만 샤옹 못함.

Set을 구현한 대표적인 클래스로 HashSet, TreeSet이 있음.

| 기능 | 리턴 타입 | 메서드 | 설명 |
| --- | --- | --- | --- |
| 객체 추가 | boolean | add(Object o) | 주어진 객체를 추가하고, 성공하면 true를, 중복 객체면 false를 반환합니다. |
| 객체 검색 | boolean | contains(Object o) | 주어진 객체가 Set에 존재하는지 확인합니다. |
|  | boolean | isEmpty() | Set이 비어있는지 확인합니다. |
|  | Iterator | Iterator() | 저장된 객체를 하나씩 읽어오는 반복자를 리턴합니다. |
|  | int | size() | 저장된 전체 객체의 수를 리턴합니다. |
| 객체 삭제 | void | clear() | Set에 저장된 모든 객체를 삭제합니다. |
|  | boolean | remove(Object o) | 주어진 객체를 삭제합니다. |

## **HashSet**,

객체를 저장하기 전 기존에 같은 객체가 있는지 확인 후

객체가 없으면 저장하고 있으면 저장하지 않는다.

1. 중복된 값인지 판단하는 방법
    1. Boolean add(Object o)를 통해 객체 저장
    2. 저장하고자 하는 객체의 int hashCode()를 통해 얻고
    3. Set에 이미 저장되어 있던 모든 객체들의 hashCode()를 통해 얻어냄
    4. boolean equals(Object o)로 비교하여 같은 해시코드가 있는지 검사
    5. equal()에서 true가 리턴되면 Set에 추가 되지 않고> add메서드가 false  리턴
    6. equal()에서 false가 리턴되면서 Set에 추가 > add메서드가 true 리턴
    

# Map

Map - 조금더 명확한 식별자를 사용할때. 

1) List와 달리 입력된 순서를 보장하지 않음(일반적으로)

2) **키(key)와 값(value)으로 구성된 객체**

객체는 Entry, 이 **Entry 객체는 키와 값을 각각 Key 객체와 Value 객체로 저장**

객체이므로 기본형 못들어옴 (wrapper class 가능), 

→참조형만 들어올 수 있음. - 제네릭으로 구현되어 있기 때문

**키는 무조건 하나만 있어야 함.**

**만약 기존에 저장된 키와 같은 키로 값을 저장하면, 기존의 값이 새로운 값으로 대치**

3) HashMap, HashTable, TreeMap, Properties 등의 구현체가 있음

4)  HashMap을 가장 많이 사용하게 될 것.

5) Hash는 hash함수를 거쳐서 저장하게 됨.

| 기능 | 리턴 타입 | 메서드 | 설명 |
| --- | --- | --- | --- |
| 객체 추가 | Object | put(Object key, Object value) | 주어진 키로 값을 저장합니다. 해당 키가 새로운 키일 경우 null을 리턴하지만, 같은 키가 있으면 기존의 값을 대체하고 대체되기 이전의 값을 리턴합니다. |
| 객체 검색 | boolean | containsKey(Object key) | 주어진 키가 있으면 true, 없으면 false를 리턴합니다. |
|  | boolean | containsValue(Object value) | 주어진 값이 있으면 true, 없으면 false를 리턴합니다. |
|  | Set | entrySet() | 키와 값의 쌍으로 구성된 모든 Map.Entry 객체를 Set에 담아서 리턴합니다. |
|  | Object | get(Object key) | 주어진 키에 해당하는 값을 리턴합니다. |
|  | boolean | isEmpty() | 컬렉션이 비어 있는지 확인합니다. |
|  | Set | keySet() | 모든 키를 Set 객체에 담아서 리턴합니다. |
|  | int | size() | 저장된 Entry 객체의 총 갯수를 리턴합니다. |
|  | Collection | values() | 저장된 모든 값을 Collection에 담아서 리턴합니다. |
| 객체 삭제 | void | clear() | 모든 Map.Entry(키와 값)을 삭제합니다. |
|  | Object | remove(Object key) | 주어진 키와 일치하는 Map.Entry를 삭제하고 값을 리턴합니다. >>>리무브를쓰면 키를 찾아서 안에 있는 Value까지 다지워버림                  |

## HashMap

HashMap은 해시 함수를 통해 '키'와 '값'이 저장되는 위치를 결정하므로, 사용자는 그 위치를 알 수 없고, 

**삽입되는 순서와 위치 또한 관계가 없음.**

이렇게, HashMap은 이름 그대로 해싱(Hashing)을 사용하기 때문에 **많은 양의 데이터를 검색하는 데 있어서 뛰어난 성능**을 보임

또한, HashMap의 개별 요소가 되는 Entry 객체는 Map 인터페이스의 내부 인터페이스인 Entry 인터페이스를 구현

[Map.Entry 인터페이스의 메서드]

| 리턴 타입 | 메서드 | 설명 |
| --- | --- | --- |
| boolean | equals(Object o) | 동일한 Entry 객체인지 비교 |
| Object | getKey() | Entry 객체의 Key 객체를 반환 |
| Object | getValue() | Entry 객체의 Value 객체를 반환 |
| int | hashCode() | Entry 객체의 해시코드를 반환 |
| Object | setValue(Object value) | Entry 객체의 Value 객체를 인자로 전달한 value 객체로 바꿈 |

```java
HashMap<String, Integer> hashmap = new HashMap<>();
```

```java
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;

public class Example03 {
    public static void main(String[] args) {
        HashMap<String,Integer>map=new HashMap<>();
        map.put("신짱구",5);
        map.put("신형만",35);
        map.put("봉미선",29);
        map.put("신짱아",0);
        map.put("김철수",5);
        map.put("한유리",5);
        map.put("이맹구",5);
        map.put("유이슬",20);

        System.out.println("총 entry의 수 :" +map.size());
        System.out.println("김철수 :"+map.get("김철수"));

        Set<String>keySet = map.keySet(); //순회하기 위해 map의 key를 Set에 담아서 리턴
        Iterator<String>keyIterator= keySet.iterator();
        while(keyIterator.hasNext()){
            String key = keyIterator.next();
            Integer value = map.get(key);
            System.out.println(key+":"+value);
        }

        map.remove("한유리");
        System.out.println("총 entry 의 수 :"+ map.size());
        Set<Map.Entry<String,Integer>>entrySet=map.entrySet(); //순회하기위해 entry객체를 요소로 가지는 set설정
        Iterator<Map.Entry<String,Integer>>entryIterator= entrySet.iterator();
        while(entryIterator.hasNext()){
            Map.Entry<String,Integer>entry = entryIterator.next();
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println(key+":"+value);
        }
        map.clear();
     }
}
/*출력:
총 entry의 수 :8
김철수 :5
신형만:35
한유리:5
이맹구:5
신짱아:0
봉미선:29
유이슬:20
신짱구:5
김철수:5
총 entry 의 수 :7
신형만:35
이맹구:5
신짱아:0
봉미선:29
유이슬:20
신짱구:5
김철수:5
*/
```

Map은 인덱스가 없어서 Map자체로 순회할 수 없음.

=Map은 키와 값을 쌍으로 저장하기 때문에 `iterator()`를 직접 호출 불가

그 대신 **`keySet()`이나 `entrySet()` 메서드를 이용해 Set 형태로 반환된 컬렉션에 `iterator()`를 호출하여 반복자를 만든 후, 반복자를 통해 순회가능. =for문으로도 가능**

아래는 Iterator대신 향상된 for문으로 돈 것.

1)

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/0e5a3c47-2972-437b-9e7d-6340fd6290fd" width=500/>

2)

​	<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d81e8216-82f1-4eed-a596-4e5ca3a90c4c" width=500/>

<br/>

<br/>

<br/>

<br/>

#  예외처리

<br/>

<br/>

## 프로그램 오류

### 컴파일 에러

1. 빨간줄이 뜨는 경우는 컴파일 하다가 오류 → 클래스를 만들지도 않고 컴파일러가  잡아준다.
2. 에러 만났을 때 내가 작성한 코드에서 어디가 잘못되었는지 확인하고 에러 메세지로 확인할 수 있어야 함.= 어디를 수정하면 되겠구나 모르겠으면 검색하고 확인할 줄 알아야함. <<꼭 찾아봐야 함.💫>>
3. null point exception 많이 만나게 되는데 이것때문에 스트레스 받아서 코틀린 넘어가는 경우가 많음.  코틀린은 notnullable.

### 런타임 에러

클래스파일로 변경되고 실행될 때까지 모르다가 jvm이 만나게 되면 터지는 것,

```java
public class RuntimeErrorTest {

    public static void main(String[] args) {
        System.out.println(4 * 4);
        System.out.println(4 / 0); // 예외 발생
    }
}

//출력값
16
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at RuntimeErrorTest.main(RuntimeErrorTest.java:5)
```

[런타임에러] 상기 코드는 빨간 줄이 뜨지도 않음. 

1. 에러와 예외

2. **에러**: 프로그램 코드에 의해서 수습될 수 없는 심각한 오류

   1. 가장 많이 만나는 에러는 = stack overflow  : 메모리를 효율적으로 사용하고 있지 못해서 그런 것.
   2. 물리적으로 메모리 부족,
   3. 예외 처리로도 극복할 수 없는 경우, 
   4. 사용자가 극복하기 힘든 경우가 대부분! 

3. **예외**: 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류

   >프로그램의 비정상 적인 종류를 막을 수 있다.
   >
   >

   <br/>

   <br/>

   

***Stack overflow*

jvm 내 stack이라는 저장소가 쌓이게 됨. 

=다쓰면 없어져야하는데 넘칠 경우 떨어져나오게 됨.

 할당할 수 있는 데이터보다 많이 들어오게 되면 발생 

ex)재귀 사용 시 탈출 조건 만들지 못하면 발생 - 일단 종료됨.



### 예외클래스의 상속계층도💫💫💫 - 알고 있어야 함.

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/afd238e3-9b51-406a-93d9-28c6c9772775" width=500/>

1. 예외 클래스는 두 그룹으로 나눠질 수 있음.

   1) Exception클래스와 그 하위클래스들 - 휴먼에러가 많음. checked예외

   2) Runtime 클래스와 그 하위클래스들. - unchecked 예외

2. Errors - 해결 불가능 한 것. 상속받아서 만듬.

3. Runtime Exceptions : 실행될 때 까지 모르는 것. ex )0으로 나누려고 할 때

4. Other Exceptions =Checked Exception : Runtime Exceptions이외의 모든 것

5. 서비스를 만들면서 사용자 지정 예외를 만들 수 있음

### try - catch문

```java
try {
    // 예외가 발생할 가능성이 있는 코드를 삽입
}
catch (ExceptionType1 e1) {
    // ExceptionType1 유형의 예외 발생 시 실행할 코드
}
catch (ExceptionType2 e2) {
    // ExceptionType2 유형의 예외 발생 시 실행할 코드
}
finally {
    // finally 블록은 꼭 포함되어야 하는 문법은 아님
    // 만약 포함 되었다면 예외 발생 여부와 상관없이 항상 실행
}
```

*finally :예외가 발생했든 안했던 항상 실행되는 것.*

예외 미발생시 try - finally 실행됨.

try 또는 catch에서 return문을 만나게 되고 finally문은 실행됨.

***try-catch  문 예제 1)***

```java
public class RuntimeExceptionTest {

    public static void main(String[] args) {
        System.out.println("[소문자 알파벳을 대문자로 출력하는 프로그램]");
        printMyName("abc"); // (1)
        printMyName(null); // (2) 넘겨주는 매개변수가 null인 경우 NullPointerException 발생
        System.out.println("[프로그램 종료]");
    }

    static void printMyName(String str) {
        String upperCaseAlphabet = str.toUpperCase();
        System.out.println(upperCaseAlphabet);
    }

}

//출력값
[소문자 알파벳을 대문자로 출력하는 프로그램]
ABC //(3) 정상 실행
Exception in thread "main" java.lang.NullPointerException // (4) 예외 발생!
	at RuntimeExceptionTest.printMyName(RuntimeExceptionTest.java:11)
	at RuntimeExceptionTest.main(RuntimeExceptionTest.java:6)
```

- null은 String에 들어올 수 있어서  코드 작성 시 빨간 줄이 뜨지 않게 됨, ⇒ 런타임 에러 

- toUpperCase 실행 시 프로그램이 터지게 됨. 그 이후 프로그램 종료도 출력 될 수 없음.

  <br/>

  <br/>

***try-catch  문 예제 1→예외처리)***

상기 코드를 try-catch문으로 예외 처리하여 아래 코드로 작성된 것.

```java
public class RuntimeExceptionTest {

    public static void main(String[] args) {

        try {
            System.out.println("[소문자 알파벳을 대문자로 출력하는 프로그램]");
            printMyName(null); // (1) 예외 발생
            printMyName("abc"); // 이 코드는 실행되지 않고 catch 문으로 이동
        }
        catch (ArithmeticException e) {
            System.out.println("ArithmeticException 발생!"); // (2) 첫 번째 catch문
        }
        catch (NullPointerException e) { // (3) 두 번째 catch문
            System.out.println("NullPointerException 발생!");
            System.out.println("e.getMessage: " + e.getMessage()); // (4) 예외 정보를 얻는 방법 - 1
            System.out.println("e.toString: " + e.toString()); // (4) 예외 정보를 얻는 방법 - 2
            e.printStackTrace(); // (4) 예외 정보를 얻는 방법 - 3
        }
        finally {
            System.out.println("[프로그램 종료]"); // (5) finally문
        }
    }

    static void printMyName(String str) {
        String upperCaseAlphabet = str.toUpperCase();
        System.out.println(upperCaseAlphabet);
    }
}

// 출력값
[소문자 알파벳을 대문자로 출력하는 프로그램]
NullPointerException 발생!
e.getMessage: null
e.toString: java.lang.NullPointerException
[프로그램 종료]
java.lang.NullPointerException
	at RuntimeExceptionTest.printMyName(RuntimeExceptionTest.java:20)
	at RuntimeExceptionTest.main(RuntimeExceptionTest.java:7)
```

1. e.printStackTrace

   1. 예외 발생 당시의 호출스택에 있었던 메서드의 정보와 예외메세지를 화면에 출력
   2. e.printStackTrace() 주석 처리 할 경우
      1. 주석처리하면 아래와 같이 출력됨.

   <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/486b5c48-1e01-46b2-8c27-f110550f53ca" width=500/>

2. ArithmeticException : 산술연산과정에서 오류가 있을 때 발생하는 예외 = 정수는 0으로 나누는 것이 금지되어 있기 때문에 발생, but 실수는 0으로 나누는 것이 금지 있지 않아 예외 발생하지 않음.

3.  NullPointerException >RuntimeException을 상속

    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/36baa1fe-a2bb-4c2f-8a51-b329106dba40" width=500/>

4. getMessage()

   1. 발생한 예외 클래스의 인스턴스에 저장된 메세지를 얻을 수 있다.

5. catch문에 Exception e로 예외처리 할 경우 컴파일 에러 발생 

   <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/cf0d505a-c4b0-4d61-9ff1-8103862f0bf7" width=500/>

   1. 이유 : Exception이 상위클래스이기 때문 (Null~, Arithmetic~,등)

      <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/8134ca29-363e-4056-8a7f-9e8bd9d9685f" width=500/>

      Exception은 모든 예외클래스의 상위클래스이므로 catch문의 제일 마지막에 등장하면 오류 안뜸> 모든 예외 처리 가능

***try-catch  문 예제 2)***

```java
public class Exception02 {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Enter a number :");
        String input = scanner.nextLine();
        try{
            int number= Integer.parseInt(input);
            System.out.println("You entered: "+number);
        }catch (NumberFormatException e){
            System.out.println("Invalid input! Please enter a valid number");
        }finally {
            scanner.close();
        }

    }
}
/* 
입력창에 숫자 입력시 
Enter a number :
5
You entered: 5
------------------------
입력 창에 문자 입력시 
Enter a number :
a
Invalid input! Please enter a valid number
*/
```

1. 계산기에서 했던 숫자나 문자입력시 예외처리와 상기코드  모두 구현할 줄 알아야함.
2. 예외처리 시 숫자만 입력하라고 할 경우 double타입으로 받을 시 숫자검증 그리고
3. dot이 있을 경우 dot은 하나만 있어야 하는 것을 다 검증하기가 쉽지 않음.
4. 정규표현식으로 연산자를 검증할 수 있지만, ++ 두개 씩 들어올 경우는 못거름.

### 예외전가

나를 호출한 곳으로 보내버림 - 이때 사용하는 것이 throws

throws 뒤에 붙은 예외를 한정해두고 그 예외가 발생했을 때만 발생한 곳으로 보내버림.

근데 예외 처리할 코드가 없다면 catch문 갔다가 터지게 됨.

### [throws : 메서드 뒤에 존재, 뒤에 만난 예외클래스를 만났을 경우 호출한 곳으로 보내

```java
//메서드 시그니처에 작성
반환타입 메서드명(매개변수, ...) throws 예외클래스1, 예외클래스2, ... {
	...생략...
}

//example
void ExampleMethod()throws Exception{
}
```

 <br/>

<br/>

**throws 예제**

```java
public class ThrowExceptionTest {

    public static void main(String[] args) {
        try {
            throwException(); 
        } catch (ClassNotFoundException e) {
            System.out.println(e.getMessage());
        }
    }

    static void throwException() throws ClassNotFoundException, NullPointerException {
        Class.forName("java.lang.StringX");
    }
}

//출력값
java.lang.String
```

---

ClassNotFoundException 는 클래스라서 클래스를 객체화 시켜 인스턴스를 리턴.

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/15e95d81-b691-45c6-bb66-ae77934bdf5f" width=500/>

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9aa1aa1d-23a6-4a08-979f-15cdf5507636" width=500/>

생성자 생성후 오버로딩 되어있음.

super에게 상속되어 있어서 상위 클래스로 타고 타고 넘어가면 

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/efb8c952-aef5-48fb-8a1f-33c4873dc5ea" width=500/>

어디서 발생했는지 볼 수 있음.

---

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/441bcbb9-0a14-4da0-b6fd-8190e5b44e97" width=500/>

void는 문제없이 실행되었을때 void를 반환 할거야라고 생각하기.

throws는 문제가 발생하면  ClassNotFoundException, NullPointerException여기로 보낼거야

### throw : 의도적으로 예외 만드는 방법

```java
public class ExceptionTest {

    public static void main(String[] args) {
        try {
            Exception intendedException = new Exception("의도된 예외 만들기");
            throw intendedException;  //throws에 s가 하나 없음
        } catch (Exception e) {
            System.out.println("고의로 예외 발생시키기 성공!");
        }
    }

}

//출력값
고의로 예외 발생시키기 성공!
```