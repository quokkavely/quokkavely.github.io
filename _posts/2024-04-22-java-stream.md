---
layout : single
title : "[JAVA] 람다, 스트림, Optional 클래스"
categories: JAVA-Learn
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: true
---

## 람다

1. 람다식
    1. 함수형 프로그래밍 기법을 지원하는 자바의 문법요소
    2. 메서드를 하나의 식으로 표현한 것, 코드를 매우 간결하면서 명확하게 표현
    3. 익명의 객체 

1. 람다식의 기본문법
    1. 반환타입과 이름 생략 가능
       
        ```java
        int sum (int num1, int num2){
        	return num1+num2;
        }
        --------------------람다식으로 변경
        
        (int num1, int num2) -> {return num1+num2;}
        ```
        
    2. 특정 조건이 충족되면 람다식을 더욱 축약해서 사용 가능.
        1. 메서드 바디에 실행문이 하나만 존재할 때 중괄호와 return 생략 가능 + 세미콜론 생략
           
            ```java
            (int num1, int num2) -> num1+num2
            ```
            
        2. 매개변수 타입을 함수형 인터페이스를 통해 유추할 수 있는 경우 매개변수 타입 생략가능
           
            ```java
            (num1,num2)-> num1+num2
            ```
            
    
2. 함수형 인터페이스
    1. **람다식은 익명객체**
        1. 익명 객체는 익명클래스를 통해 만들 수 있는데
        2. 익명 클래스란 ***객체의 선언과 생성을 동시에 하여 오직 하나의 객체를 생성하고 단 한번만 생성되는 일회용 클래스***
        3. 람다식이 객체라면 참조변수를 통해 접근 가능해야하지만 사용할 수 없음
        4. 이를 해결하기 위한 자바의 문법요소가 **함수형 인터페이스.**
    2. 함수형 인터페이스
        1. 기존의 인터페이스 문법을 활용하여 람다식을 다루는 것.
        2. 람다식과 인터페이스의 메서드가 1:1로 매칭되어야 함. = 단 하나의 추상 메서드만 선언된 인터페이스
        3. 함수형 인터페이스 타입의 참조변수로 람다식 참조할 수 있음.
        
        ```java
        public class LamdaExample1{
        	public static void main(String[]args){
        		/*Object obj = new Object(){
        			int sum(int num1, int num2){
        				return num1+num2;
        				}
        			};
        			*/
        		ExampleFunction exampleFunction = (num1,num2) ->num1+num2;
        		System.out.println(exampleFunction.sum(10,15));
        	}
        	
        	
        @FunctionalInterface // 컴파일러가 인터페이스가 바르게 정의되었는지 확인
        	interface ExampleFunction{
        		int sum(int num1, int num2);
        		}
        		
        
        //출력값 : 25
        ```
        
    3. 매개변수와 리턴값이 없는 람다식
        1. 예제-1
           
            ```java
            @FunctionalInterface
            public interface MyFunctionalInterface{
            	void accept();
            	}
            ```
            
            ```java
            MyFuctionalInterface example = ()->{...};
            //example.accept();
            
            람다식이 대입된 인터페이스의 참조 변수는 위의 주석과 같이 
            accept()를 호출할 수 있습니다.accept()의 호출은 
            람다식의 중괄호 {}를 실행시킵니다.
            ```
            
        2. 예제 -2
           
            ```java
            @FunctionalInterface
            interface MyFuctionalInterface{
            	void accept();
            	}
            ```
            
            ```java
            pulbic class MyFuctionalInterfaceExample{
            	public static void main(String[]args)throws Exception{
            		MyFuctionalInterface exmample = () -> System.out.println("accept()호출");
            		
            		example.aceept();
            		}
            	}
            //출력값 : accept() 호출	
            ```
            
        
    4. 매개변수와 리턴값이 있는 람다식
        1. 예제-1
           
            ```java
            @FunctionalInterface
            public interface MyFunctionalInterface {
                void accept(int x);
            }
            ```
            
            ```java
            public class MyFunctionalInterfaceExample {
                public static void main(String[] args) throws Exception {
                    MyFunctionalInterface example;
                    example = (x) -> {
                        int result = x * 5;
                        System.out.println(result);
                    };
                    example.accept(2);
                    example = (x) -> System.out.println(x * 5);
                    example.accept(2);
                }
            }
            
            // 출력값
            10
            10
            ```
            
            람다식이 대입된 인터페이스 참조 변수는 다음과 같이 `accept()`를 호출할 수 있습니다. 위의 예시와 같이 매개값으로 2를 주면 람다식의 x 변수에 2가 대입되고, x는 중괄호 { }에서 사용됩니다.
        
    5. 리턴값이 있는 람다식
        1. 예제-1
           
            ```java
            @FunctionalInterface
            public interface MyFunctionalInterface {
                int accept(int x, int y);
            }
            ```
            
            ```java
            public class MyFunctionalInterfaceExample {
                public static void main(String[] args) throws Exception {
                    MyFunctionalInterface example;
            
                    example = (x, y) -> {
                        int result = x + y;
                        return result;
                    };
                    int result1 = example.accept(2, 5);
                    System.out.println(result1);
            
                    example = (x, y) -> { return x + y; };
                    int result2 = example.accept(2, 5);
                    System.out.println(result2);
            
                    example = (x, y) ->  x + y;
                    //return문만 있으면, 중괄호 {}와 return문 생략 가능
                    int result3 = example.accept(2, 5);
                    System.out.println(result3);
            
                    example = (x, y) -> sum(x, y);
                    //return문만 있으면, 중괄호 {}와 return문 생략 가능
                    int result4 = example.accept(2, 5);
                    System.out.println(result4);
                }
                public static int sum(int x, int y){
                    return x + y;
                }
            }
            
            //출력값
            7
            7
            7
            7
            ```
    
