# 전략 패턴 (Strategy Pattern)

## **전략 패턴이란?**

- 전략 패턴(Strategy Pattern) : 알고리즘(전략)을 캡슐화하고, 실행 중에 이 알고리즘을 교체할 수 있게 설계하는 디자인 패턴
- **목적**: 동일한 작업을 수행하는 여러 알고리즘을 정의하고, 필요에 따라 교체하면서 유연성과 확장성을 제공한다.
- **사용 사례**
    - 결제 방식 (카드, 페이팔 등)
    - 데이터 압축 (ZIP, GZIP 등)
    - 인증 방법 (OAuth, JWT 등)

## **전략 패턴의 구조**

1. **Context**: 전략을 사용하는 환경(컨텍스트)
2. **Strategy Interface**: 알고리즘을 캡슐화 한 인터페이스
3. **Concrete Strategies**: `Strategy Interface`를 구현하여 구체적인 알고리즘을 정의

## **전략 패턴 코드 예제**

아래는 `배송비 계산`을 전략 패턴으로 구현했다.

```java
// 배송 전략 인터페이스
interface ShippingStrategy {
    double calculateShippingCost(double weight, double distance);
}

// 구체적인 배송 전략 1: 일반 배송
class StandardShipping implements ShippingStrategy {
    @Override
    public double calculateShippingCost(double weight, double distance) {
        return weight * 0.5 + distance * 0.3; // 단순 계산 로직
    }
}

// 구체적인 배송 전략 2: 빠른 배송
class ExpressShipping implements ShippingStrategy {
    @Override
    public double calculateShippingCost(double weight, double distance) {
        return weight * 0.8 + distance * 0.5; // 빠른 배송은 더 높은 요금
    }
}

// 구체적인 배송 전략 3: 해외 배송
class InternationalShipping implements ShippingStrategy {
    @Override
    public double calculateShippingCost(double weight, double distance) {
        return weight * 1.2 + distance * 0.7; // 해외 배송 추가 요금
    }
}

// Context: 배송 관리
class ShippingService {
    private ShippingStrategy strategy;

    // 전략 설정
    public void setShippingStrategy(ShippingStrategy strategy) {
        this.strategy = strategy;
    }

    // 배송비 계산
    public double calculateCost(double weight, double distance) {
        if (strategy == null) {
            throw new IllegalStateException("배송 전략이 설정되지 않았습니다.");
        }
        return strategy.calculateShippingCost(weight, distance);
    }
}

// 실행
public class StrategyPatternExample {
    public static void main(String[] args) {
        ShippingService service = new ShippingService();

        // 배송 정보
        double weight = 5.0; // kg
        double distance = 100.0; // km

        // 일반 배송 전략
        service.setShippingStrategy(new StandardShipping());
        System.out.println("Standard Shipping Cost: " + service.calculateCost(weight, distance));

        // 빠른 배송 전략
        service.setShippingStrategy(new ExpressShipping());
        System.out.println("Express Shipping Cost: " + service.calculateCost(weight, distance));

        // 해외 배송 전략
        service.setShippingStrategy(new InternationalShipping());
        System.out.println("International Shipping Cost: " + service.calculateCost(weight, distance));
    }
}

```

## **실행 결과**

```
Standard Shipping Cost: 40.0
Express Shipping Cost: 65.0
International Shipping Cost: 95.0
```

## **전략 패턴의 장단점**

### **장점**

1. **유연성**: 알고리즘을 쉽게 교체할 수 있음
2. **확장성**: 새로운 전략을 추가하더라도 기존 코드를 수정하지 않아도 됨(Open-Closed Principle)
3. **재사용성**: 동일한 전략을 여러 컨텍스트에서 재사용 가능

### **단점**

1. **복잡성 증가**: 전략과 컨텍스트를 분리하면서 클래스와 인터페이스가 증가
2. **전략 관리 문제**: 너무 많은 전략이 추가되면 관리가 어려워질 수 있음

## **추가적으로 알아두면 좋은 점**

1. **Java 내장 전략 패턴**
    - Java의 `Comparator` 인터페이스는 전략 패턴의 대표적인 예
    
    ```java
    Collections.sort(list, Comparator.comparing(String::length));
    ```
    
2. **실제 사용 사례**
    - 인증 라이브러리: `passport.js`, `Spring Security`에서의 인증 전략
    - 결제 시스템: 결제 방식(PayPal, 카드 등)을 유연하게 변경

---

전략 패턴은 알고리즘을 캡슐화하여 동적으로 변경할 수 있게 해주는 유용한 디자인 패턴이다. 이를 통해 애플리케이션의 유연성과 확장성을 높일 수 있으며, 특히 다양한 옵션을 제공해야 하는 시스템(결제, 배송, 인증 등)에 적합하다.!

<br>
<br>
<br>
<br>