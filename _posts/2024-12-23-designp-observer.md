---
layout : single
title : "[디자인패턴] Observer Pattern"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## Observer Pattern

옵저버 패턴(Observer Pattern)은 **객체의 상태 변화**를 관찰하고, 그 상태 변화가 발생하면 구독자(옵저버)에게 알림을 보내는 디자인 패턴이다.

- **핵심 개념**
    - 주체(Subject): 상태 변화를 관리하고, 옵저버들에게 알리는 역할.
    - 옵저버(Observer): 주체의 상태 변화에 관심이 있으며, 알림을 받는 객체.
- **사용 사례**
    - **SNS 알림 시스템**: 팔로우한 사용자가 글을 올리면 알림.
    - **MVC 패턴**: 모델(Model)이 변경되면 뷰(View)에게 업데이트를 알림.
    - **이벤트 기반 시스템**: 버튼 클릭, 데이터 업데이트 등.

### **구조**

1. **Subject**
    - 상태 변화를 관리.
    - 옵저버 등록/해지/알림 기능 제공.
2. **Observer**
    - 주체의 상태 변화를 관찰하고, 업데이트 내용을 처리.

### 예제

아래는 주식 가격 변화를 관찰하는 옵저버 패턴이다,.

```java
import java.util.ArrayList;
import java.util.List;

// Subject: 주식(Stock)
interface Stock {
    void registerObserver(Investor investor);  // 옵저버 등록
    void removeObserver(Investor investor);    // 옵저버 해지
    void notifyObservers();                    // 상태 변화 알림
}

// Concrete Subject: 구체적인 주식 구현
class StockPrice implements Stock {
    private List<Investor> investors = new ArrayList<>();
    private String stockName;
    private double price;

    public StockPrice(String stockName, double initialPrice) {
        this.stockName = stockName;
        this.price = initialPrice;
    }

    public void setPrice(double newPrice) {
        System.out.println(stockName + " 주식 가격 변경: " + price + " → " + newPrice);
        this.price = newPrice;
        notifyObservers(); // 가격 변경 시 옵저버들에게 알림
    }

    public double getPrice() {
        return price;
    }

    public String getStockName() {
        return stockName;
    }

    @Override
    public void registerObserver(Investor investor) {
        investors.add(investor);
    }

    @Override
    public void removeObserver(Investor investor) {
        investors.remove(investor);
    }

    @Override
    public void notifyObservers() {
        for (Investor investor : investors) {
            investor.update(this);
        }
    }
}

// Observer: 투자자(Investor)
interface Investor {
    void update(Stock stock);  // 주식 상태 업데이트
}

// Concrete Observer: 개별 투자자
class IndividualInvestor implements Investor {
    private String name;

    public IndividualInvestor(String name) {
        this.name = name;
    }

    @Override
    public void update(Stock stock) {
        System.out.println(name + "님, "
            + ((StockPrice) stock).getStockName()
            + " 주식의 새로운 가격: "
            + ((StockPrice) stock).getPrice());
    }
}

// 실행 예제
public class ObserverPatternExample {
    public static void main(String[] args) {
        // 주식 생성
        StockPrice appleStock = new StockPrice("Apple", 150.0);

        // 투자자 등록
        Investor investor1 = new IndividualInvestor("Alice");
        Investor investor2 = new IndividualInvestor("Bob");

        appleStock.registerObserver(investor1);
        appleStock.registerObserver(investor2);

        // 주식 가격 변경
        appleStock.setPrice(155.0);
        appleStock.setPrice(160.0);

        // 투자자 해지
        appleStock.removeObserver(investor1);

        // 가격 변경 후 남은 투자자에게만 알림
        appleStock.setPrice(165.0);
    }
}

```

- **실행 결과**
    
    ```
    Apple 주식 가격 변경: 150.0 → 155.0
    Alice님, Apple 주식의 새로운 가격: 155.0
    Bob님, Apple 주식의 새로운 가격: 155.0
    
    Apple 주식 가격 변경: 155.0 → 160.0
    Alice님, Apple 주식의 새로운 가격: 160.0
    Bob님, Apple 주식의 새로운 가격: 160.0
    
    Apple 주식 가격 변경: 160.0 → 165.0
    Bob님, Apple 주식의 새로운 가격: 165.0
    
    ```
    

### **장단점**

1. 장점
    - **느슨한 결합**: Subject와 Observer가 독립적으로 동작, 유지보수가 용이.
    - **유연성**: 주체와 옵저버를 쉽게 추가/제거할 수 있음.
    - **실시간 반응성**: 주체 상태 변화가 즉각적으로 알림을 통해 전파됨.
2. 단점
    - **성능 저하**: 옵저버가 많아질수록 알림 처리에 비용 증가.
    - **디버깅 어려움**: 많은 옵저버와 주체 간의 복잡한 의존 관계로 디버깅이 어려워질 수 있음.
    - **메모리 누수 위험**: 옵저버 해지를 제대로 관리하지 않으면 메모리 누수가 발생할 가능성이 있음.

### **실제 사용 사례**

1. **Java API**
    - `java.util.Observer`와 `java.util.Observable` 클래스를 사용한 구현.
    - ex) `Swing` UI 컴포넌트의 이벤트 리스너
2. **이벤트 기반 프로그래밍**
    - GUI 라이브러리에서 버튼 클릭, 텍스트 입력 등 이벤트 감지.
    - ex) JavaScript의 `addEventListener` 메서드.
3. **배포 및 구독 시스템**
    - 뉴스레터, SNS 알림 시스템 등

---

옵저버 패턴은 시스템 간의 느슨한 결합을 유지하며, 상태 변화를 효과적으로 전달하는 강력한 디자인 패턴이다.

특히 이벤트 기반의 시스템에서 필수적으로 사용되며, 다양한 실제 사례에 응용된다~