3. 메서드 레퍼런스
    1. 메서드레퍼런스
        1. 불필요한 매개변수 제거할때 주로 사용
        2. 람다식을 더욱더 간단하게 사용하고 싶은 개발자의 요구에 의해 반영된 산물
    2. static Method와 인스턴스 Method reference
        1. 클래스::메서드
           
            ```java
              Integer method(String s) {  // 그저 Integer.parseInt(String s)만 호출 
                  return Integer.parseInt(s);
              }
            
            	Function<String, Integer> f = (String s) -> Integer.parseInt(s);
            	Function<String, Integer> f = Integer::parseInt;  // 메서드 참조
            
            ```
            
        2. 참조변수 ::메서드
           
            ```java
            BiFunction<String,String,Boolean> f = (s1, s2) -> s1.equals(s2);
            BiFunction<String,String,Boolean> f = String::equals;
            ```
            
        3. 예
        
        ```java
        //Calculator.java
        pulbic class Calculator{
        	public static int staticMethod(int x, int y){
        		return x + y;
        }
        	public int instanceMethod(int x, int y){
        		return x * y;
        	}
        }
        ```
        
        ```java
        import java util.fuction.IntBinaryOpoerator;
        
        public class MethodReference{
        	public static void main(String[]args)throws Exception{
        	IntBinaryOperator operator;
        	
        	/*static 메서드 
        		클래스이름 :: 메서드이름
        		*/
        		
        	operator = Calculator::staticMethod;
        	System.out.println("정적메서드 결과 :"+operator.applyasInt(3,5));
        	
        	/*인스턴스 메서드
        	인스턴스명:메서드명
        	*/
        	Caculator calculator = new Calcuator();
        	operator = calculator::instanceMethod;
        	System.out.println("인스턴스메서드 결과:"+operator.applyAsInt(3,5));
        	}
        }
        ```
    
4. 메서드 레퍼런스는 생성자 레퍼런스도 포함 > 객체생성
    1. (a,b) → new 클래스(a,b)
    2. 클래스::new
    3. 예제
       
        ```java
        //Member.java
        public class Member{
        	private String name;
        	private String id;
        	
        	public Member(){
        	 System.out.println("Member()실행");
        	 }
        	public Member(String name, String id){
        	System.out,println("Member(String name, String id)실행");
        	this.id = id;
        	this.name=name;
        	}
        	
        	public String getName(){
        		return name;
        	}
        	public String getId(){
        		return id;
        		}
        	}
        	
        	 
        ```
        
        ```java
        import java.util.function.Bifuction;
        import java.util.function.Fuction;
        
        public class ConstructorRef{
        	public static void main(String[]args)throws Exception{
        	Fuction<String,Member>fuction1= Member::new;
        	Member member1 = fuction1.apply("kimcoding");
        	
        	BiFuction<String,String,Member>function2 = Member::new;
        	Member member2 = fuction2.apply("kimcoding","김코딩");
        	}
        }
        
        /*
        Member(String id) 실행
        Member(String name, String id) 실행
        */
        
        ```
    
