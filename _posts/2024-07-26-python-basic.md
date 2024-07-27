---
layout : single
title : "[python] basic"
categories: python
tag : [python, 실습, 개념]
author_profile: true
---

# python basic

파이썬이랑 자바스크립트가 동적 언어라고 하는데
동적언어라고 불리는 이유는
1. 타입이 고정되지 않는다
    변수 선언시 타입 명시하지 않으며 할당되는 값에 따라 타입이 결정 된다.
2. 런타임 타입 검사
    변수의 타입은 코드가 실행때 결정되고 검사한다.
    컴파일 시점이 아닌 런타임시점에 에러 확인 가능
3. 인터프리터 실행
    파이썬은 인터프리터 언어로 코드 한 줄 한 줄 실행한다.
이외에도 여러 이유로 동적언어라고 불린다!

### 변수 선언 및 할당 그리고 출력

```python
# 변수 선언 및 값 할당
age = 25  # 정수형 변수
height = 1.75  # 실수형 변수
name = "Alice"  # 문자열 변수
is_student = True  # 불리언 변수

# 변수 출력
print(age)  # 25
print(height)  # 1.75
print(name)  # Alice
print(is_student)  # True

```

### 연산자

1. 산술연산자
    
    
    | 연산자 | 설명 | 예시 | 결과 |
    | --- | --- | --- | --- |
    | + | 덧셈 | 5 + 3 | 8 |
    | - | 뺄셈 | 5 - 3 | 2 |
    | * | 곱셈 | 5 * 3 | 15 |
    | / | 나눗셈 | 5 / 3 | 1.67 |
    | // | 몫 | 5 // 3 | 1 |
    | % | 나머지 | 5 % 3 | 2 |
    | ** | 제곱 | 5 ** 3 | 125 |
    
    ```python
    a = 5
    b = 3
    
    print(a + b)  # 8
    print(a - b)  # 2
    print(a * b)  # 15
    print(a / b)  # 1.6666666666666667
    print(a // b)  # 1
    print(a % b)  # 2
    print(a ** b)  # 125
    
    ```
    
    - 자바에서는 Math함수 사용해하는데 기호로 있어서  좋은듯..!
2. 비교 연산자
    
    
    | 연산자 | 설명 | 예시 | 결과 |
    | --- | --- | --- | --- |
    | == | 같다 | 5 == 3 | False |
    | != | 같지 않다 | 5 != 3 | True |
    | > | 크다 | 5 > 3 | True |
    | < | 작다 | 5 < 3 | False |
    | >= | 크거나 같다 | 5 >= 3 | True |
    | <= | 작거나 같다 | 5 <= 3 | False |
    - 비교연산자는 자바와 동일
3. 논리 연산자
    
    
    | 연산자 | 설명 | 예시 | 결과 |
    | --- | --- | --- | --- |
    | and | 논리적 AND | True and False | False |
    | or | 논리적 OR | True or False | True |
    | not | 논리적 NOT | not True | False |
    - 기호가 아니고 영어를 사용한다. (자바의 && / || / ! 와 동일)
4. 할당연산자
    
    
    | 연산자 | 설명 | 예시 | 결과 |
    | --- | --- | --- | --- |
    | = | 값 할당 | a = 5 | a가 5 |
    | += | 덧셈 후 할당 | a += 3 | a가 8 |
    | -= | 뺄셈 후 할당 | a -= 3 | a가 2 |
    | *= | 곱셈 후 할당 | a *= 3 | a가 15 |
    | /= | 나눗셈 후 할당 | a /= 3 | a가 1.67 |
    | //= | 몫 후 할당 | a //= 3 | a가 1 |
    | %= | 나머지 후 할당 | a %= 3 | a가 2 |
    | **= | 제곱 후 할당 | a **= 3 | a가 125 |
    
    ```python
    a //= 3  # a = a // 3
    print(a)  # 1.0
    
    a %= 3  # a = a % 3
    print(a)  # 1.0
    
    a **= 3  # a = a ** 3
    print(a)  # 1.0
    
    ```
    

### if 문

조건문 뒤에 : 을 작성해 주어야 한다.

```python

score = 85
if score >= 90:
    print("A학점")
elif score >= 80:
    print("B학점")  # 출력: B학점
elif score >= 70:
    print("C학점")
else:
    print("F학점")
    
# 중첩 조건문

age = 20
is_student = True
if age < 18:
    if is_student:
        print("학생 미성년자입니다.")
    else:
        print("미성년자입니다.")
else:
    if is_student:
        print("학생 성인입니다.")  # 출력: 학생 성인입니다.
    else:
        print("성인입니다.")

```

- 자바에서 elseif 는 python에서는 elif 이고 나머지는 동일

### for문

```python
for i in range(10):
    print(i)
# 0 에서 9 까지 출력

for i in range(5, 10):
    print(i)
    
# 5 에서 9 까지 출력
```

사용 시 주의 사항 :  만약에 아래와 같이 쓰면 count에서 에러 발생한다.

<img src="https://github.com/user-attachments/assets/99437ec0-379c-42a4-8b78-cfc93b3e9bb0" width =500/>

