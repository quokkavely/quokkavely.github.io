---
layout : single
title : "[MSA] 분산 저장 시 데이터 동기화 문제"
categories: MSA
tag : [MSA, SpringCloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

인프런 Dowon Lee님의 Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의를 듣고 정리한 내용입니다.😊<br>
[Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의 들으러 가기👩‍🏫](https://inf.run/GHeRm)
{: .notice--warning}



# 데이터 동기화 문제

이번 강의에서는 하나의 서비스를 여러 인스턴스로 띄운 상황에서 데이터가 분산 저장되며 발생하는 문제와 이를 해결하기 위한 방법들에 대해 학습했다.

<img src = "https://github.com/user-attachments/assets/d0c9413d-27c2-4d68-ba0d-b57385f05201" width= 400/>


## **주요 문제: 여러 인스턴스에서의 데이터 분산**

1. **MSA 환경에서의 로드 밸런싱**
    - 클라이언트 요청이 들어오면 로드 밸런서(예: 라운드 로빈 방식)가 요청을 여러 인스턴스에 분산 처리한다.
    - 두 개의 오더 서비스 인스턴스가 서로 다른 **H2 내장 데이터베이스**를 사용하고 있는 상황.
    - 같은 유저가 5번의 주문을 연속으로 했을 때, 요청이 번갈아가며 두 인스턴스에서 처리되었다.
    
    >
      👉🏻
      인텔리제이에서 Terminal tab 을 통해 2개의 인스턴스를 실행하고 유레카서버에서 두가지 인스턴스를 확인하여 Test 가능

    
2. **데이터 분산의 문제**
    - 한 인스턴스의 데이터베이스에는 2개의 주문 데이터가, 다른 인스턴스의 데이터베이스에는 3개의 주문 데이터가 저장되었다.
        
        <img src = "https://github.com/user-attachments/assets/d8a3b229-acb7-477a-a2b3-f8420225e371" width= 400/>
        
        <img src = "https://github.com/user-attachments/assets/5edf10fc-4ff5-418b-833c-eb9b5996f88e" width= 400/>
        
    - 동일한 사용자 데이터임에도 불구하고, **인스턴스마다 데이터가 분산**되어 있어 데이터 정합성 문제가 발생했다.
3. **결과**:
    - Postman으로 요청을 보냈을 때, 번갈아가며 다른 인스턴스에서 데이터를 가져왔다.
    - 동일한 데이터가 한 곳에 통합되어 있지 않아 **데이터 불일치와 동기화 문제**가 발생.


## **해결 방안**

### **(1) 하나의 데이터베이스 사용**

- 여러 인스턴스가 존재하더라도 데이터베이스를 하나로 통합하면 데이터 분산 문제를 해결할 수 있다.
- 모든 인스턴스가 같은 데이터베이스를 참조하므로 데이터의 정합성을 유지할 수 있다.
- **문제점**
    - **트랜잭션 관리 문제**
        - 다수의 인스턴스가 동시에 하나의 DB를 참조하면, **동시성** 제어나 **트랜잭션 관리**가 복잡해진다.

---

### **(2) 데이터 동기화를 위한 메시지 큐 도입**

- 각 인스턴스가 독립적인 DB를 유지하면서 **RabbitMQ**나 **Apache Kafka** 같은 메시지 큐(Message Queue)를 이용해 데이터를 동기화한다.
- 한 인스턴스에서 데이터가 저장되면, 메시지 큐를 통해 다른 인스턴스의 데이터베이스에도 해당 데이터를 전달해 업데이트한다
- **장점**
    - 각 서비스의 독립성을 유지하면서 데이터 정합성을 확보할 수 있다.
    - 메시지 큐는 데이터 전송 순서를 보장하고, 대량의 데이터를 효율적으로 처리할 수 있다.
- **구현 방식**
    1. 오더 서비스는 데이터베이스에 저장하기 전, 메시지 큐에 데이터 변경 이벤트를 전달한다.
    2. 메시지 큐를 구독하는 다른 인스턴스들이 이벤트를 받아 자신의 데이터베이스를 업데이트한다.
- **예제**
    
    ```java
    // 메시지 발행 (Producer)
    KafkaTemplate<String, Order> kafkaTemplate = new KafkaTemplate<>(producerFactory);
    kafkaTemplate.send("order-topic", newOrder);
    
    // 메시지 수신 (Consumer)
    @KafkaListener(topics = "order-topic", groupId = "order-group")
    public void consumeOrder(Order order) {
        // 수신한 주문 데이터를 DB에 저장
        orderRepository.save(order);
    }
    ```
    

---

### **(3) 메시지 큐와 단일 데이터베이스의 혼합 사용**

- 메시지 큐를 통해 데이터를 동기화하면서, 최종적으로 **하나의 통합 데이터베이스**에 데이터를 저장하는 방식.
- 메시지 큐는 데이터를 임시로 수집하고, 데이터베이스는 최종 데이터를 보관하는 역할을 한다.
- **장점**
    - 메시지 큐는 동시성을 처리하고 데이터 전달의 순서를 보장한다.
    - 데이터베이스는 최종 데이터 저장소로 활용되며 데이터의 일관성을 유지한다.
- **구현 방식**:
    1. 모든 오더 서비스 인스턴스는 메시지 큐로 데이터를 전송.
    2. 메시지 큐에서 데이터를 처리하여 하나의 데이터베이스에 저장.
    3. 각 인스턴스는 필요한 데이터를 공통 데이터베이스에서 가져온다.

---

MSA 환경에서의 데이터 동기화는 시스템의 복잡성을 증가시키지만, **메시지 큐**, **하나의 데이터베이스**, 혹은 이 둘의 혼합 사용을 통해 효과적으로 문제를 해결할 수 있다는 것을 알게되었다,

아는만큼 보인다는 말처럼 사실 프로젝트 할 때만 해도 이런 문제에 대해서는 고민해보지도 않았고 하나의 DB만 사용하기 때문에 데이터 동기화 문제가 당연히 없다고 생각했다. 물론 그 말도 맞지만 트랜잭션까지는 깊이 생각해본 적이 없었는데 MSA 환경에서는 여러 서비스가 서로 독립적으로 동작하면서 동시에 같은 데이터에 접근할 가능성이 있기 때문에, 동시성 문제가 더 복잡질 것이라는 생각까지는 못했던 것 같다.

ERP 프로젝트 할 때도 동시성 문제에 대해서 잠깐 나왔는데 그 때 동시성에 대해서 조금 더 고민해볼 걸 그랬다.

앞으로 동시성 문제랑 그리고 메세지큐 방식으로 데이터 동기화 어떻게 해결할 수 있는지 강의 들으면서 고민해보아야겠다..@!