5. java.util.fuction 패키지
    1. **자주 사용되는 다양한 함수형 인터페이스를 제공.** 
       
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled.png)
        
    2. 매개변수가 2개인 함수형 인터페이
       
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%201.png)
        
    3.  **매개변수의 타입과 반환타입이 일치하는 함수형 인터페이스**   
        
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%202.png)
        
    4. **함수형 인터페이스를 사용하는 컬렉션 프레임웍의** **메서드**   
       
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%203.png)
        
    5. **기본형을 사용하는 함수형 인터페이스**   
       
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%204.png)
        
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%205.png)
        
    6. 



## 스트림

1. 배열, 컬렉션의 저장 요소를 **하나씩 참조해서 람다식으로 처리할 수 있도록 해주는 반복자**
2. List, Set, Map, 배열 등 **다양한 데이터 소스로부터 스트림**을 만들 수 있고, 이를 **표준화된 방법**으로 다룰 수 있음
3. 기존의 반복문과 비교 
    1. Iterator
    2. Stream
4. **선언형 프로그래밍(Declarative Programming)** 방식 ↔ 명령형프로그래밍 방식
    1. 명령형 프로그래밍 방식 - 어떻게 코드를 작성할지에 대한 내용에 초점
       
        ```java
        import java.util.List;
        public class ImperativeProgramming {
            public static void main(String[] args){
                // List에 있는 숫자 중에서 4보다 큰 짝수의 합계 구하기
                List<Integer> numbers = List.of(1, 3, 6, 7, 8, 11);
                int sum = 0;
        
                for(int number : numbers){
                    if(number > 4 && (number % 2 == 0)){
                        sum += number;
                    }
                }
        
                System.out.println("명령형 프로그래밍을 사용한 합계 : " + sum);
            }
        }
        
        //출력값
        명령형 프로그래밍을 사용한 합계 : 14
        ```
        
    2. 선언형 프로그래밍 방식 - 어떻게가 아닌 무엇에 집중, 
       
        ```java
        import java.util.List;
        
        public class DeclarativePrograming {
            public static void main(String[] args){
                // List에 있는 숫자들 중에서 4보다 큰 짝수의 합계 구하기
                List<Integer> numbers = List.of(1, 3, 6, 7, 8, 11);
        
                int sum =
                        numbers.stream()
                                .filter(number -> number > 4 && (number % 2 == 0))
                                .mapToInt(number -> number)
                                .sum();
        
                System.out.println("선언형 프로그래밍을 사용한 합계 : " + sum);
            }
        }
        
        //출력값
        선언형 프로그래밍을 사용한 합계 : 14
        ```
    
5. **데이터 소스가 무엇이냐에 관계없이 같은 방식으로 데이터를 가공/처리가능**
6. 즉, 배열이냐 컬렉션이냐에 관계없이 **하나의 통합된 방식으로 데이터를 다룰 수 있게 되었다는 뜻**

## 스트림의 특징

1. **스트림 처리 과정은 생성, 중간 연산, 최종 연산 *세 단계의 파이프라인*으로 구성될 수 있다.**
   
    ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%206.png)
    
    ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%207.png)
    
2. **스트림은 원본 데이터 소스를 변경하지 않는다(read-only).**
    1. 오직 데이터를 읽어올 수 있고, 데이터에 대한 변경과 처리는 생성된 스트림 안에서만 수행
    2. 원본 데이터가 스트림에 의해 임의로 변경되거나 데이터가 손상되는 일을 방지
3. **스트림은 일회용이다(onetime-only).**
    1. 스트림이 생성되고 여러 중간 연산을 거쳐 마지막 연산이 수행되고 난 후에는 스트림은 닫히고 다시 사용할 수 없음.
    2. 추가적인 작업이 필요하다면, 다시 스트림을 생성
4. **스트림은 내부 반복자이다.**
    1. ↔외부반복자 : for/Iterator/while문, 개발자가 코드로 직접 컬렉션의 요소를 반복해서 가져오는 코드 패턴
    2. 내부반복자 : 컬렉션 내부에 데이터 요소 처리 방법(람다식)을 주입해서 요소를 반복처리
       
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%208.png)
        
    3. 

**`스트림 생성`**

1. 배열 Stream<타입>변수명 = Arrays.stream(배열변수명)
    1. IntStream 변수명 = Arrays.stream(정수형배열 변수명)
