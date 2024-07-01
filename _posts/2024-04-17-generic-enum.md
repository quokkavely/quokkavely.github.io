---
layout : single
title : "[JAVA] Generic,Enum"
categories: JAVA-Learn
tag : [JAVA, 개념]
toc : true
toc_sticky : true
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## getter와 setter

- getter와 setter 메서드를 활용하면 **데이터를 효과적으로 보호하면서도 의도하는 값으로 값을 변경**하여 캡슐화를 보다 효과적으로 달성
- getter-setter는 public이 default

**getter 메서드**

**→ 설정한 변수값을 읽어오는 데 사용**하는 메서드

→ **`get-`**을 메서드명 앞에 붙여서 사용

**setter 메서드**

→ 외부에서 메서드에 접근하여 조건에 맞을 경우 **데이터 값을 변경할 수 있게 해 줌**

→ 일반적으로 메서드명에 **`set-`**을 붙여서 정의

**예제**

```java
public class GetterSetterTest {
    public static void main(String[] args) {
        Worker w = new Worker();
        w.setName("김제리");
        w.setAge(30);
        w.setId(5);

        String name = w.getName();
        System.out.println("근로자의 이름은 " + name);
        int age = w.getAge();
        System.out.println("근로자의 나이는 " + age);
        int id = w.getId();
        System.out.println("근로자의 ID는 " + id);
    }
}

class Worker {
    private String name; // 변수의 은닉화. 외부로부터 접근 불가
    private int age;
    private int id;

    public String getName() { // 멤버변수의 값 
        return name;
    }

    public void setName(String name) { // 멤버변수의 값 변경
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if(age < 1) return;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
}

// 출력값
근로자의 이름은 김제리
근로자의 나이는 30
근로자의 ID는 5
```
<br/>

## 제네릭

1. 제네릭이란?
    - 타입을 구체적으로 지정하는 것이 아니라, 추후에 지정할 수 있도록 일반화해 두는 것을 의미
    - 즉, 작성한 클래스 또는 메서드의 코드가 특정 데이터 타입에 얽매이지 않게 해 둔 것
2. 제네릭 사용이유
    1. 기존의 방식
        
        ```java
        class Basket {
            private String item;
        
            Basket(String item) {
                this.item = item;
            }
        
            public String getItem() {
                return item;
            }
        
            public void setItem(String item) {
                this.item = item;
            }
        }
        ```
        
        - item은 String type 만 올 수 있음
    2. 제네릭 사용
        
        제네릭을 사용하면 **단 하나의 `Basket` 클래스만으로 모든 타입의 데이터를 저장할 수 있는 인스턴스를 만들 수 있음**
        
        ```java
        class Basket<T> {
            private T item;
        
            public Basket(T item) {
                this.item = item;
            }
        
            public T getItem() {
                return item;
            }
        
            public void setItem(T item) {
                this.item = item;
            }
        }
        ```
        
        - Basket class의 인스턴스화
            
            ```java
            Basket<String> basket1 = new Basket<String>("item");
            ```
            
            →위 코드를 실행하면 `Basket` 클래스 내부의 `T`가 모두 `String`으로 치환
            
            → 만약 **원시타입으로 치환시 wrapper class로 치환**됨
            
            Ex) Integer, Double과 같은 래퍼 클래스
            

### 제네릭 클래스 사용법

1. 제네릭 클래스
    
    ```java
    class Basket<T> {
    
    }
    ```
    
    - `T`를 **타입 매개변수**
    - `<T>`와 같이 꺾쇠 안에 넣어 클래스 이름 옆에 작성해 줌으로써 클래스 내부에서 사용할 타입 매개변수를 선언가능

1. 만약 타입 매개변수를 여러 개 사용할 경우
    
    ```java
    class Basket<K, V> { ... }
    ```
    
    - `T`, `K`, `V`는 각각 Type, Key, Value의 첫 글자를 따온 것

1. 제네릭클래스 주의사항
    
    **클래스 변수(static)에는 타입 매개변수를 사용불가**
    
    - 클래스 변수는 모든 인스턴스가 공유하는 변수 ⇒ 클래스 변수의 타입이 인스턴스 별로 달라지게 됨.
    
