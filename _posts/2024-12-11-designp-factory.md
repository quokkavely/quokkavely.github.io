---
layout : single
title : "[디자인패턴] Factory Pattern"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 팩토리 패턴 (Factory Pattern)

팩토리 패턴은 객체 생성 로직을 캡슐화하여 상위 클래스와 하위 클래스 간의 결합도를 줄이고, 코드의 **유연성**과 **유지보수성**을 높이는 데 중점을 둔 설계 패턴이다. 이 패턴은 **상위 클래스**가 프로그램의 뼈대를 결정하고, **하위 클래스**가 객체 생성의 구체적인 내용을 책임진다.

## 팩토리 패턴의 특징

1. **객체 생성 책임 분리**
    - 상위 클래스는 객체 생성 방법에 대해 몰라도 된다.
    - 객체 생성의 구체적인 로직은 하위 클래스 또는 팩토리 클래스가 처리한다.
2. **유연한 확장성**
    - 새로운 타입의 객체를 추가할 때, 기존 코드를 수정하지 않고도 확장 가능하다.
    - 유지보수성과 확장성이 뛰어나다.
3. **결합도 감소**
    - 객체 생성 로직을 캡슐화하여 상위 클래스와 하위 클래스 간의 의존성을 줄인다.

## 팩토리 패턴의 구조

1. **추상 클래스(또는 인터페이스)**: 객체 생성에 필요한 공통적인 메서드를 정의한다.
2. **구체 클래스**: 객체의 구체적인 타입과 생성 로직을 구현한다.
3. **팩토리 클래스**: 클라이언트 코드가 직접 객체를 생성하지 않고, 객체 생성을 대신 처리한다.

## 팩토리 패턴의 장점

1. **유지보수성 증가**
    - 객체 생성 로직이 팩토리 클래스에 집중되어 있으므로, 새로운 객체를 추가하거나 생성 방식을 수정할 때 변경이 용이하다.
2. **코드의 단순화**
    - 클라이언트 코드에서 객체 생성 로직이 제거되어, 코드가 간결해진다.
3. **확장성**
    - 새로운 객체 타입을 추가해도 기존 코드를 거의 수정하지 않아도 된다.

## 팩토리 패턴의 단점

1. **팩토리 클래스의 복잡성 증가**
    - 객체의 종류가 많아질수록 팩토리 클래스가 복잡해질 수 있다.
2. **추가적인 클래스 필요**
    - 추상 클래스와 팩토리 클래스 등 설계의 유연성을 확보하기 위해 코드 구조가 다소 복잡해질 수 있다.

## 팩토리 패턴의 확장: Factory Method vs Abstract Factory

1. **Factory Method**
    - 객체 생성의 책임을 하위 클래스가 담당한다. 팩토리 메서드를 하위 클래스에서 오버라이딩하여 필요한 객체를 생성한다.
2. **Abstract Factory**
    - 관련 객체군을 생성하는 팩토리를 제공한다. 서로 관련된 객체를 그룹화하여 생성할 때 유용하다.

## 팩토리 패턴 구현 예제

게임에서 플레이어가 이동할 때마다 다양한 몬스터를 만나게 된다고 가정

몬스터에는 여러 타입이 있고, 각 타입마다 다른 속성과 동작을 가진다.

- **Goblin**: 기본적인 근접 공격을 하는 몬스터.
- **Dragon**: 화염 공격을 사용하는 강력한 몬스터.
- **Slime**: 느리지만 여러 개로 분열하는 몬스터.

플레이어가 위치에 따라 특정 몬스터를 만날 수 있도록 **팩토리 패턴**으로 몬스터 생성 로직을 구현

### 1. 몬스터 타입 정의

몬스터의 타입을 Enum으로 정의한다.

```java
enum MonsterType {
    GOBLIN,
    DRAGON,
    SLIME
}
```

### 2. 몬스터의 공통 인터페이스 정의

