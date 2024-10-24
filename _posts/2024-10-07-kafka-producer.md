---
layout : single
title : "[Project] kafka+SSE 알림 기능 구현- 3 : kafka producer"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## kafka : Consumer - Producer 구현

전자결재 서비스는 거의 대부분 구현되어서 가장 먼저 kafka 설정을 시도했다.

### 1️⃣ kafkaConfig

Kafka의 Producer 설정을 담당하는 구성 클래스

```java
@Configuration
public class kafkaConfig {
    @Bean
    public ProducerFactory<String, NotificationMessage> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, NotificationMessage> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

1. ConsumerFactory<String, NotificationMessage>
    - Kafka에서 메시지를 수신할 때 사용하는 Consumer를 생성하는 팩토리
    - 메시지의 키 타입과 값 타입을 지정해야 함.
    - 이 경우, 키는 String, 값은 NotificationMessage 객체
2. ConcurrentKafkaListenerContainerFactory 
    - @KafkaListener를 통해 메시지를 수신할 때 사용할 리스너 컨테이너 팩토리를 설정하는 데 사용
    - 이 팩토리는 멀티스레드 환경에서 동시에 여러 메시지를 처리할 수 있도록 지원
3.  **KafkaTemplate<String, NotificationMessage>**
    - KafkaTemplate은 Kafka로 메시지를 전송할 때 사용하는 템플릿 클래스
    - 이 클래스는 설정된 ProducerFactory를 기반으로 생성된다
    - KafkaTemplate.send() 메서드를 통해 토픽 이름, 메시지 키, 메시지 값을 지정해 전송할 수 있어..

### 2️⃣ NotificationMessage

- Kafka를 사용할 때 `Producer`와 `Consumer`가 사용하는 메시지 구조는 동일해아 한다.
- 메시지 구조가 동일해야만 `Producer`가 전송한 데이터를 `Consumer`가 올바르게 해석하고 처리할 수 있기 때문이다.
- 그래서 복사한 다음 전자결재에서 사용하는 notificationType만 수정하였다.

```java
@Builder
@Getter
public class NotificationMessage {
    private Long employeeId;
    private String message;
    private Long typeId;
    private NotificationType type;

    public enum NotificationType {
        APPROVAL,
        DOCUMENT;
    }
}
```

### 3️⃣ApporvalProducer

`KafkaTemplate`을 사용해서 특정 토픽에 알림 메시지를 전송하는 역할을 함

전자결재에서는 승인자한테 보내는 알림이랑 전자결재 작성자에게 보내는 알림이랑 분리할 필요가 있을 것 같아서 나눠서 구현하였다.!

```java
package com.alarm.kafka;

import lombok.RequiredArgsConstructor;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class ApprovalProducer {
		//template 의존성 주입
    private final KafkaTemplate<String, NotificationMessage> kafkaTemplate;
		
		
		//전자결재 결재선 (승인자)에 대한 알림
    public void sendApprovalNotification(Long employeeId, String message, Long approvalId) {
        NotificationMessage notificationMessage = NotificationMessage.builder()
                .message(message)
                .employeeId(employeeId)
                .typeId(approvalId)
                .type(NotificationMessage.NotificationType.APPROVAL)
                .build();
        kafkaTemplate.send("approval-topic", notificationMessage);
    }
    
    
    //전자결재 작성자에 대한 알림 : 문서가 승인되었는지 반려되었는지 알기 위해서
    public void sendDocumentNotification(Long employeeId, String message, Long documentId) {
        NotificationMessage notificationMessage = NotificationMessage.builder()
                .message(message)
                .employeeId(employeeId)
                .typeId(documentId)
                .type(NotificationMessage.NotificationType.DOCUMENT)
                .build();
        kafkaTemplate.send("document-topic", notificationMessage);
    }
}

```

1. KafkaTemplate
    - 템플릿을 통해 메시지를 특정 토픽으로 전송이 가능해진다
    - 예를 들어 `sendApprovalNotification` 메서드에서는 `"approval-topic"` 토픽에 메시지를 전송한다.
2. 토픽별로 메세지를 분리해서 특정 토픽에 대한 메시지만 처리하도록 구성

---

Kafka를 사용해 `Producer`와 `Consumer`를 구현하면서, 메시지를 토픽별로 분리해 전송하는 것이 얼마나 중요한지 깨달았다. 특히 `ApprovalProducer` 클래스에서 전자결재와 문서 관련 알림을 각각 다른 토픽으로 관리함으로써, 메시지 흐름을 명확하게 구분할 수 있었고, 시스템 확장에도 유리하다는 점을 느꼈다.

또한, `NotificationMessage` 객체를 일관되게 사용해 `Producer`와 `Consumer` 간의 데이터 호환성을 유지하는 방법도 배웠다. Kafka를 처음 접했지만, 이벤트 기반 아키텍처가 실시간 데이터 처리에 어떻게 유용하게 사용될 수 있는지 실감할 수 있는 기회였다


<br>
<br><br><br><br><br>