2. 컬렉션 스트림 Stream<타입>변수명=list.stream(배열변수명)
3. 난수생성 
    1. **IntStream    ints(int begin, int end)                    // 무한 스트림**
    2. **LongStream   longs(long begin, long end)**
    3. **DoubleStream doubles(double begin, double end)**
    4. **IntStream    ints(long streamSize, int begin, int end)   // 유한 스트림**
    5. **LongStream   longs(long streamSize, long begin, long end)**
    6. **DoubleStream**
    7. **doubles(long streamSize, double begin, double end)**
    
    ```java
    IntStream ints=new Random().ints();
    ints.forEach(System.out::println);
    ```
    
1. 특정 범위의 정수값을 스트림으로 생성
    1. IntStream intStream = IntStream.rangeClosed(1,10);
    2. IntStream.forEach(System.out::println);
    
    rangeClosed() - 끝번호 포함
    
    range() - 끝번호 미포함
    

**`스트림 중간연산`** : **필터링/맵핑/정렬** - 가장 많이 사용

1. 필터링 
    1. distinct() - 중복제거 
    2. filter() - 매개 값으로 조건 predicate을 주고, 조건이 참이되는 요소만 필터링
2. 맵핑 map() 
    1. 원하는 필드만 추출하거나 특정형태로 변환할 때
    2. flatMap() - 중첩구조를 제거하고 단일 컬렉션(Stream<String>)으로 만들어주는 역할
    3. sum() 을 사용하려면 IntStream, LongStream, DoubleStream 와 같은 기본형 (Primitive Type) 특화 스트림을 사용해야함.
        1. mapToInt : 스트림을 `IntStream`으로 변환해주는 메서드
        2. mapToLong, mapToDouble 등이 있음.
    
3. 정렬(sorted())
    1. 아무값도 넣지 않으면 기본정렬(오름차순)
    2. stream.sorted(Comparator.naturalOrder())
    3.  역순정렬 : **sorted(Comparator.reverseOrder())**
    4. 
    
    ![IMG_0593.png](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/53ad5f17-868d-4a92-a090-0be8e539b41c.png)
    
1. 기타
    1. **`skip()`** - 스트림의 일부 요소들을 건너뛰기
        1. skip(5) : 앞의 5개 숫자 건너뛰고 6부터 출력
    2. **`limit()`** - 스트림의 일부를 자릅니다.
    3. **`peek()`** - `forEach()`와 마찬가지로, 요소들을 순회하며 특정 작업을 수행

### **`스트림 최종연산`**

1. forEach()
2. **기본집계 (`sum()` , `count()` , `average()`, `max()` , `min()`)**
    1. **int                         sum()**
    2. long                      count() 
    3. **OptionalInt           max()**
    4. **OptionalInt           min()**
    5. **OptionalDouble   average()**
3. **매칭(`allMatch()`, `anyMatch()`, `noneMatch()` ) -boolean** 
    1. **`allMatch()`** - **모든 요소가 조건을 만족하는지** 여부를 판단
    2. **`noneMatch()`** - **모든 요소가 조건을 만족하지 않는지** 여부를 판단
    3. **`anyMatch()`** - **하나라도 조건을 만족하는 요소가 있는지** 여부를 판단
4. **요소 소모(`reduce()`)**
    1. **스트림의 요소를 줄여나가면서 연산을 수행하고 최종적인 결과를 반환**
    2. 매개변수 타입 : `BinaryOperator<T>`, identity -초기값, accumulator : 누적결과 생성
    3. .reduce((a , b) -> a + b) - 초기값 없는 reduce 
    4.  .reduce(5, (a ,b) -> a + b); 초기값 있는 reduce
5. 요소 수집**(`collect()`)**
    1. 요소를 수집하는 기능 이외에도 **요소 그룹핑 및 분할 등 다른 기능들을 제공**
        1. Map<key ,value> 으로 변환 : collect(Collectors.toMap)
        2. List로 저장 :  **collect(Collectors.toList())**   
        3. 
6. 이외
    1. findAny() -  조건에 일치하는 아무거나 하나를 반환
    2. findFirst() - 조건에 일치하는 첫번째 요소
    3. toArray()
        1. 그냥 .toArray()만 쓰면 object[]
        2. 원하는 타입(String)이 있을때    .toArray(String[]::new)
    4. 참고
- `getAsDouble()`과 `getAsInt()`는 객체로 반환되는 값을 다시 기본형으로 변환하기 위해 사용되는 메서드 - 파이프라인과는 관계없음.
- **isPresent() – Optional객체의 값이 null이면 false, 아니면 true를 반환**
    


## Optional class