모든 몬스터가 가져야 할 공통 속성과 동작을 추상 클래스나 인터페이스로 정의한다.

```java
abstract class Monster {
    protected String name;

    public String getName() {
        return name;
    }

    public abstract void attack();
```

### 3. 구체적인 몬스터 클래스

각 몬스터 타입별로 상속받아 구체적인 동작과 속성을 구현한다.

```java
class Goblin extends Monster {
    public Goblin() {
        name = "Goblin";
    }

    @Override
    public void attack() {
        System.out.println(name + " attacks with a club!");
    }
}

class Dragon extends Monster {
    public Dragon() {
        name = "Dragon";
    }

    @Override
    public void attack() {
        System.out.println(name + " breathes fire!");
    }
}

class Slime extends Monster {
    public Slime() {
        name = "Slime";
    }

    @Override
    public void attack() {
        System.out.println(name + " splits into smaller slimes!");
    }
}
```

### 4. 팩토리 클래스

팩토리 클래스에서 요청받은 타입에 따라 적절한 몬스터 객체를 생성한다.

```java
class MonsterFactory {
    public static Monster createMonster(MonsterType type) {
        switch (type) {
            case GOBLIN:
                return new Goblin();
            case DRAGON:
                return new Dragon();
            case SLIME:
                return new Slime();
            default:
                throw new IllegalArgumentException("Unknown monster type: " + type);
        }
    }
}
```

### 5. 클라이언트 코드

클라이언트는 팩토리 클래스를 통해 몬스터를 생성하고, 객체 생성 로직에 신경 쓰지 않고도 몬스터를 사용할 수 있다.

```java
public class Main {
    public static void main(String[] args) {
        // Goblin 생성
        Monster goblin = MonsterFactory.createMonster(MonsterType.GOBLIN);
        goblin.attack(); // Goblin attacks with a club!

        // Dragon 생성
        Monster dragon = MonsterFactory.createMonster(MonsterType.DRAGON);
        dragon.attack(); // Dragon breathes fire!

        // Slime 생성
        Monster slime = MonsterFactory.createMonster(MonsterType.SLIME);
        slime.attack(); // Slime splits into smaller slimes!
    }
}
```

### 코드 설명

1. **확장성**
    
    새로운 몬스터 타입을 추가할 때, 기존 코드를 수정할 필요 없이 새로운 클래스를 추가하고 팩토리 클래스에 해당 타입을 등록하면 된다.
    
2. **유지보수성**
    
    객체 생성 로직이 팩토리 클래스에 집중되어 있으므로, 생성 로직을 수정하거나 변경할 때 코드의 다른 부분에 영향을 미치지 않는다.
    
3. **단일 책임 원칙**
    
    몬스터 생성 로직과 몬스터의 동작을 분리하여 코드의 책임이 명확해졌다.
    

## 팩토리 패턴의 활용 사례

1. **프레임워크**
    - 스프링(Spring) 프레임워크의 Bean Factory가 대표적인 예다. 객체 생성과 의존성 주입을 관리한다.
2. **GUI 라이브러리**
    - GUI 애플리케이션에서 다양한 버튼, 창 등을 생성할 때 팩토리 패턴이 사용된다.
3. **게임 개발**
    - 다양한 캐릭터나 아이템을 동적으로 생성할 때 팩토리 패턴을 활용한다.


---

팩토리 패턴은 객체 생성 로직을 캡슐화하여 코드의 유연성과 유지보수성을 높이는 데 유용하다.

특히, 새로운 객체를 추가할 때 기존 코드를 수정하지 않아도 되므로 확장성이 중요한 애플리케이션에서 널리 사용된다.

팩토리 패턴을 이해하고 활용하면, 객체 생성의 책임을 분리하여 더 깔끔하고 유지보수 가능한 코드를 작성할 수 있다.

<br>
<br>
<br>
<br>