2. **타입 매개변수에 치환될 타입으로 기본 타입을 지정불가**
    1. int, double과 같은 원시 타입을 지정해야 하는 맥락에서는 Integer, Double과 같은 래퍼 클래스를 활용
        
        ```java
        Basket<String>  basket1 = new Basket<String>("Hello");
        Basket<Integer> basket2 = new Basket<Integer>(10);
        Basket<Double>  basket3 = new Basket<Double>(3.14);
        
        ----
        // new Basket<…>은 아래와 같이 구체적인 타입을 생략하고 작성가능
        // => 참조변수의 타입으로부터 유추할 수 있기 때문
        Basket<String>  basket1 = new Basket<>("Hello");
        Basket<Integer> basket2 = new Basket<>(10);
        Basket<Double>  basket2 = new Basket<>(3.14);
        ```
        
    
3. 다형성 적용 가능
    
    ```java
    class Flower { ... }
    class Rose extends Flower { ... }
    class RosePasta { ... }
    
    class Basket<T> {
        private T item;
    
        public T getItem() {
            return item;
        }
    
        public void setItem(T item) {
            this.item = item;
        }
    }
    
    class Main {
        public static void main(String[] args) {
            Basket<Flower> flowerBasket = new Basket<>();
            flowerBasket.setItem(new Rose());      // 다형성 적용
            flowerBasket.setItem(new RosePasta()); // 에러
        }
    }
    ```
    
    - Main 클래스의 main method에서 flowerBasket은 Basket의 타입으로 객체 Flower로 지정하여 객체 생성
    - new Rose()를 통해 생성된 객체는 Rose 타입이며, Rose 클래스는 Flower 클래스를 상속받고 있으므로, flowerBasket 의 item에 할당가능
    - 반면, new RosePasta()를 통해 생성된 인스턴스는 RosePasta 타입이며, RosePasta 클래스는 Flower 클래스와 아무런 관계가 없으므로 할당 불가.
    
    > 즉,  **`Basket` 클래스를 인스턴스화할 때 타입으로 `Flower` 클래스의 하위 클래스만 지정하도록 제한**
    > 
    
4. 특정 인터페이스를 구현한 클래스만 타입으로 지정할 수 있도록 제한가
    
    ```java
    interface Plant { ... }
    class Flower implements Plant { ... }
    class Rose extends Flower implements Plant { ... }
    
    class Basket<T extends Plant> {
        private T item;
    
    		...
    }
    
    class Main {
        public static void main(String[] args) {
    
            // 인스턴스화
            Basket<Flower> flowerBasket = new Basket<>();
            Basket<Rose> roseBasket = new Basket<>();
        }
    }
    ```
    
    - 특정 클래스를 상속받으면서 동시에 특정 인터페이스를 구현한 클래스만 타입으로 지정할 수 있도록 제한하려면 아래와 같이 `&`를 사용 ⇒ 이 경우에는 클래스를 인터페이스 앞에 위치 시켜야 함.
        
        ```java
        class Basket<T extends **Flower** & Plant> { // (1)
            private T item;
            		...
        }
        ```
        

### 제네릭메서드

1. 클래스 내부의 특정 메서드만 제네릭으로 선언가능
    
    ```java
    class Basket {
    		...
    		public <T> void add(T element) {
    				...
    		}
    }
    ```
    
    - 제네릭 메서드의 타입 매개 변수 선언은 반환타입 앞에서 이루어지며, 해당 메서드 내에서만 선언한 타입 매개 변수를 사용가능

