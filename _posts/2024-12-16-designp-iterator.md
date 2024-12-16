---
layout : single
title : "[디자인패턴] Factory Pattern"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 이터레이터 패턴(Iterator Pattern)

**이터레이터 패턴**은 자료 구조 내부의 요소들을 **순차적으로 접근**할 수 있는 방법을 제공하는 디자인 패턴이다. 이 패턴은 서로 다른 자료 구조(예: 배열, 리스트, 맵 등)를 동일한 인터페이스로 탐색할 수 있도록 도와준다. 이를 통해 자료 구조 내부 구현 방식에 관계없이 동일한 방식으로 접근이 가능하다.

## 이터레이터 패턴의 핵심 개념

1. **컨테이너(Container)**
    - 동일한 요소를 담고 있는 집합, 예를 들어 배열, 리스트, 맵 등이 컨테이너에 해당한다.
2. **이터레이터(Iterator)**
    - 컨테이너 내부의 요소를 순차적으로 접근하는 데 사용되는 객체.
    - 내부적으로 복잡한 탐색 로직이 있어도, 외부에서는 단순히 `hasNext()`와 `next()` 메서드만으로 순회 가능하다.

## 이터레이터 패턴의 장점

1. **일관된 인터페이스 제공**
    - 서로 다른 자료 구조를 동일한 방식으로 순회할 수 있다.
2. **캡슐화**
    - 컨테이너의 내부 구조를 숨기고, 이터레이터를 통해서만 요소에 접근한다.
3. **유연성**
    - 자료 구조를 변경하더라도, 이터레이터 인터페이스를 유지하면 코드의 다른 부분에 영향을 주지 않는다.

## 이터레이터 패턴의 구조

이터레이터 패턴은 일반적으로 다음 세 가지 요소로 구성된다.

1. **Iterator 인터페이스**
    - `hasNext()`: 다음 요소가 있는지 확인.
    - `next()`: 다음 요소를 반환.
2. **Concrete Iterator(구체적인 이터레이터)**
    - Iterator 인터페이스를 구현하고, 실제 데이터 탐색 로직을 처리.
3. **Aggregate(컨테이너)**
    - 자료 구조를 나타내며, 이터레이터 객체를 반환하는 `createIterator()` 메서드 제공.

## 자바로 구현: 커스텀 컬렉션 순회

아래는 이터레이터 패턴을 활용해 커스텀 컬렉션을 순회하는 예제이다.

### 1. Iterator 인터페이스 정의

```java
public interface Iterator<T> {
    boolean hasNext(); // 다음 요소가 있는지 확인
    T next(); // 다음 요소 반환
}
```

### 2. Concrete Iterator 구현

```java
public class CustomIterator<T> implements Iterator<T> {
    private final T[] items; // 순회할 데이터
    private int position = 0; // 현재 위치

    public CustomIterator(T[] items) {
        this.items = items;
    }

    @Override
    public boolean hasNext() {
        return position < items.length; // 배열 끝에 도달했는지 확인
    }

    @Override
    public T next() {
        if (!hasNext()) {
            throw new IllegalStateException("No more elements");
        }
        return items[position++]; // 현재 요소 반환 후, 포인터 이동
    }

```

### 3. Aggregate(컨테이너) 정의

```java
public class CustomCollection<T> {
    private final T[] items;

    public CustomCollection(T[] items) {
        this.items = items;
    }

    public Iterator<T> createIterator() {
        return new CustomIterator<>(items); // 이터레이터 반환
    }
}
```

### 4. 클라이언트 코드

이터레이터를 사용하여 컨테이너 내부 데이터를 순회한다.

```java
public class Main {
    public static void main(String[] args) {
        String[] data = {"A", "B", "C", "D"}; // 테스트 데이터
        CustomCollection<String> collection = new CustomCollection<>(data);

        Iterator<String> iterator = collection.createIterator();

        while (iterator.hasNext()) {
            System.out.println(iterator.next()); // A, B, C, D 출력
        }
    }
}
```

## 기존 자바 컬렉션과 이터레이터

자바에서 이터레이터 패턴은 이미 내장된 `java.util.Iterator`로 제공된다. 아래는 자바 컬렉션 프레임워크에서 이터레이터를 사용하는 예제이다.

```java
import java.util.ArrayList;
import java.util.Iterator;

public class Main {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Java");
        list.add("Python");
        list.add("C++");

        Iterator<String> iterator = list.iterator(); // 내장 이터레이터 생성

        while (iterator.hasNext()) {
            System.out.println(iterator.next()); // Java, Python, C++ 출력
        }
    }
}
```

---

## 이터레이터 패턴의 활용 사례

1. **자바 컬렉션 프레임워크**
    - `ArrayList`, `HashSet`, `TreeMap` 등에서 이터레이터를 통해 데이터 순회.
2. **데이터베이스 커서(Cursor)**
    - 데이터베이스의 결과 집합을 순차적으로 접근할 때 사용.
3. **파일 시스템 순회**
    - 디렉토리 내 파일 및 하위 디렉토리를 탐색할 때 활용.

---

이터레이터 패턴은 자료구조 내부에 반드시 iterator속성이 존재하지 않더라도 적용 가능하다.

이터레이터 인터페이스를 별도로 구현하면,**기존 자료구조의 내부 구현 방식에 상관없이 통일된 방식으로 순회**할 수 있다.
이터레이터 패턴의 강점은 **자료구조와 상관없이 일관된 인터페이스를 제공**하여 코드 재사용성과 확장성을 높이는 데 있다.