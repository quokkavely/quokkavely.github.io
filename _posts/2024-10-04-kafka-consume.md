---
layout : single
title : "[Project] kafka+SSE 알림 기능 구현- 2 : kafka consumer"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## kafka : Consumer 및 알림 서비스 구현

Kafka는 분산형 메시징 시스템으로, Producer가 보낸 메시지를 Broker를 통해 Consumer가 받아 처리하는 구조

### 1️⃣ consumer configuration

`ConsumerFactory`와 `ConcurrentKafkaListenerContainerFactory`는 Kafka에서 메시지를 수신할 때 설정을 담당하며, 비동기적으로 데이터를 처리하는 데 중요한 역할

```java
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, NotificationMessage> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        
        //Kafka 클러스터의 주소를 설정(여러 서버일 경우 콤마로 구분하여 설정 가능)
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        
        // 이 Consumer가 속하는 그룹 ID를 설정
        // 동일한 그룹 ID를 가진 Consumer들은 메시지를 공유하여 처리 (즉, 그룹 내에서 메시지 로드 밸런싱이 발생)
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "alarm-api");
        
        // Kafka 메시지의 키를 역직렬화할 클래스를 지정 (문자열로 역직렬화)
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
     // Kafka 메시지의 값을 역직렬화할 클래스를 지정 (JSON 형태로 역직렬화)
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
				
 // NotificationMessage라는 커스텀 메시지 타입을 역직렬화하기 위해 JsonDeserializer 객체를 생성
        JsonDeserializer<NotificationMessage> deserializer = new JsonDeserializer<>(NotificationMessage.class);
        
        // 역직렬화할 수 있는 신뢰할 수 있는 패키지들을 추가
        // "*"는 모든 패키지를 신뢰하도록 설정 (보안상 유의해야 함)
        deserializer.addTrustedPackages("*"); 
        
        //config 설정과 함께 StringDeserializer(키용)와 JsonDeserializer(값용)를 사용하여 Kafka Consumer 팩토리를 생성
        return new DefaultKafkaConsumerFactory<>(config, new StringDeserializer(), deserializer);
    }

	
		// Kafka Consumer가 메시지를 수신하기 위한 Kafka 리스너 컨테이너 팩토리를 생성
		//이 컨테이너 팩토리는 메시지를 수신하는 @KafkaListener 메서드를 트리거하는 데 사용
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, NotificationMessage> kafkaListenerContainerFactory() {
    
    //Kafka 리스너 컨테이너 팩토리 객체를 생성
    ConcurrentKafkaListenerContainerFactory<String, NotificationMessage> factory = new ConcurrentKafkaListenerContainerFactory<>();
    
    //Kafka 리스너가 사용할 Consumer 팩토리를 설정
    factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

1. ConsumerFactory<String, NotificationMessage>
    - Kafka에서 메시지를 수신할 때 사용하는 Consumer를 생성하는 팩토리
    - 메시지의 키 타입과 값 타입을 지정해야 함.
    - 이 경우, 키는 String, 값은 NotificationMessage 객체
2. ConcurrentKafkaListenerContainerFactory 
    - `@KafkaListener`를 통해 메시지를 수신할 때 사용할 리스너 컨테이너 팩토리를 설정하는 데 사용
    - 이 팩토리는 멀티스레드 환경에서 동시에 여러 메시지를 처리할 수 있도록 지원
3. `JsonDeserializer` 
    - `NotificationMessage` 클래스의 인스턴스를 JSON 데이터로부터 역직렬화하기 위해 사용
    - `addTrustedPackages("*")` 설정을 통해 모든 패키지를 신뢰할 수 있도록 설정하고 있지만 보안적인 이유로 실제 운영 환경에서는 신뢰할 수 있는 특정 패키지만 추가하는 것이 좋다.
    - 그래서 와일든카드가 아닌 신뢰할 수 있는 패키지로 com.alarm.kafka 로 설정해 두었다.

### 2️⃣ NotificationMessage

알림 메세지 : 이벤트가 발생했을 때 사용자에게 전달하는데 사용된다.

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class NotificationMessage {
    private Long employeeId; //알림 받을 직원의 id (pk)
    private String message; //알림 메세지 내용
	    private Long typeId; //알림 관련된 엔티티의 id (pk)
    private NotificationType type; 

    public enum NotificationType {
        APPROVAL, //전자결재 승인자에게 알림
        NOTICE, //공지사항 > 전직원에게 알림
        DOCUMENT; //전자결재 문서 > 전자결재 작성자에게 알림 또는 관련된 사람들
    }
}
```