1. **제네릭 메서드의 타입 매개 변수는 제네릭 클래스의 타입 매개 변수와 별개의 것**
    
    ```java
    class Basket<T> { // 1 : 여기에서 선언한 타입 매개 변수 T와
    		...
    		public <T> void add(T element) { // 2 : 여기에서 선언한 타입 매개 변수 T는 서로 다른 것입니다.
    				...
    		}
    }
    ```
    
    - **클래스명 옆에서 선언한 타입 매개 변수는 클래스가 인스턴스화될 때 타입이 지정됨**
    - **제네릭 메서드의 타입 지정은 메서드가 호출될 때 이루어짐.**
        
        ```java
        Basket<String> basket = new Bakset<>(); 
        // 위 예제의 1의 T가 String으로 지정됩니다.
        basket.<Integer>add(10);                
        // 위 예제의 2의 T가 Integer로 지정됩니다.
        basket.add(10);                         
        // 타입 지정을 생략할 수도 있습니다.
        ```
        
    
2.  `static` 메서드에서도 선언하여 사용가능
    
    클래스 타입 매개 변수와 달리 메서드 타입 매개 변수는 `static` 메서드에서도 선언하여 사용가능
    
    ```java
    class Basket {
    		...
    		static <T> int setPrice(T element) {
    				...
    		}
    }
    ```
    
3.  `String` 클래스의 메서드는 제네릭 메서드를 정의하는 시점에 사용불가 but, Object 클래스는 가능
    1. 메서드가 호출되는 시점에서 제네릭 타입이 결정되므로, 제네릭 메서드를 정의하는 시점에서는 실제 어떤 타입이 입력되는지 알 수 없기 때문에 String 클래스의 메서드는 사용 불가
    2. 모든 자바 클래스의 최상위 클래스인 `Object` 클래스의 메서드는 사용 가능
        
        Ex) `equals()`, `toString()` 등이 `Object` 클래스의 메서드
        
    

### 와일드카드

 **어떠한 타입으로든 대체될 수 있는** 타입 파라미터, 기호`?`로 와일드카드를 사용가능

1. 일반적으로 와일드카드는 `extends`와 `super` 키워드를 조합하여 사용
    
    ```java
    <? extends T><? super T>
    ```
    
    - **<? extends T>**는 와일드카드에 상한 제한을 두는 것으로서, T와 T를 상속받는 하위 클래스 타입만 타입 파라미터로 받을 수 있도록 지정
    - **<? extends T>**는 와일드카드에 상한 제한을 두는 것으로서, T와 T를 상속받는 하위 클래스 타입만 타입 파라미터로 받을 수 있도록 지정
    
    - 반면, **<? super T>**는 와일드카드에 하한 제한을 두는 것으로, T와 T의 상위 클래스만 타입 파라미터로 받도록 함
    - 
2. extends 및 super 키워드와 조합하지 않은 와일드카드(<?>)는 <? extends Object>
    - 모든 클래스 타입은 Object 클래스를 상속받으므로, 모든 클래스 타입을 타입 파라미터로 받을 수 있음을 의미
3. 예제
    1. 사용할 클래스 정의 - 상속계층도
        
        ```java
        class Phone {}
        
        class IPhone extends Phone {}
        class Galaxy extends Phone {}
        
        class IPhone12Pro extends IPhone {}
        class IPhoneXS extends IPhone {}
        
        class S22 extends Galaxy {}
        class ZFlip3 extends Galaxy {}
        
        class User<T> {
        		public T phone;
        
        		public User(T phone) {
        				this.phone = phone;
        		}
        }
        ```
        
    2. 휴대전화별 기능별 분류
        
        ```java
        class PhoneFunction {
            public static void call(User<? extends Phone> user) {
        			System.out.println("user.phone = " + user.phone.getClass().getSimpleName());
              System.out.println("모든 Phone은 통화를 할 수 있습니다.");
            }
        
            public static void faceId(User<? extends IPhone> user) {
        			System.out.println("IPhone만 Face ID를 사용할 수 있습니다. ");
            }
        
            public static void samsungPay(User<? extends Galaxy> user) {
        	    System.out.println("Galaxy만 삼성 페이를 사용할 수 있습니다. ");
            }
        
            public static void recordVoice(User<? super Galaxy> user) {
          
            }
        }
        ```
        
    3. 메서드 호출
        
        ```java
        public class Example {
        	public static void main(String[] args) {
         // PhoneFunction.faceId(new User<Phone>(new Phone())); // X
            PhoneFunction.faceId(new User<IPhone>(new IPhone()));
            PhoneFunction.faceId(new User<IPhone15Pro>(new IPhone15Pro
        	  PhoneFunction.recordVoice(new User<Phone>(new Phone()));
        //  PhoneFunction.recordVoice(new User<IPhone>(new IPhone()));
        //  PhoneFunction.recordVoice(new User<S24>(new S24())); // X
        //  PhoneFunction.recordVoice(new User<ZFlip5>(new ZFlip5())); // X
        ```
        
        - 주석 처리는 에러 발생 하는 부분
            - faceId는 <? extends IPhone> 타입이 와야 하므로 Iphone의 하위클래스만 가능, 그래서 상위클래스인 phone은 올 수 없음
            - recordVoice 메서드는 (User<? super Galaxy> user) ⇒ 갤럭시의 상위클래스의 타입만 지정할 수 있음, 그래서 ZFlip5와 S24는 올 수 없고 Phone만 가능.
        

