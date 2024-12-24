---
layout : single
title : "[디자인패턴] Proxy Pattern"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## 프록시 패턴 (Proxy Pattern)

### **프록시 패턴이란?**

- 프록시 패턴(Proxy Pattern)은 특정 객체에 접근하기 전, 중간에 대리 객체(Proxy)를 두어 접근을 제어하는 디자인 패턴
- **프록시(Proxy)**: 원래 객체에 대한 대리인 역할을 하며, 요청을 중간에서 가로채거나 처리 로직을 추가.
- **실제 객체(Real Subject)**: 클라이언트가 최종적으로 접근하려는 객체.
- **사용 사례**
    - **보안 강화**: 민감한 객체에 대한 접근 제어.
    - **성능 최적화**: 캐싱이나 지연 로딩(Lazy Loading)을 통한 성능 향상.
    - **로깅 및 모니터링**: 객체의 요청 및 응답을 기록.

### **프록시 패턴의 종류**

1. **가상 프록시 (Virtual Proxy)**
    - 실제 객체 생성 비용이 클 때, 필요할 때만 객체를 생성
    - ex) 대용량 이미지 로딩
2. **보호 프록시 (Protection Proxy)**
    - 접근 권한을 제어
    - ex) 관리자 계정만 데이터 수정 허용
3. **캐싱 프록시 (Caching Proxy)**
    - 동일한 요청에 대해 캐시된 응답을 반환
    - ex) CDN(Content Delivery Network)
4. **원격 프록시 (Remote Proxy)**
    - 원격 서버의 객체를 로컬에서 접근하듯이 처리
    - ex) RMI(Remote Method Invocation)

### **예제**

아래는 프록시 패턴을 사용하여 데이터베이스 연결을 캐싱 및 지연 로딩하는 예제

```java
// Real Subject: 실제 데이터베이스 연결
interface Database {
    void connect();
    String fetchData();
}

// Concrete Real Subject: 실제 데이터베이스 구현
class RealDatabase implements Database {
    private String data;

    public RealDatabase() {
        // 데이터베이스 연결 시 리소스를 소모하는 작업
        System.out.println("RealDatabase: Connecting to the database...");
        this.data = "Database Data";
    }

    @Override
    public void connect() {
        System.out.println("RealDatabase: Connected.");
    }

    @Override
    public String fetchData() {
        return data;
    }
}

// Proxy: 데이터베이스 프록시
class DatabaseProxy implements Database {
    private RealDatabase realDatabase;
    private String cachedData;

    @Override
    public void connect() {
        if (realDatabase == null) {
            System.out.println("DatabaseProxy: Establishing a new connection...");
            realDatabase = new RealDatabase();
        } else {
            System.out.println("DatabaseProxy: Using cached connection.");
        }
    }

    @Override
    public String fetchData() {
        if (cachedData == null) {
            System.out.println("DatabaseProxy: Fetching data from the real database...");
            if (realDatabase == null) {
                connect();
            }
            cachedData = realDatabase.fetchData();
        } else {
            System.out.println("DatabaseProxy: Returning cached data.");
        }
        return cachedData;
    }
}

// Client: 클라이언트 코드
public class ProxyPatternExample {
    public static void main(String[] args) {
        Database database = new DatabaseProxy();

        System.out.println("First request:");
        database.connect();
        System.out.println("Data: " + database.fetchData());

        System.out.println("\nSecond request:");
        System.out.println("Data: " + database.fetchData());
    }
}

```

- **실행 결과**
    
    ```
    First request:
    DatabaseProxy: Establishing a new connection...
    RealDatabase: Connecting to the database...
    RealDatabase: Connected.
    DatabaseProxy: Fetching data from the real database...
    Data: Database Data
    
    Second request:
    DatabaseProxy: Returning cached data.
    Data: Database Data
    ```
    

### **프록시 패턴의 장단점**

1. **장점**
    - **접근 제어**: 민감한 데이터나 객체에 대한 접근을 제어
    - **성능 최적화**: 지연 로딩이나 캐싱을 통해 시스템 성능 개선
    - **추가 기능 구현**: 로깅, 모니터링 등 기존 객체에 영향을 주지 않고 기능 추가
2. **단점**
    - **복잡성 증가**: 프록시 객체를 추가하면 시스템 구조가 복잡해질 수 있음
    - **오버헤드**: 프록시 처리로 인해 응답 시간이 약간 증가

### **프록시 패턴의 실제 사용 사례**

1. **프록시 서버**
    - 인터넷 요청을 중간에서 처리하며, 캐싱, 필터링, 로깅 등을 수행
    - ex) Cloudflare, Nginx
2. **Java RMI(Remote Method Invocation)**
    - 원격 객체에 대한 호출을 처리하는 원격 프록시
3. **Spring AOP**
    - 메서드 실행 전후에 로깅, 보안 검사 등의 기능을 추가하는 데 사용
4. **Hibernate Lazy Loading**
    - 데이터베이스의 엔터티를 필요할 때만 가져오는 방식

---

프록시 패턴은 특히 대규모 시스템에서 성능 최적화와 접근 제어를 위해 필수적으로 활용되는 패턴이다.

**캐싱과 지연 로딩**을 적절히 활용하면 성능과 리소스 관리 측면에서 큰 이점을 얻을 수 있다.