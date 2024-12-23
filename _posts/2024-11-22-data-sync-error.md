---
layout : single
title : "동시성 문제 및 해결방안"
categories: OS
tag : [CS, JAVA, 프로세스]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

어제 [Data 동기화 문제에 대한 강의](https://quokkavely.github.io/msa/data-synchronization/)를 들으면서 나온 동시성 문제에 대해서 잘 알지 못해서 이번 기회에 새로 정리해보려고 한다. <br> MSA 환경이 아닌 MA에서도 발생하는 동시성 문제와 어떻게 해결할 수 있는지 찾아본 자료를 정리했다.

# **동시성 문제란?**

다수의 사용자가 DB에 동시 접근하는 일은 비일비재하게 발생하는데 이런 문제는 transaction에 의해 순차적으로 실행되지 않고 순서에 상관없이 동시에 실행되는 것을 의미한다.

즉 , 동시성 문제는 여러 클라이언트나 프로세스가 동시에 같은 자원(예: 데이터베이스, 파일)에 접근하거나 수정하려고 할 때, 데이터 일관성이 깨지거나 예상치 못한 결과가 발생하는 상황을 말한다.

예를 들어, 두 개의 요청이 동시에 같은 데이터를 읽고 수정한다면, 레이스 컨디션(Race Condition)이나 데드락(Deadlock) 같은 문제가 발생할 수 있다.

## 🪄주요 동시성 문제

1. **레이스 컨디션 (Race Condition)**
    - 두 개 이상의 스레드가 같은 데이터를 동시에 읽고 수정하려 할 때, 결과가 예측 불가능해지는 상황
    - ex ) 은행 계좌에서 동시에 돈을 인출할 때 잔액이 잘못 계산되는 문제.
2. **데드락 (Deadlock)**
    - 두 개 이상의 스레드가 서로의 리소스를 기다리며 영원히 대기 상태에 빠지는 문제
    - ex ) 두 트랜잭션이 서로의 락을 기다리는 상황.
3. **라이블락 (Livelock)**
    - 스레드가 작업을 반복하며 진전을 이루지 못하는 상황. 데드락과 비슷하지만, 계속 상태를 바꾼다는 점이 다름
4. **기아상태 (Starvation)**
    - 특정 스레드가 우선순위가 낮아 자원을 할당받지 못하고 대기 상태에 머무는 상황

## **동시성 문제 해결의 기본 원칙**

1. **공유 자원 최소화**:
    - 최대한 공유 자원을 줄이고, 스레드 간 독립적으로 작업할 수 있도록 설계
2. **불변 객체 사용**:
    - 불변 객체는 상태를 변경할 수 없으므로 동시성 문제가 발생하지 않음
3. **작업 단위 축소**:
    - 락을 사용할 때, 가능한 한 짧은 시간 동안만 락을 유지하여 성능 저하를 줄임
4. **테스트와 모니터링**:
    - 동시성 문제는 복잡하기 때문에 철저한 테스트와 모니터링이 필요


# **동시성 문제 해결 방안**

### 1️⃣ 락(Lock) 사용

- **비관적 락 (Pessimistic Lock)**
    - **데이터가 충돌 가능성이 높은 경우 사용**
    - 자원을 사용하는 동안 다른 스레드가 접근하지 못하도록 강제 락을 거는 방식.
    - **장점**: 데이터의 안전성을 보장.
    - **단점**: 성능 저하, 데드락 위험.
- **낙관적 락 (Optimistic Lock)**:
    - 자원을 사용할 때 충돌이 발생하지 않을 것으로 가정하고 작업을 수행하며, 충돌이 발생하면 롤백.
    - 보통 **버전 관리 필드**(Versioning)를 사용.
    - **장점**: 높은 성능.
    - **단점**: 충돌이 빈번하면 성능 저하.

### 2️⃣ 트랜잭션 격리 수준

- 데이터베이스에서 제공하는 격리 수준으로 동시성 문제를 완화할 수 있다.
    - **READ UNCOMMITTED**: 가장 낮은 수준, 데이터의 무결성 보장 X.
    - **READ COMMITTED**: 대부분의 애플리케이션에서 기본값으로 사용.
    - **REPEATABLE READ**: 동일 트랜잭션 내에서 읽은 데이터가 변경되지 않음.
    - **SERIALIZABLE**: 가장 높은 격리 수준, 동시성 저하 가능성.

### 3️⃣ Java에서 제공하는 도구

#### **Synchronized**

- Java에서 특정 블록이나 메서드에 락을 걸어 다른 스레드가 동시에 접근하지 못하게 함.
- 성능 저하를 초래할 수 있다 → 모든 스레드가 순차적으로 자원에 접근해야 하기 때문
- 결국, 실무에서 잘 사용하지 않음

#### **Concurrent 패키지**

- 자바에서 동시성 문제를 해결하기 위한 다양한 도구를 제공
- ReentrantLock 클래스는 synchronized보다 더 유연한 락을 제공 : 락을 명시적으로 획득하고 해제할 수 있도록 함
- ConcurrentHashMap 클래스는 동시성을 지원하는 해시맵 :  여러 스레드가 동시에 안전하게 데이터를 읽고 쓸 수 있다.
- 이 외에도 CountDownLatch, Semaphore, CyclicBarrier 등 다양한 클래스 존재

#### **CAS (Compare and Swap)**

- 자원의 현재 상태를 읽고, 예상했던 상태와 일치할 경우 자원을 업데이트하는 방식.
    - 사용 예: Java의 **AtomicInteger**, **AtomicLong**.
    - **장점 :** 락 없이 동시성 문제 해결.
    - **단점 :** 복잡한 작업에서는 사용 어려움.

### 4️⃣ 메시지 큐 활용

- 동시성 문제를 완화하기 위해 RabbitMQ, Kafka와 같은 메시지 큐를 사용.
    - 데이터를 하나의 프로듀서가 메시지로 보내고, 여러 컨슈머가 순차적으로 처리.
    - 데이터의 처리 순서와 정합성을 유지하는 데 유리.

---

**참고자료**

https://devwithpug.github.io/java/java-thread-safe/

[https://rachel0115.tistory.com/entry/Java-멀티-스레드-환경에서-발생할-수-있는-동시성-이슈와-해결-방법](https://rachel0115.tistory.com/entry/Java-%EB%A9%80%ED%8B%B0-%EC%8A%A4%EB%A0%88%EB%93%9C-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-%EB%B0%9C%EC%83%9D%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88%EC%99%80-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95)

https://f-lab.kr/insight/java-concurrency-issues-20241111

---

이번 정리를 통해 동시성 문제와 그 해결 방안에 대해 더 깊이 이해할 수 있었다.<br> 특히, 락이나 트랜잭션 격리 수준 같은 기본적인 기법뿐만 아니라,<br> 메시지 큐와 같은 현대적인 해결책이 실무에서 얼마나 중요한지 알게 되었다.<br> 앞으로 실무 설계 시 이러한 기법들을 어떻게 적용할지 고민해 보아야겠다.


<br>
<br>
<br>
<br>
<br>