### 3️⃣NotificationCounsumer

- kafka를 통해 수신된 알림 메시지를 처리하는 역할을 하는 서비스
- `@KafkaListener`를 사용해 특정 토픽(topic)에서 메시지를 수신하고, 이를 처리하여 알림 저장 및 실시간 전송을 수행

```java
@Service
@RequiredArgsConstructor
public class NotificationConsumer {

    private final NotificationService notificationService; 

    @KafkaListener(topics = "approval-topic", groupId = "alarm-api")
    public void consume(NotificationMessage message) {
         // 수신된 메세지 저장
        notificationService.saveNotification(message);
       
        //실시간 알림 전송
        notificationService.sendRealTimeNotification(message);
    }

    @KafkaListener(topics = "document-topic", groupId = "alarm-api")
    public void consumeDocument(NotificationMessage message) {

        notificationService.saveNotification(message);
        notificationService.sendRealTimeNotification(message);
    }

    @KafkaListener(topics = "notice-topic", groupId = "alarm-api")
    public void consumeNotice(NotificationMessage message) {

        notificationService.saveNotification(message);
        notificationService.sendRealTimeNotification(message);
    }
}
```

1. Kafka의 특정 토픽에서 수신된 메시지를 알림 서비스(`NotificationService`)로 전달하여, 데이터베이스에 알림을 저장하고 실시간 알림을 사용자에게 전송하는 역할
2. 각 메서드는 수신된 메시지의 유형에 따라 분리되어 있으며, 이를 통해 토픽별로 메시지를 처리할 수 있다.
3. 예를들어, @KafkaListener(topics = "approval-topic", groupId = "alarm-api")
    - Kafka의 "approval-topic" 토픽으로부터 메시지를 수신
    - groupId는 "alarm-api"로 설정되어 있어서 동일한 그룹 ID를 가진 다른 Consumer들이 있을 경우 메시지를 공유해서 처리

### 4️⃣ NotificationService

 알림 저장과 실시간 알림 전송을 처리하는 서비스

```java
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final NotificationRepository notificationRepository;
    private final SseEmitterService sseEmitterService;

    public void saveNotification(NotificationMessage message) {
	     // NotificationMessage를 기반으로 알림 엔터티를 생성하고 DB에 저장
        Notification notification = createNotificationEntity(message);
        notificationRepository.save(notification);
    }

    private Notification createNotificationEntity(NotificationMessage message) {
        Notification notification;
				
				// NotificationMessage의 타입에 따라 서로 다른 알림 엔터티를 생성
        switch (message.getType()) {
            case APPROVAL:
                notification = new ApprovalNotification();
                ((ApprovalNotification) notification).setApprovalId(message.getTypeId());
                break;
            case NOTICE:
                notification = new NoticeNotification();
                ((NoticeNotification) notification).setPostId(message.getTypeId());
                break;
            case DOCUMENT:
                notification = new DocumentNotification();
                ((DocumentNotification) notification).setDocumentId(message.getTypeId());
                break;
            default:
                throw new IllegalArgumentException("Unknown notification type: " + message.getType());
        }
				
				// 공통 필드 설정
        notification.setMessage(message.getMessage());
        notification.setEmployeeId(message.getEmployeeId());

        return notification;
    }

    public void sendRealTimeNotification(NotificationMessage message) {
        // 실시간으로 알림을 특정 직원에게 전송
        sseEmitterService.sendToEmployee(message.getEmployeeId(), message);
    }

}
```