1.  Optional<T>
    1. 사용 목적
        1. NullPointerException(NPE), 즉 **null 값으로 인해 에러가 발생하는 현상을 객체 차원에서 효율적으로 방지하고자 도입**
        2. 연산 결과를 Optional에 담아서 반환하면, 따로 조건문을 작성해주지 않아도 NPE가 발생하지 않도록 코드를 작성가능
    2. **모든 타입의 객체를 담을 수 있는 Wrapper클래스**
       
        ```java
        public final class Optional<T> {
        	private final T value; // T타입의 참조변수
        }
        ```
        
    3. Optional 객체를 생성하려면 `of()` 또는 `ofNullable()`을 사용합니다. 참조변수의 값이 null일 가능성이 있다면, `ofNullable()`을 사용합니다.
       
        ```java
        Optional<String> opt1 = Optional.ofNullable(null);
        Optional<String> opt2 = Optional.ofNullable("123");
        opt1.isPresent(); //Optional 객체의 값이 null인지 아닌지를 리턴합니다.
        System.out.println(opt2.isPresent());
        ```
        
    4. Optional<T> 타입의 참조변수를 기본값으로 초기화 : `empty()`
       
        ```java
        Optional<String> opt3 = Optional.<String>empty(
        ```
        
    5. `get()` : Optional 객체에 객체에 저장된 값을 가져오기
       
        ```java
        Optional<String> optString = Optional.of("java");
        System.out.println(optString);
        System.out.println(optString.get());
        ```
        
    6. `orElse()` : 참조변수의 값이 null일 가능성이 있을 때 디폴트 값 지정 가능
       
        ```java
        String nullName = null;
        String name = Optional.ofNullable(nullName).orElse("kimcoding");
        System.out.println(name);
        ```
        
    7.  Optional 객체는 스트림과 유사하게 여러 메서드를 연결해서 작성가능 = 메서드체이닝
        
        ```java
        import java.util.Arrays;
        import java.util.List;
        import java.util.Optional;
        
        public class OptionalExample {
            public static void main(String[] args) {
                List<String> languages = Arrays.asList(
                        "Ruby", "Python", "Java", "Go", "Kotlin");
                Optional<List<String>> listOptional = Optional.of(languages);
        
                int size = listOptional
                        .map(List::size)
                        .orElse(0);
                System.out.println(size);
            }
        }
        ```
        
    
    e.  optional 객체에서 제공하는 전체 메서드 : [Optional (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
    
    [Optional 심화학습](https://www.notion.so/Optional-5f789b404ae2412f875387adfeafb651?pvs=21)
    
- 스트림 활용예제 - lazy evaluation임을 잘 알 수 있음,  Lazy and Short-circuit
  
    ```java
    import java.lang.invoke.CallSite;
    import java.util.*;
    
    public class ListTest {
        public static void main(String[] args) {
            List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
        }
    
        private static List<Integer> filter(List<Integer> list) {
            List<Integer> result = new ArrayList<>();
            List<Integer>filteredList = filter(list);
            System.out.println("-".repeat(40));
            System.out.println(filteredList);
    
            int count = 0;
            for (Integer integer : list) {
                System.out.println("숫자를 필터링합니다. integer: " + integer);
                if (integer % 3 == 0) {
                    if (count < 3) {
                        result.add(integer * 10);
                        count++;
                    }
                }
            }
            return result;
        }
    }
    ```
    
    1. 상기 코드를 스트림으로 구현한 코드를 아래에서 볼 수 있음
    
    ```java
    import java.util.*;
    import java.util.stream.Collectors;
    
    public class ListTest {
        public static void main(String[] args) {
            List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
            List<Integer>filteredList = streamFilter(list);
            System.out.println("-".repeat(40));
            System.out.println(filteredList);
        }
        private static List<Integer> streamFilter(List<Integer> list) {
    
            return list.stream()
            .filter(integer->{
                System.out.println("숫자를 필터링합니다. integer: " + integer);
                    return integer %3 ==0;
            }).map(integer -> integer*10)
                    .limit(3)
                    .collect(Collectors.toList());
        }
    }
    ```
    
    1. 스트림으로 구현시 출력 결과
       
        ![Untitled](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/Untitled%2011.png)
    
    

---

**스트림 요약**

**`스트림 생성`**

1. 배열 Stream<타입>변수명 = Arrays.stream(배열변수명)
    1. IntStream 변수명 = Arrays.stream(정수형배열 변수명)
2. 컬렉션 스트림 Stream<타입>변수명=list.stream(배열변수명)
3. 난수생성 
    1. **IntStream    ints(int begin, int end)                    // 무한 스트림**
    2. **LongStream   longs(long begin, long end)**
    3. **DoubleStream doubles(double begin, double end)**
    4. **IntStream    ints(long streamSize, int begin, int end)   // 유한 스트림**
    5. **LongStream   longs(long streamSize, long begin, long end)**
    6. **DoubleStream**
    7. **doubles(long streamSize, double begin, double end)**
    
    ```java
    IntStream ints=new Random().ints();
    ints.forEach(System.out::println);
    ```
    
1. 특정 범위의 정수값을 스트림으로 생성
    1. IntStream intStream = IntStream.rangeClosed(1,10);
    2. IntStream.forEach(System.out::println);
    
    rangeClosed() - 끝번호 포함
    
    range() - 끝번호 미포함
    

**`스트림 중간연산`** : **필터링/맵핑/정렬** - 가장 많이 사용

1. 필터링 
    1. distinct() - 중복제거 
    2. filter() - 매개 값으로 조건 predicate을 주고, 조건이 참이되는 요소만 필터링
2. 맵핑 map() 
    1. 원하는 필드만 추출하거나 특정형태로 변환할 때
    2. flatMap() - 중첩구조를 제거하고 단일 컬렉션(Stream<String>)으로 만들어주는 역할
    3. sum() 을 사용하려면 IntStream, LongStream, DoubleStream 와 같은 기본형 (Primitive Type) 특화 스트림을 사용해야함.
        1. mapToInt : 스트림을 `IntStream`으로 변환해주는 메서드
        2. mapToLong, mapToDouble 등이 있음.
    
3. 정렬(sorted())
    1. 아무값도 넣지 않으면 기본정렬(오름차순)
    2. stream.sorted(Comparator.naturalOrder())
    3.  역순정렬 : **sorted(Comparator.reverseOrder())**
    4. 
    
    ![IMG_0593.png](28%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1+%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20fce1d28bc56e42bd8eb1be8858fd236c/53ad5f17-868d-4a92-a090-0be8e539b41c.png)
    
1. 기타
    1. **`skip()`** - 스트림의 일부 요소들을 건너뛰기
        1. skip(5) : 앞의 5개 숫자 건너뛰고 6부터 출력
    2. **`limit()`** - 스트림의 일부를 자릅니다.
    3. **`peek()`** - `forEach()`와 마찬가지로, 요소들을 순회하며 특정 작업을 수행

### **`스트림 최종연산`**

1. forEach()
2. **기본집계 (`sum()` , `count()` , `average()`, `max()` , `min()`)**
    1. **int                         sum()**
    2. long                      count() 
    3. **OptionalInt           max()**
    4. **OptionalInt           min()**
    5. **OptionalDouble   average()**
3. **매칭(`allMatch()`, `anyMatch()`, `noneMatch()` ) -boolean** 
    1. **`allMatch()`** - **모든 요소가 조건을 만족하는지** 여부를 판단
    2. **`noneMatch()`** - **모든 요소가 조건을 만족하지 않는지** 여부를 판단
    3. **`anyMatch()`** - **하나라도 조건을 만족하는 요소가 있는지** 여부를 판단
4. **요소 소모(`reduce()`)**
    1. **스트림의 요소를 줄여나가면서 연산을 수행하고 최종적인 결과를 반환**
    2. 매개변수 타입 : `BinaryOperator<T>`, identity -초기값, accumulator : 누적결과 생성
    3. .reduce((a , b) -> a + b) - 초기값 없는 reduce 
    4.  .reduce(5, (a ,b) -> a + b); 초기값 있는 reduce
5. 요소 수집**(`collect()`)**
    1. 요소를 수집하는 기능 이외에도 **요소 그룹핑 및 분할 등 다른 기능들을 제공**
        1. Map<key ,value> 으로 변환 : collect(Collectors.toMap)
        2. List로 저장 :  **collect(Collectors.toList())**   
        3. 
6. 이외
    1. findAny() -  조건에 일치하는 아무거나 하나를 반환
    2. findFirst() - 조건에 일치하는 첫번째 요소
    3. toArray()
        1. 그냥 .toArray()만 쓰면 object[]
        2. 원하는 타입(String)이 있을때    .toArray(String[]::new)
    4. 참고
- `getAsDouble()`과 `getAsInt()`는 객체로 반환되는 값을 다시 기본형으로 변환하기 위해 사용되는 메서드 - 파이프라인과는 관계없음.
- **isPresent() – Optional객체의 값이 null이면 false, 아니면 true를 반환**