## Enum

- **열거형(enum; enumerated type)**은 여러 상수들을 보다 편리하게 선언 할 수 있는 자바의 문법요소 = 서로 연관된 상수들의 집합을 의미
- 서로 관련 있는 내용들을 모아 한 번에 간편하게 관리할 때 사용 ⇒ 실무에서 많이 쓰임

### Enum의 필요성

1. 기존에 상수는  `public static final`을 사용시 단점
    1. 상수 명이 중복되는 경우가 발생
        
        ```java
        public static final int SPRING = 1;
        public static final int SUMMER = 2;
        public static final int FALL   = 3;
        public static final int WINTER = 4;
        
        public static final int DJANGO  = 1;
        public static final int SPRING  = 2; // 계절의 SPRING과 중복 발생!
        public static final int NEST    = 3;
        public static final int EXPRESS = 4;
        ```
        
    2. 위 문제를 해결하기 위해 서로 다른 객체로 만들어 주어야 함.
        
        ```java
        class Seasons {
            public static final Seasons SPRING = new Seasons();
            public static final Seasons SUMMER = new Seasons();
            public static final Seasons FALL   = new Seasons();
            public static final Seasons WINTER = new Seasons();
        }
        
        class Frameworks {
            public static final Frameworks DJANGO  = new Frameworks();
            public static final Frameworks SPRING  = new Frameworks();
            public static final Frameworks NEST    = new Frameworks();
            public static final Frameworks EXPRESS = new Frameworks();
        }
        ```
        
        - but, 코드가 길어지고 사용자 정의 타입이기 때문에 switch문에 활용할 수 없다는 문제 발생
2. Enum 사용 이점
    
    ```java
    enum Seasons { SPRING, SUMMER, FALL, WINTER }
    enum Frameworks { DJANGO, SPRING, NEST, EXPRESS }
    ```
    
    1) switch문 사용 가능
    
    ```java
    enum Seasons {SPRING, SUMMER, FALL, WINTER}
    
    public class Main {
        public static void main(String[] args) {
            Seasons seasons = Seasons.SPRING;
            switch (seasons) {
                case SPRING:
                    System.out.println("봄");
                    break;
                case SUMMER:
                    System.out.println("여름");
                    break;
                case FALL:
                    System.out.println("가을");
                    break;
                case WINTER:
                    System.out.println("겨울");
                    break;
            }
        }
    }
    
    //출력값
    봄
    ```
    
    2) **여러 상수들을 보다 편리하게 선언하고 관리 가능**
    
    **3) 상수 명의 중복을 피하고, 타입에 대한 안정성을 보장**
    

### Enum 사용법

