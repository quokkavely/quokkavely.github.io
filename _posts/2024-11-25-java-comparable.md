---
layout : single
title : "[JAVA] Comparable과 Comparator"
categories: JAVA-Learn
tag : [JAVA, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


# Java의 Comparable과 Comparator

이번에 코드트리로 DP(동적 계획법)를 공부하려고 결제했는데, 객체 정렬을 사용하는 문제가 나왔다. 평소 깊게 생각하지 않았던 `Comparable`과 `Comparator`에 대해 다뤄야 했고, 이참에 자바의 정석을 다시 보며 정리해 보았다. 두 인터페이스는 자바에서 객체 정렬을 지원하는 중요한 구성요소로, 정렬 기준을 설정하는 방식이 조금 다르다.

Java의 `Arrays.sort()`는 내부적으로 `Comparable` 또는 `Comparator` 인터페이스를 구현한 객체를 활용하여 정렬한다.

이 두 인터페이스는 정렬 기준을 정의하기 위해 설계된 인터페이스로, 컬렉션이나 배열과 같은 데이터 구조를 정리하는 데 사용된다.

## **Comparable**

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

- `Comparable` 인터페이스는 **클래스 내부에 "기본 정렬 기준"을 정의**할 때 사용한다.
- 같은 타입의 인스턴스끼리 비교할 수 있도록 만들어 준다.
- 기본적으로 오름차순(작은 값 → 큰 값)으로 정렬되도록 구현되어 있다.
- Java의 `Integer`, `String`, `Date`와 같은 클래스는 모두 `Comparable`을 구현하고 있다.
- `compareTo()` 메서드 반환값
    - **음수**: 현재 객체(this)가 비교 대상보다 작을 경우.
    - **0**: 현재 객체와 비교 대상이 같을 경우.
    - **양수**: 현재 객체가 비교 대상보다 클 경우.

### 예제 :  기본 정렬 기준 정의

```java
class Student implements Comparable<Student> {
    String name;
    int score;

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    @Override
    public int compareTo(Student other) {
        return this.score - other.score; // 점수를 기준으로 오름차순 정렬
    }
}

// 사용
Student[] students = {
    new Student("Alice", 85),
    new Student("Bob", 90),
    new Student("Charlie", 80)
};

Arrays.sort(students);
```

## **Comparator**

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
    boolean equals(Object obj);
}
```

- `Comparator` 인터페이스는 **클래스 외부에서 "정렬 기준"을 정의**할 때 사용한다.
- `Comparable`과 달리, 한 클래스에 대해 여러 정렬 기준을 유연하게 설정 가능하다.
- 예를 들어, 학생의 이름, 점수, 나이 등 다양한 기준으로 정렬하고 싶을 때 적합하다.\
- `compare()` 메서드 반환값
    - **음수**: 첫 번째 객체(o1)가 두 번째 객체(o2)보다 작을 경우.
    - **0**: 두 객체가 같을 경우.
    - **양수**: 첫 번째 객체(o1)가 두 번째 객체(o2)보다 클 경우.

### 예제: 여러 기준으로 정렬

```java
class NameComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return s1.name.compareTo(s2.name); // 이름 기준 오름차순 정렬
    }
}

class ScoreComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return s2.score - s1.score; // 점수 기준 내림차순 정렬
    }
}

// 사용
Arrays.sort(students, new NameComparator()); // 이름 기준 정렬
Arrays.sort(students, new ScoreComparator()); // 점수 기준 정렬

```

## **Comparable vs Comparator 비교**

| 특징 | Comparable | Comparator |
| --- | --- | --- |
| **역할** | 클래스 내부에 "기본 정렬 기준" 정의 | 외부에서 다양한 정렬 기준 정의 |
| **메서드** | `compareTo(T o)` | `compare(T o1, T o2)` |
| **사용 위치** | 정렬 대상 클래스 내부에서 구현 | 클래스 외부에서 별도로 정의 |
| **정렬 기준 수** | 한 가지 정렬 기준 | 여러 정렬 기준 |
| **사용 예** | `Arrays.sort(arr)` | `Arrays.sort(arr, comparator)` |
| **기존 클래스 수정 필요 여부** | 필요 (정렬 기준 변경 시 클래스 수정) | 불필요 (클래스를 수정하지 않아도 정렬 기준 추가 가능) |

## **문자열 정렬 예제**

자바의 정석 예제 11-19

```java
import java.util.*;

class ComparatorEx {
    public static void main(String[] args) {
        String[] strArr = {"cat", "Dog", "lion", "tiger"};

        // 기본 정렬 (Comparable 사용)
        Arrays.sort(strArr);
        System.out.println("Default Sort: " + Arrays.toString(strArr));

        // 대소문자 구분 없는 정렬
        Arrays.sort(strArr, String.CASE_INSENSITIVE_ORDER);
        System.out.println("Case-Insensitive Sort: " + Arrays.toString(strArr));

        // 역순 정렬 (Comparator 사용)
        Arrays.sort(strArr, new DescendingOrder());
        System.out.println("Descending Sort: " + Arrays.toString(strArr));
    }
}

class DescendingOrder implements Comparator<String> {
    @Override
    public int compare(String s1, String s2) {
        return s2.compareTo(s1); // 역순 정렬
    }
}
```

## 문제 풀면서 헷갈렸던 부분

### **Q. `Arrays.sort(arr, fromIndex, toIndex)`**

- `fromIndex`: 정렬을 시작할 인덱스.
- `toIndex`: 정렬을 멈출 인덱스 **바로 다음 위치**.
- 예를 들어, `Arrays.sort(arr, 0, 5)`는 배열의 인덱스 `0`부터 `4`까지를 정렬한다.

---

코딩테스트 준비하면서 하나의 플랫폼에만 익숙하다보니, 비슷한 유형의 문제를 많이 풀었던 것 같은데 이번에 코드트리로 넘어오면서 자바에 대해 부족했던 부분을 채울 수 있어서 좋은 것 같다. 허점이 마구 드러나는듯… 그래도 채워나가야지~ Comparable이랑 Comparator 오버라이딩해서 구현도 하게 되었다 이제 !