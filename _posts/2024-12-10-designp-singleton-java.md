---
layout : single
title : "[디자인패턴] Singleton Pattern - java"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# Java -Singleton

보통의 클래스는  하나의 클래스를 기반으로 여러 개의 인스턴스를 만들 수 있다. 그러나

싱글톤 패턴은 **단 하나의 인스턴스만 생성**하여 애플리케이션 전역에서 공유하도록 설계된 패턴이다.

**멀티스레드 환경에서의 안전성**과 같은 실질적인 문제 해결 능력을 키울 수 있다.

 Java는 **싱글톤 패턴을 구현하기에 적합한 언어적 특성을 제공한다.**

- **클래스 로딩 특성**
    - Java는 클래스 로드 시점에 한 번만 `static` 초기화가 이루어진다. 이 특성을 활용하면 싱글톤 패턴을 구현하기 쉽다.
    - 특히, **Lazy Holder 방식**이나 **정적 멤버 초기화 방식**은 Java의 클래스 로딩 동작을 기반으로 설계된다.
- **Enum 지원**
    - Java의 `enum`은 기본적으로 싱글톤 패턴을 지원하도록 설계되어 있다. 하나의 `enum` 상수는 애플리케이션 전역에서 유일하게 유지되며, JVM이 생성과 스레드 안전성을 보장한다.
- **언어적 안전성**
    - Java는 멀티스레드 환경에서 동기화와 메모리 가시성 문제를 해결하기 위해 `synchronized` 키워드, `volatile` 키워드 등을 제공한다.
    - 이러한 기능은 멀티스레드 환경에서도 안전한 싱글톤 패턴 구현을 돕는다

### 1. 단순한 메서드 호출

가장 기본적인 싱글톤 패턴 구현 방법으로, 객체를 생성하기 전에 존재 여부를 확인하고 없으면 생성한다.

그러나 **멀티스레드 환경**에서는 두 개 이상의 인스턴스가 생성될 가능성이 있어 실무에서 권장되지 않는다.

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // private 생성자
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```

만약 이렇게 실행했을 경우 여러 개의 스레드가 동작할  경우에 동시성 문제가 발생하게 된다. 그 문제를 해결하기 위해 2번과 같은 방법으로 해결 할 수 있다.

### 2. `synchronized` 키워드를 활용한 방법

**동기화**를 통해 멀티스레드 환경에서 안전성을 확보할 수 있지만 **getInstance()** 메서드 호출 시마다 lock이 걸리기 때문에 성능 저하가 발생할 수 있다.

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```

### 3. 정적 필드

싱글톤 인스턴스를 **정적 멤버로 선언**하고, 클래스가 로드될 때 초기화하는 방법이다.

초기화가 빠르고 간단하지만, 인스턴스를 필요로 하지 않아도 **클래스 로드 시점에 메모리에 할당**된다.

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

```

### 4. 정적 블록을 활용한 방법

정적 블록을 사용하여 즉시 초기화를 수행한다. 복잡한 초기화 로직이 필요한 경우 유용하다.

하지만 필요하지 않은 경우에도 초기화가 진행되는 문제는 동일하다.

```java
public class Singleton {
    private static Singleton instance;

    static {
        instance = new Singleton();
    }

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}

```

### 5. 중첩 클래스 (Lazy Holder)

중첩 클래스(Lazy Holder)를 활용한 방법은 가장 널리 쓰이는 방식 중 하나

`getInstance()` 메서드가 호출될 때만 중첩 클래스가 로드되어 인스턴스를 생성한다.

```java
public class Singleton {
    private Singleton() {}

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

이 방법은 **클래스 로드 시점에 초기화하지 않으므로 메모리 효율적**이며, 멀티스레드 환경에서도 안전하다.

### 6. 이중 확인 잠금 (Double Checked Locking)

이중 확인 잠금(DCL)은 성능과 스레드 안전성을 모두 고려한 방법

먼저 인스턴스 생성 여부를 확인하고, 필요 시에만 동기화를 수행한다. **volatile** 키워드를 사용하여 메모리 가시성 문제를 해결한다.( = 한 스레드에서 `volatile` 변수를 변경하면, 다른 모든 스레드에서 즉시 변경된 값을 읽을 수 있다)

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

<details>
<summary> volatile 키워드 참고 </summary>
<div markdown="1">

메모리 구조에는 메인 메모리(RAM) 위에 캐시 메모리가 있다.  java에서는 변수를 메인 메모리에서 가져온느 것이 아니라 성능 최적화를 위해 캐시 메모리를 기반으로 가져오게 된다.

- **일반 변수**: 각 스레드는 CPU의 캐시에 변수를 저장하고 이를 참조할 수 있다. 이로 인해 다른 스레드에서 변수 값을 변경하더라도, 메인 메모리(Main Memory)에 반영되지 않으면 다른 스레드가 변경 사항을 감지하지 못할 수 있다.
- **`volatile` 변수**: 스레드가 항상 변수의 값을 **메인 메모리에서 읽고, 메인 메모리에 직접 쓰도록 강제**한다.
    - 즉, 캐시를 사용하지 않고 항상 최신 값을 보장한다.

</div>
</details>

### 7. Enum을 활용한 방법

열거형(enum)을 사용하여 싱글톤을 구현하면 기본적으로 스레드 안전성이 보장되며, 인스턴스가 여러 번 생성되지 않는다.

Joshua Bloch(이펙티브 자바 저자)가 추천하는 방법이다.

```java
public enum Singleton {
    INSTANCE;

    public void performAction() {
        // 필요한 동작 구현
    }
}
```

이 방법은 간결하면서도 안전하며, **역직렬화** 시에도 별도의 조치 없이 인스턴스의 유일성을 보장한다.

---

싱글톤 패턴은 객체의 생성을 제어하고 자원을 효율적으로 관리하는 데 유용하지만 **멀티스레드 환경**이나 **테스트의 독립성**이 중요한 경우 주의해서 설계해야 한다.

위 방법 중 중첩클래스가 가장 많이 사용되며, 코드의 간결성과 성능 측면에서 효율적이며 Joshua Bloch가 추천하는 enum 방식은 구현이 간단하고 안정적이어서 구현하기 가장 좋은 방법이다.