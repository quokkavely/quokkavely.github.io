---
layout : single
title : "[디자인패턴] Singleton Pattern"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 싱글톤 패턴 (Singleton Pattern)

싱글톤 패턴은 하나의 클래스에서 단 하나의 인스턴스만 생성되도록 보장하는 설계 패턴이다. 이 패턴은 보통 **데이터베이스 연결 모듈**처럼 비용이 큰 자원을 하나의 인스턴스로 공유하여 사용하는 데 유용하다.

단 하나의 인스턴스를 기반으로 여 모듈에서 공유하며 사용할 수 있어 **인스턴스 생성 비용을 절감**할 수 있는 장점이 있지만, **의존성이 높아질 수 있는 단점**도 있다.

## 싱글톤 패턴의 특징

1. **단일 인스턴스 보장**
    - 하나의 클래스에서 단 하나의 객체만 생성.
2. **글로벌 접근**
    - 생성된 인스턴스를 어디서든 접근 가능.
3. **자원 절약**
    - 데이터베이스 연결 같은 자원 집약적 작업에 효과적.

## 싱글톤 패턴의 구현 (Java)

### 1. Lazy Initialization (중첩 클래스를 사용한 구현)

가장 일반적이고 효율적인 싱글톤 패턴 구현 방법 중 하나는 중첩 클래스를 사용하는 방식이다.

```java
class Singleton {
    // Singleton을 내부 클래스에서 초기화
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    // 외부에서 인스턴스를 가져오는 메서드
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}

public class SingletonExample {
    public static void main(String[] args) {
        Singleton instance1 = Singleton.getInstance();
        Singleton instance2 = Singleton.getInstance();

        // 같은 인스턴스를 가리키는지 확인
        System.out.println(instance1 == instance2); // true
        System.out.println(instance1.hashCode());
        System.out.println(instance2.hashCode());
    }
}

```

**출력 예시**

```arduino
true
12345678
12345678

```

### 2. 데이터베이스 연결 모듈에서의 싱글톤 활용

싱글톤 패턴은 데이터베이스 연결 관리에 자주 사용된다. 아래는 MySQL 데이터베이스 연결을 Java로 구현한 예시다.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

class DatabaseConnection {
    private static DatabaseConnection instance;
    private Connection connection;

    private DatabaseConnection() {
        try {
            String url = "jdbc:mysql://localhost:3306/mydatabase";
            String user = "username";
            String password = "password";
            connection = DriverManager.getConnection(url, user, password);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }

    public Connection getConnection() {
        return connection;
    }
}

public class DatabaseExample {
    public static void main(String[] args) {
        DatabaseConnection db1 = DatabaseConnection.getInstance();
        DatabaseConnection db2 = DatabaseConnection.getInstance();

        System.out.println(db1.getConnection());
        System.out.println(db2.getConnection());
        System.out.println(db1 == db2); // true
    }
}

```

## 싱글톤 패턴의 단점

1. **의존성 증가**
    - 모듈 간 강한 결합이 발생할 수 있다.
    - 테스트 코드 작성이 어려워질 수 있다.
2. **단위 테스트의 어려움**
    - 단일 인스턴스를 공유하기 때문에 테스트 독립성을 유지하기 어려워, TDD(Test Driven Development)에서 걸림돌이 될 수 있다.

## 싱글톤과 의존성 주입 (Dependency Injection)

### 문제 해결 방안

- 의존성 주입(DI)를 사용하면 싱글톤의 단점을 보완할 수 있다.
- 의존성 주입은 **모듈 간의 결합을 느슨하게 만들어**, 테스트 용이성과 유연성을 높인다.

### 의존성 주입의 장점

1. **테스트 용이성**: 모듈 교체가 쉬워짐.
2. **유연성 증가**: 추상화 계층을 통해 구현체 변경이 용이.
3. **디커플링**: 상위 모듈과 하위 모듈 간 결합도 감소.

## 싱글톤 패턴의 활용

1. **데이터베이스 연결**
    - 자주 재사용되는 연결을 하나의 인스턴스로 관리.
2. **캐싱 시스템**
    - 자주 조회되는 데이터를 메모리에 저장하여 성능 향상.
3. **설정 관리**
    - 전역적으로 사용되는 설정 값을 관리.

---

싱글톤 패턴은 자원을 효율적으로 관리하고, 시스템 내에서 일관성을 유지하는 데 중요한 역할을 한다. 그러나 의존성이 높아질 수 있는 단점이 있으므로, 상황에 따라 **의존성 주입** 같은 대안과 병행하여 사용하면 더 효과적이다.

디자인 패턴을 올바르게 이해하고 활용하면, 유지보수성과 확장성이 뛰어난 애플리케이션을 설계할 수 있다.