**풀어보기**

1. 1부터 100 중에 홀수만 출력하기
2. 구구단 출력하기 (2 ~ 9단까지)

```python
for i in range(1,100):
    if i % 2 != 0:
        print(i)

for i in range(2,10):
    for j in range(2,10):
        print (i*j)
```

## 함수

함수는 `def` 키워드를 사용하여 정의하며, 함수 이름, 매개변수 목록, 그리고 함수 본문으로 구성

### 기본 구조

```python

def 함수이름(매개변수1, 매개변수2, ...):
    """문서화 문자열 (옵션)"""
    함수 본문
    return 반환값 (옵션)
```

### 가변 매개변수

- **가변 인자**: 함수에 전달되는 인자의 개수가 가변적일 때 사용합니다.
- `args`: 위치 인자를 튜플로 받습니다.
- `*kwargs`: 키워드 인자를 딕셔너리로 받습니다.

```python

def print_numbers(*args):
    for number in args:
        print(number)

print_numbers(1, 2, 3, 4)  # 출력: 1, 2, 3, 4

def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="홍길동", age=27)  # 출력: name: 홍길동, age: 27

```

### 입출력 (Input/Output)

- 표준입력
    - `input()` 함수를 사용하여 사용자로부터 입력을 받을 수 있고
    - `input()` 함수는 항상 문자열을 반환하므로 필요에 따라 형 변환해야 한다.
    
    ```python
    name = input("이름을 입력하세요: ")
    print(f"안녕하세요, {name}님!")  # 출력: 안녕하세요, 입력한 이름님!
    ```
    
- 표준 출력
    - `print()` 함수를 사용하여 값을 출력 가능
- 파일 입출력
    - 파일을 읽거나 쓰기 위해 `open()` 함수를 사용
    - 파일 모드: 읽기(`'r'`), 쓰기(`'w'`), 추가(`'a'`), 바이너리 모드(`'b'`)
    
    ```python
    
    # 파일 쓰기
    with open("example.txt", "w") as file:
        file.write("Hello, Python!\n")
        file.write("파일 입출력 예제입니다.")
    
    # 파일 읽기
    with open("example.txt", "r") as file:
        content = file.read()
        print(content)  # 출력: 파일의 내용
    
    ```
    

### 재귀함수

- 재귀함수는 잘못 사용하면 성능 문제나 스택 오버플로우와 같은 문제가 발생할 수 있으므로 사용을 자제해야한다.
- JSON 직렬화와 같은 작업에서는 재귀함수가 자주 사용되는데, 이는 JSON 구조가 종종 중첩된 객체와 배열로 구성되기 때문이다.  재귀함수를 사용하면 이러한 중첩 구조를 간편하게 순회하고 처리할 수 있다.
- 기본조건은 탈출구가 있어야한다.

**📌 재귀함수로 풀어보기**

1. 팩토리얼
    
    ```python
    def is_factorial(num):
        if num == 1:
            return 1
        return num * is_factorial(num - 1)
    
    print(is_factorial(100))
    ```
    
    위는 내가 푼 코드인데 아래와 같이 개선할 수 있다.
    
2. 피보나치
    
    ```python
    def is_fibonacchi(num):
        if num <= 0:
            return 0
        elif num == 1:
            return 1
        else:
            return is_fibonacchi(num-1)+ is_fibonacchi(num-2)
    ```
    
    위는 내가 푼 코드인데 아래 코드와 같이 개선할 수 있다
    
    ```python
    def is_fibonacchi(num):
        if num <= 1:
            return num
        return is_fibonacchi(num-1)+ is_fibonacchi(num-2)
    ```
    
    이 방법은 효율이 좋지 않아서 작은 수는 괜찮지만 조금만 커져도 엄청 오래 걸리게 된다.
    
    ---
    

📌 **소수 구하기 - 에라토스테네스의 체**

```python
# 1

def is_prime(num):
    count = 0
    for i in range(1, num+1):
        if num % i == 0:
            count += 1
    if count == 2:
        return True
    return False

# 2 - 에라토스테네스의 체

def is_prime(num):
    for i in range(2, int(num ** 0.5) + 1):
        if num % i == 0:
            return False
    return True
```

- 1번은 내가 구현한 소수 구하는 방법
- 2번은 에라토스 테네스의 체 : 소수 구하는 알고리즘
- 만약에 숫자가 엄청 크다면 결과가 기다려도 오래 걸리는데 에라토스테네스의 체로 풀면 금방 찾을 수 있다.

📌  최대공약수 - **유클리드 호제법**

```python
def gcd(a, b):
	if b == 0:
		return a
	return gcd(b, a % b) 
```

<br/>
<br/>

## Comment
파이썬은 동적언어라고 하는데 참 .. 자바 먼저 배우고 파이썬 배우니까<br/>
낯설기도 하고 신기하기도 하다, 정말 코딩테스트 풀기 좋겠다는 생각밖에 안듬..
자바로 몇줄이나 적어야 되는 걸 파이썬은 다된다..
코드에 대해 아무것도 모르는 사람이 처음 접하기에는 정말 좋은 언어인 듯
