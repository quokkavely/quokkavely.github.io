---
layout : single
title : "[디자인패턴] DI와 DIP"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# **DI와 DIP**

## **1. 비유를 통한 DI와 DIP 이해**

### **의존성 (Dependency)이란?**

- **비유**: 커피 머신이 특정 브랜드의 커피콩만 사용할 수 있다면 어떻게 될까?→ 커피콩 브랜드가 바뀌면 커피 머신도 바꿔야 한다.

**해결책**: 커피 머신이 "커피콩" 인터페이스만 요구하도록 만들고, 어떤 커피콩도 연결할 수 있게 하면 커피콩을 쉽게 교체할 수 있다.

### **DI (Dependency Injection, 의존성 주입)**

- **비유**: 커피 머신에 커피콩을 직접 넣지 않고, 외부에서 커피콩을 주입해주는 방식.

즉, **의존 객체를 외부에서 제공**해줌으로써 커피 머신은 커피콩의 세부사항에 대해 알 필요가 없다.

### **DIP (Dependency Inversion Principle, 의존관계 역전 원칙)**

- 커피 머신은 **"커피콩"이라는 추상화**에 의존해야지, 특정 커피콩 브랜드(세부 사항)에 의존해서는 안 된다.
- 상위 모듈(커피 머신)은 추상화(인터페이스)에만 의존하고, 실제 세부 구현은 외부에서 주입한다.

## **2. 새로운 예제: "알림 시스템"**

이번 예제로 **DI와 DIP를 적용하여 다양한 알림(Notification) 시스템**을 유연하게 확장하는 모습을 볼 수 있다.

### **시나리오**

- 사용자가 알림(Notification)을 받을 수 있는 다양한 방식:
    1. 이메일(E-mail)
    2. 문자 메시지(SMS)
    3. 푸시 알림(Push Notification)
- `NotificationService`는 어떤 알림 방식이든 사용될 수 있어야 하고, 새로운 알림 방식이 추가되더라도 변경이 최소화되어야 한다.

## **3. 코드 예제**

### **1. 알림 인터페이스 정의**

추상화를 통해 `NotificationService`는 알림의 세부 구현을 몰라도 된다.

```java
// 추상화: Notification 인터페이스
interface Notification {
    void send(String message);
}
```

### **2. 구체적인 알림 방식 구현**

각각의 알림 방식은 `Notification` 인터페이스를 구현한다.

```java
// 이메일 알림
class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("이메일 알림: " + message);
    }
}

// SMS 알림
class SMSNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("SMS 알림: " + message);
    }
}

// 푸시 알림
class PushNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("푸시 알림: " + message);
    }
}
```

### **3. NotificationService: DIP 적용**

`NotificationService`는 구체적인 알림 방식에 의존하지 않고, **Notification 인터페이스**에 의존한다.

```java
import java.util.List;

class NotificationService {
    private final List<Notification> notifications;

    // 의존성 주입: 알림 객체를 외부에서 주입받음
    public NotificationService(List<Notification> notifications) {
        this.notifications = notifications;
    }

    // 모든 알림을 순회하며 메시지 전송
    public void notifyUsers(String message) {
        for (Notification notification : notifications) {
            notification.send(message);
        }
    }
}
```

### **4. Main 클래스: 다양한 알림 방식을 주입**

여기서 `Notification` 객체를 외부에서 주입한다.

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        // 알림 방식을 준비
        List<Notification> notificationList = new ArrayList<>();
        notificationList.add(new EmailNotification());
        notificationList.add(new SMSNotification());
        notificationList.add(new PushNotification());

        // NotificationService에 의존성 주입
        NotificationService service = new NotificationService(notificationList);

        // 알림 발송
        service.notifyUsers("안녕하세요! 새로운 공지가 도착했습니다.");
    }
}
```

### **5. 실행 결과**

```
이메일 알림: 안녕하세요! 새로운 공지가 도착했습니다.
SMS 알림: 안녕하세요! 새로운 공지가 도착했습니다.
푸시 알림: 안녕하세요! 새로운 공지가 도착했습니다.
```

## **4. DI와 DIP의 장단점**

### **장점**

1. **유연성과 확장성**
    - 새로운 기능이 추가될 때 기존 코드 수정이 최소화된다.(예: 새로운 알림 방식 `SlackNotification` 추가 시, `NotificationService`는 수정할 필요가 없음)
2. **결합도 감소**
    - 상위 모듈이 하위 모듈의 구체적인 구현이 아닌 **추상화(인터페이스)**에 의존하므로 코드 재사용성이 높아진다.
3. **단위 테스트 용이**
    - 테스트 시 Mock 객체나 더미 객체를 쉽게 주입할 수 있다.→ 테스트 환경과 운영 환경에서 다른 객체를 사용 가능.
4. **코드의 가독성과 유지보수성 향상**
    - 의존성이 명확하게 드러나므로 애플리케이션 구조를 쉽게 이해할 수 있다.

### **단점**

1. **초기 설정의 복잡도**
    - 추상화(인터페이스)를 정의하고 외부에서 의존성을 주입해야 하므로 코드의 구조가 복잡해질 수 있다.
2. **클래스 수 증가**
    - DIP를 적용하면서 인터페이스와 구현 클래스가 별도로 필요하므로 클래스의 수가 늘어난다.
3. **런타임 에러 가능성**
    - 의존성 주입이 런타임에 이루어지기 때문에 잘못된 객체가 주입되면 오류가 발생할 수 있다.

---
### DI와 DIP 요약
1. **DI (의존성 주입)**
    
    객체의 의존성을 외부에서 주입하여 코드 간 결합도를 낮춘다.
    
2. **DIP (의존관계 역전 원칙)**
    - 상위 모듈(알림 서비스)은 추상화(인터페이스)에 의존하고,하위 모듈(이메일, SMS 등)은 추상화를 구현한다.
3. **새로운 코드 작성 흐름**
    - 알림 인터페이스(`Notification`)를 중심으로 설계.
    - 알림 방식(이메일, SMS 등)만 추가되면 확장이 끝난다.
4. **장단점**
    - **장점**
        - 코드의 유연성과 확장성 향상
        - 테스트와 유지보수 용이
    - **단점**
        - 클래스 증가 및 초기 설정의 복잡도
        - 런타임 의존성 오류 가능성


spring 들어가면서 DI랑 DIP 는 공부했던 거라서 그렇게 어렵지 않았는데 이 원칙들이 디자인 패턴의 일부라는 것은 이번에 처음 알게 되었다
CS 도 내가 배운 부분이 있구나 하고 뭔가 뿌듯했다.