1. Kafka에서 수신된 `NotificationMessage`를 `NotificationConsumer`가 받아서 이 서비스의 `saveNotification` 메서드를 호출해 알림을 저장
2. `NotificationMessage`는 알림 유형에 따라 적절한 엔터티로 변환되어 데이터베이스에 저장되고, 이후 `sendRealTimeNotification`을 통해 SSE를 사용해 실시간으로 알림이 전송 됨

### 5️⃣ SseEmitterService

 SSE(Server-Sent Events)를 사용해 클라이언트에게 실시간으로 알림을 전송하는 서비스

```java
@Service
public class SseEmitterService {

    // 사용자별로 SseEmitter를 저장하는 맵 (동시성을 위해 ConcurrentHashMap 사용)
    private final Map<Long, SseEmitter> emitters = new ConcurrentHashMap<>();

    // 새로운 SSE 연결 생성 및 저장
    public SseEmitter createEmitter(Long employeeId) {
        SseEmitter emitter = new SseEmitter(60 * 1000L);  // 타임아웃 1분 (필요에 따라 조정 가능)

        // 타임아웃 발생 시 Emitter 제거
        emitter.onTimeout(() -> emitters.remove(employeeId));

        // 에러 발생 시 Emitter 제거
        emitter.onError(e -> emitters.remove(employeeId));

        emitters.put(employeeId, emitter);

        return emitter;
    }

    // 특정 사용자에게 실시간 알림 전송
    public void sendToEmployee(Long employeeId, NotificationMessage message) {
        SseEmitter emitter = emitters.get(employeeId);

        if (emitter != null) {
            try {
                emitter.send(SseEmitter.event()
                        .name("notification")
                        .data(message));  // 메시지 전송

            } catch (IOException e) {
                // 전송 중 에러 발생 시 해당 Emitter 제거
                emitters.remove(employeeId);
            }
        }
    }

    // 사용자가 연결을 닫을 때 Emitter 제거
    public void removeEmitter(Long employeeId) {
        emitters.remove(employeeId);
    }
}

```

1.   private final Map<Long, SseEmitter> emitters = new ConcurrentHashMap<>();
    - 각 사용자별로 `SseEmitter` 객체를 저장하는 맵
    - `employeeId`로, 사용자를 구분하기 위해 사용하고, 값은 해당 사용자의 `SseEmitter` 객체
    - ConcurrentHashMap
        - 자바의 `java.util.concurrent` 패키지에 포함된 클래스
        - 동시성을 지원하는 `HashMap`으로, 멀티스레드 환경에서 안전하게 사용 가능
        - 일반적인 `HashMap`은 여러 스레드가 동시에 접근할 경우, 데이터 일관성이 깨지거나 `ConcurrentModificationException`이 발생할 수 있지만 `ConcurrentHashMap`은 그런 문제를 방지해준다.
        - null 키나 null 값을 허용하지 않음
        - 고성능 : 멀티스레드 환경에서 성능을 고려해 설계됐기 때문에, 여러 스레드가 동시에 데이터를 추가하거나 수정해도 성능 저하가 적다.
    - 여러 사용자의 `SseEmitter`를 관리하는 맵에서 동시에 여러 스레드가 접근해 알림을 추가하거나 삭제할 수 있기 때문에 ConcurrentHashMap을 사용하여 동시성 문제를 해결하고 안전하게 데이터를 처리할 수 있다.

---

알림 시스템을 처음 설계하고 구현하는 과정에서 Kafka와 SSE를 활용하는 방법을 배우게 된 것은 큰 성과라고 생각된다. 특히 Kafka를 사용해 분산된 시스템 간에 메시지를 주고받고, 토픽 별로 메시지를 처리하는 구조는 확장성과 유지보수 측면에서 매우 유용하다는 것을 알게 되었다. SSE는 실시간 알림 기능을 비교적 간단하게 구현할 수 있게 되었다.🤔


<br>
<br><br><br><br><br>