1. 열거형 정의
    
    ```java
    enum 열거형이름 { 상수명1, 상수명2, 상수명3, ...}
    ```
    
    - 상수는 대소문자로 모두 작성이 가능하지만, **관례적으로 대문자로 작성**
    - 따로 값을 지정해주지 않아도 **자동적으로 0부터 시작하는 정수값이 할당**되어 각각의 상수를 가리키게 됨 (아래 예제 -1 참고)
    - 열거형에 선언된 상수에 접근하는 방법은 열거형이름.상수명
    - 변경되지 않는 한정적인 데이터들을 효과적으로 관리가능
    - 열거형에서 사용할 수 있는 메서드 (최상위 클래스 Object에 정의된 메서드들을 사용할 수 있었던 것과 동일)
        - `java.lang.Enum` 에 정의
            
            
            | 리턴 타입 | 메서드(매개변수) | 설명 |
            | --- | --- | --- |
            | String | name() | 열거 객체가 가지고 있는 문자열을 리턴하며, 리턴되는 문자열은 열거타입을 정의할 때 사용한 상수 이름과 동일합니다. |
            | int | ordinal() | 열거 객체의 순번(0부터 시작)을 리턴합니다. |
            | int | compareTo(비교값) | 주어진 매개 값과 비교해서 순번 차이를 리턴합니다. |
            | 열거 타입 | valueOf(String name) | 주어진 문자열의 열거 객체를 리턴합니다. |
            | 열거 배열 | values() | 모든 열거 객체들을 배열로 리턴합니다. |
2. 예제-1
    
    ```java
    enum Seasons {
        SPRING, //정수값 0 할당
        SUMMER,  //정수값 1 할당
        FALL, //정수값 2 할당
        WINTER //정수값 3 할당
    }
    
    public class EnumExample {
        public static void main(String[] args) {
            System.out.println(Seasons.SPRING); // SPRING
        
    ```
    
    - `Seasons`라는 이름의 열거형은 `SPRING, SUMMER, FALL, WINTER`는 총 네 개의 열거 객체를 포함
3. 예제 -2
    
    ```java
    enum Level {
      LOW, // 0
      MEDIUM, // 1
      HIGH // 2
    }
    
    public class EnumTest {
        public static void main(String[] args) {
            Level level = Level.MEDIUM;
    
            Level[] allLevels = Level.values();
            for(Level x : allLevels) {
                System.out.printf("%s=%d%n", x.name(), x.ordinal());
            }
    
            Level findLevel = Level.valueOf("LOW");
            System.out.println(findLevel);
            System.out.println(Level.LOW == Level.valueOf("LOW"));
    
            switch(level) {
                case LOW:
                    System.out.println("낮은 레벨");
                    break;
                case MEDIUM:
                    System.out.println("중간 레벨");
                    break;
                case HIGH:
                    System.out.println("높은 레벨");
                    break;
            }
        }
    }
    
    //출력값
    LOW=0
    MEDIUM=1
    HIGH=2
    LOW
    true
    중간 레벨
    ```
    
    - 향상된 for문과 열거형의 최상위 클래스로부터 확장된 `name()`과 `ordinal()`을 사용하여 각각 이름과 순서를 출력값으로 반환
    - `valueOf()` 메서드를 활용하여 지정된 열거형에서 이름과 일치하는 열거형 상수를 반환
    - 반환된 상수가 의도했던 상수와 일치하는지 여부를 불리언 값으로 확인
    - switch문에서 level은 위에서  Levle.MEDIUM으로 초기화 되었고 case MEDIUM이 실행되어 중간레벨이 출력됨, 맨 윗줄에 해당하는  Level level = Level.MEDIUM; 이 가장 늦게 출력되는 건 switch문이 프로그램 제일 마지막에 있기 때문

<br/>
<br/>

### COMMENT

오늘은 뭔가 실무에서 사용하기 좋은 것들에 대해 배웠다.
제네릭은 자바의 꽃이라고 한다던데
타입을 지정하지 않는 것은 이점이 확실한 듯.
타입을 지정하지 않고 임의의 타입으로 지정할 수 있는 것인데
모든 타입을 허용하는 것 같지만 타입을 제한할 때 자주 사용한다고 하니
유의해서 사용해봐야겠다.


지금 당장은 많이 사용하기가 낯설어서
활용한 코드도 많이보고
프로그램을 만들때 사용해서 자주 볼 수 있으면 좋을 것 같다.



<br/>

---

<br/>
<br/>
<br/>
<br/>
<br/>