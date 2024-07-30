---
layout : single
title : "[python] basic collection"
categories: python
tag : [python, 실습, 개념]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## 튜플(Tuple)

파이썬의 기본 자료형 중 하나로, 여러 개의 값을 하나의 변수에 저장할 수 있는 데이터 구조

소괄호 `()`를 사용하여 정의하며, 각 요소는 쉼표 `,`로 구분

### 리스트와 튜플의 차이

1. 변경 가능성 (Mutability)

- **리스트**는 **변경 가능**(mutable)  즉, 리스트의 요소를 추가, 수정, 삭제할 수 있다.
- **튜플**은 **변경 불가능**(immutable) 즉, 튜플이 생성된 후에는 요소를 변경할 수 없다.

2. 사용 용도

- **리스트**는 요소의 추가, 삭제, 변경이 필요한 경우 사용된다.
- **튜플**은 요소가 고정된 경우(변경되지 않아야 할 데이터)를 저장할 때 사용된다. 예를 들어, 함수의 반환값을 여러 개 묶어서 반환하거나, 고정된 설정 값을 저장할 때 튜플이 유용하다.

3. 성능

```python
import sys

list_example = [1, 2, 3, 4, 5]
tuple_example = (1, 2, 3, 4, 5)

print(sys.getsizeof(list_example))  # 출력: 리스트의 메모리 사용량
print(sys.getsizeof(tuple_example))  # 출력: 튜플의 메모리 사용량
```

- **튜플**은 **리스트**보다 메모리 사용량이 적고, 생성 시간이 더 빠릅니다. 이는 튜플이 불변성이 있기 때문에 가능한 최적화 덕분입니다.
- 따라서, 변경이 필요 없는 데이터를 저장할 때는 튜플을 사용하는 것이 성능 면에서 유리하다.

1. 메서드
    - **리스트**는 요소를 변경할 수 있기 때문에 많은 메서드를 제공합니다. 예를 들어, `append()`, `extend()`, `insert()`, `remove()`, `pop()`, `clear()`, `sort()`, `reverse()` 등의 메서드가 있다.
    - **튜플**은 불변이기 때문에 제공하는 메서드가 적습니다. 주요 메서드는 `count()`와 `index()`입니다.
2. 자료형의 불변성
    - **튜플**은 불변이므로, 딕셔너리의 키로 사용할 수 있다. 리스트는 변경 가능하기 때문에 딕셔너리의 키로 사용할 수 없다.
3. 장단점
    - 리스트의 장단점
    - 튜플의 장단점
    
4. 리스트와 튜플을 선택하는 기준
    - **변경 가능 여부**: 요소를 변경해야 하는 경우 리스트를 사용하고, 변경할 필요가 없는 경우 튜플을 사용
    - **성능 요구 사항**: 메모리 사용량을 최소화하고 빠른 생성 속도가 필요한 경우 튜플을 사용
    - **사용 용도**: 함수의 반환값을 여러 개 묶어서 반환하거나, 설정 값을 고정하여 저장하는 경우 튜플을 사용

### 튜플

1. 튜플 생성
    - 기본 생성 방법
        
        ```python
        # 빈 튜플 생성
        empty_tuple = ()
        
        # 여러 요소를 포함하는 튜플 생성
        fruit_tuple = ("사과", "바나나", "체리")
        
        # 튜플은 다양한 자료형을 포함할 수 있습니다.
        mixed_tuple = (1, "안녕", 3.14, True)
        
        ```
        
    - 요소가 하나인 튜플 생성
        
        ```python
        single_element_tuple = (1,)
        not_a_tuple = (1)
        
        print(type(single_element_tuple))  # 출력: <class 'tuple'>
        print(type(not_a_tuple))  # 출력: <class 'int'>
        
        ```
        
    - 튜플 생성 시 괄호 생략
        
        ```python
        # 괄호를 생략하여 튜플 생성
        fruit_tuple = "사과", "바나나", "체리"
        print(fruit_tuple)  # 출력: ('사과', '바나나', '체리')
        print(type(fruit_tuple))  # 출력: <class 'tuple'>
        ```
        
2. 튜플 접근 및 슬라이싱
    - **튜플**의 요소는 인덱스를 사용하여 접근가능, 인덱스는 0부터 시작한다.
    - **슬라이싱**을 사용하여 튜플의 일부를 추출가능
    
    ```python
    fruit_tuple = ("사과", "바나나", "체리")
    
    # 인덱스를 사용한 접근
    print(fruit_tuple[0])  # 출력: 사과
    print(fruit_tuple[1])  # 출력: 바나나
    print(fruit_tuple[2])  # 출력: 체리
    
    # 음수 인덱스를 사용한 접근
    print(fruit_tuple[-1])  # 출력: 체리
    print(fruit_tuple[-2])  # 출력: 바나나
    print(fruit_tuple[-3])  # 출력: 사과
    
    # 슬라이싱
    print(fruit_tuple[0:2])  # 출력: ('사과', '바나나')
    print(fruit_tuple[1:])   # 출력: ('바나나', '체리')
    print(fruit_tuple[:2])   # 출력: ('사과', '바나나')
    print(fruit_tuple[:])    # 출력: ('사과', '바나나', '체리')
    ```
    
3. 튜플 연산
    - **튜플**은 `+` 연산자를 사용하여 결합(concatenate)할 수 있으며, `` 연산자를 사용하여 반복(repeat)가능
    
    ```python
    fruit_tuple = ("사과", "바나나")
    veg_tuple = ("당근", "브로콜리")
    
    # 튜플 결합
    combined_tuple = fruit_tuple + veg_tuple
    print(combined_tuple)  # 출력: ('사과', '바나나', '당근', '브로콜리')
    
    # 튜플 반복
    repeated_tuple = fruit_tuple * 2
    print(repeated_tuple)  # 출력: ('사과', '바나나', '사과', '바나나')
    ```
    
4. 튜플의 불변성
    - 튜플은 한 번 생성되면 그 요소를 변경할 수 없습니다. 하지만, 튜플 내의 가변 객체(예: 리스트)는 변경할 수 있다.
        
        ```python
        # 요소 변경 시도 (오류 발생)
        # fruit_tuple[0] = "오렌지"  # TypeError: 'tuple' object does not support item assignment
        
        # 가변 객체를 포함한 튜플
        nested_list = ([1, 2, 3], [4, 5, 6])
        nested_list[0][1] = 9
        print(nested_list)  # 출력: ([1, 9, 3], [4, 5, 6])
        ```
        
5. 튜플의 메서드
    - **튜플**은 리스트와 달리 몇 가지 메서드만 제공합니다. 대표적으로 `count()`와 `index()` 메서드가 있다.
    - count() : 튜플에서 특정 요소의 개수를 센다.
        
        ```python
        fruit_tuple = ("사과", "바나나", "체리", "사과")
        print(fruit_tuple.count("사과"))  # 출력: 2
        ```
        
    - index() : 튜플에서 특정 요소의 첫 번째 인덱스를 반환
        
        ```python
        fruit_tuple = ("사과", "바나나", "체리", "사과")
        print(fruit_tuple.index("바나나"))  # 출력: 1
        ```
        
6. 튜플 언패킹
    
    언패킹을 사용하면 튜플의 각 요소를 별도의 변수에 할당할 수 있다.
    
    ```python
    fruit_tuple = ("사과", "바나나", "체리")
    
    # 튜플 언패킹
    apple, banana, cherry = fruit_tuple
    print(apple)  # 출력: 사과
    print(banana)  # 출력: 바나나
    print(cherry)  # 출력: 체리
    
    # 일부 요소 무시하기 (언더스코어 사용)
    _, banana, _ = fruit_tuple
    print(banana)  # 출력: 바나나
    ```
    
7. 중첩된 튜플
    
    **중첩된 튜플**은 튜플 내에 또 다른 튜플을 포함하는 튜플
    
    ```python
    # 중첩된 튜플 접근
    print(nested_tuple[0])  # 출력: ('a', 'b', 'c')
    print(nested_tuple[1][2])  # 출력: 3
    ```
    
8. 튜플과 리스트 변환
    
    **튜플**을 **리스트**로, **리스트**를 **튜플**로 변환가능
    
    ```python
    fruit_tuple = ("사과", "바나나", "체리")
    fruit_list = list(fruit_tuple)
    print(fruit_list)  # 출력: ['사과', '바나나', '체리']
    
    # 리스트를 튜플로 변환
    fruit_list = ["사과", "바나나", "체리"]
    fruit_tuple = tuple(fruit_list)
    print(fruit_tuple)  # 출력: ('사과', '바나나', '체리')
    ```
    

### 예제 및 실습 문제

1. 문제 1: 두 개의 튜플을 합치는 함수 작성
    
    ```python
    def merge_tuples(tuple1, tuple2):
        return tuple1 + tuple2
    
    tuple1 = (1, 2, 3)
    tuple2 = (4, 5, 6)
    merged_tuple = merge_tuples(tuple1, tuple2)
    print(merged_tuple)  # 출력: (1, 2, 3, 4, 5, 6)
    ```
    
2. 문제 2: 튜플에서 최대값과 최소값을 찾는 함수 작성
    
    ```python
    def find_max_min(t):
        return max(t), min(t)
    
    numbers = (5, 2, 9, 1, 5, 6)
    max_value, min_value = find_max_min(numbers)
    print(f"최대값: {max_value}, 최소값: {min_value}")  # 출력: 최대값: 9, 최소값: 1
    
    ```
    
3. 문제 3: 주어진 리스트를 튜플로 변환하고, 해당 튜플의 요소를 출력하는 함수 작성
    
    ```python
    def list_to_tuple(lst):
        return tuple(lst)
    
    lst = [10, 20, 30, 40, 50]
    t = list_to_tuple(lst)
    print(t)  # 출력: (10, 20, 30, 40, 50)
    ```
    

## 리스트(list)

- 파이썬에서 가장 많이 사용되는 자료형 중 하나로, 여러 개의 값을 하나의 변수에 저장할 수 있는 데이터 구조이다.
- 대괄호([])를 사용하여 정의하며, 각 요소는 쉼표로 구분한다.
- 리스트의 요소는 다양한 자료형(정수, 문자열, 다른 리스트 등)을 가질 수 있으며, 하나의 리스트에 다양한 자료형을 혼합가능.

### **리스트 생성**

```python
# 빈 리스트 생성
empty_list = []

# 정수형 리스트 생성
numbers = [1, 2, 3, 4, 5]

# 문자열 리스트 생성
fruits = ["사과", "바나나", "체리"]

# 다양한 자료형을 포함하는 리스트 생성
mixed_list = [1, "안녕", True, [1, 2, 3]]
```

### **리스트 접근 및 슬라이싱**

- 리스트의 요소는 인덱스를 사용하여 접근가능 하고 인덱스는 0부터 시작한다.
- 음수 인덱스를 사용하여 리스트의 끝에서부터 요소에 접근할 수도 있다.

```python
fruits = ["사과", "바나나", "체리"]

# 인덱스를 사용한 접근
print(fruits[0])  # 출력: 사과
print(fruits[1])  # 출력: 바나나
print(fruits[2])  # 출력: 체리

# 음수 인덱스를 사용한 접근
print(fruits[-1])  # 출력: 체리
print(fruits[-2])  # 출력: 바나나
print(fruits[-3])  # 출력: 사과

# 리스트 슬라이싱
print(fruits[0:2])  # 출력: ['사과', '바나나']
print(fruits[1:])   # 출력: ['바나나', '체리']
print(fruits[:2])   # 출력: ['사과', '바나나']
print(fruits[:])    # 출력: ['사과', '바나나', '체리']
```

### **리스트 수정 및 삭제**

- 리스트의 요소는 인덱스를 사용하여 수정할 수 있다.
- append(), extend(), insert(), remove(), pop(), clear() 등의 메서드를 사용하여 리스트를 수정 가능

```python
fruits = ["사과", "바나나", "체리"]

# 요소 수정
fruits[1] = "블루베리"
print(fruits)  # 출력: ['사과', '블루베리', '체리']

# 요소 추가
fruits.append("오렌지")
print(fruits)  # 출력: ['사과', '블루베리', '체리', '오렌지']

# 요소 삽입
fruits.insert(1, "키위")
print(fruits)  # 출력: ['사과', '키위', '블루베리', '체리', '오렌지']

# 요소 삭제
fruits.remove("블루베리")
print(fruits)  # 출력: ['사과', '키위', '체리', '오렌지']

# 마지막 요소 삭제
fruits.pop()
print(fruits)  # 출력: ['사과', '키위', '체리']

# 특정 인덱스의 요소 삭제
fruits.pop(1)
print(fruits)  # 출력: ['사과', '체리']

# 리스트 초기화
fruits.clear()
print(fruits)  # 출력: []
```

### **리스트 메서드**

1. **요소 추가**
    
    ```python
    numbers = [1, 2, 3]
    numbers.append(4)
    print(numbers)  # 출력: [1, 2, 3, 4]
    
    numbers.extend([5, 6])
    print(numbers)  # 출력: [1, 2, 3, 4, 5, 6]
    
    numbers.insert(3, 2.5)
    print(numbers)  # 출력: [1, 2, 3, 2.5, 4, 5, 6]
    ```
    
    - append(x): 리스트의 끝에 요소 x를 추가한다.
    - extend(iterable): 반복 가능한 객체의 모든 요소를 리스트의 끝에 추가 한다.
    - insert(i, x): 인덱스 i에 요소 x를 삽입한다.

1. **요소 삭제**
    
    ```python
    numbers = [1, 2, 3, 2, 4, 5]
    numbers.remove(2)
    print(numbers)  # 출력: [1, 3, 2, 4, 5]
    
    numbers.pop(2)
    print(numbers)  # 출력: [1, 3, 4, 5]
    
    numbers.clear()
    print(numbers)  # 출력: []
    ```
    
    - remove(x): 리스트에서 첫 번째로 나오는 요소 x를 삭제한다
    - pop([i]): 인덱스 i에 있는 요소를 삭제하고 반환합니다. 인덱스를 지정하지 않으면 마지막 요소를 삭제하고 반환한다.
    - clear(): 리스트의 모든 요소를 삭제한다

1. **요소 찾기 및 개수 세기**
    - index(x[, start[, end]]): 리스트에서 첫 번째로 나오는 요소 x의 인덱스를 반환한다.
    - count(x): 리스트에서 요소 x의 개수를 센다.
    
    ```python
    numbers = [1, 2, 3, 2, 4, 2, 5]
    print(numbers.index(2))  # 출력: 1
    print(numbers.index(2, 2))  # 출력: 3
    print(numbers.count(2))  # 출력: 3
    ```
    
2. **리스트 정렬**
    - sort(key=None, reverse=False): 리스트를 정렬
    - key는 정렬 기준이 되는 함수, reverse는 정렬 순서를 지정
    - reverse(): 리스트의 요소 순서를 반대로 뒤집는다
    
    ```python
    numbers = [5, 2, 9, 1, 5, 6]
    numbers.sort()
    print(numbers)  # 출력: [1, 2, 5, 5, 6, 9]
    
    numbers.sort(reverse=True)
    print(numbers)  # 출력: [9, 6, 5, 5, 2, 1]
    
    numbers.reverse()
    print(numbers)  # 출력: [1, 2, 5, 5, 6, 9]
    ```
    

1. **리스트 복사**
    
    copy(): 리스트의 얕은 복사본을 만든다.
    
    ```python
    numbers = [1, 2, 3, 4, 5]
    numbers_copy = numbers.copy()
    print(numbers_copy)  # 출력: [1, 2, 3, 4, 5]
    ```
    

### **리스트 컴프리헨션 (List Comprehensions)**

- **리스트 컴프리헨션**은 기존 리스트를 기반으로 새로운 리스트를 간단하고 직관적으로 생성하는 방법.
- 반복문과 조건문을 한 줄로 결합하여 복잡한 리스트를 생성할 수 있으며, 코드의 가독성과 효율성을 높인다.
- 리스트 컴프리헨션은 기존의 for 루프를 간결하게 표현할 수 있어 더 읽기 쉽고 유지 관리하기 쉬운 코드를 작성가능

1. 기본 형태
    
    ```python
    [expression for item in iterable if condition]
    ```
    
    **expression**: 각 요소에 대해 실행할 표현식
    
    **item**: 반복 가능한 객체(예: 리스트, 튜플, 문자열 등)에서 가져온 요소
    
    **iterable**: 반복 가능한 객체
    
    **condition**: 선택 사항으로, 이 조건이 참인 경우에만 요소가 리스트에 포함된다.
    

### **예제**

1. 예제 1
    
    1부터 10까지의 숫자 중에서 짝수만 포함된 리스트를 생성합니다.
    
    ```python
    even_numbers = [x for x in range(1, 11) if x % 2 == 0]
    print(even_numbers)  # 출력: [2, 4, 6, 8, 10]
    ```
    
    - 여기서 x는 1부터 10까지의 숫자이며, x % 2 == 0 조건은 x가 짝수인 경우에만 참이 된다.
2. 예제 2
    
    1부터 10까지의 숫자의 제곱을 포함한 리스트를 생성합니다.
    
    ```python
    squares = [x**2 for x in range(1, 11)]
    print(squares)  # 출력: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100
    ```
    
    - 여기서 x**2는 x의 제곱을 계산한다.
    - 결과적으로, [1, 4, 9, 16, 25, 36, 49, 64, 81, 100] 형태의 리스트가 생성됨

1. **예제 3** 
    
    **두 리스트의 Cartesian product 생성**
    
    ```python
    colors = ["red", "green", "blue"]
    objects = ["ball", "cube"]
    cartesian_product = [(color, obj) for color in colors for obj in objects]
    print(cartesian_product)
    # 출력: [('red', 'ball'), ('red', 'cube'), ('green', 'ball'), ('green', 'cube'), ('blue', 'ball'), ('blue', 'cube'
    ```
    
    여기서 for color in colors와 for obj in objects는 중첩된 반복문을 나타낸다.
    
    각 색상과 객체의 조합을 튜플로 생성하여 리스트에 추가한다.
    

1.  **문자열의 각 문자를 대문자로 변환**
    
    주어진 문자열의 각 문자를 대문자로 변환하여 리스트를 생성
    
    ```python
    text = "hello"
    uppercase_letters = [char.upper() for char in text]
    print(uppercase_letters)  # 출력: ['H', 'E', 'L', 'L', 'O']
    ```
    

 ****

1. **리스트의 요소 중 조건을 만족하는 요소만 선택**
    
    1부터 20까지의 숫자 중에서 3의 배수만 포함된 리스트를 생성
    
    ```python
    multiples_of_three = [x for x in range(1, 21) if x % 3 == 0]
    print(multiples_of_three)  # 출력: [3, 6, 9, 12, 15, 18]
    ```
    
    여기서 x % 3 == 0 조건은 x가 3의 배수인 경우에만 참
    
2. **리스트의 요소를 조건에 따라 변환**
    
    1부터 10까지의 숫자 중에서 짝수는 제곱하고, 홀수는 그대로 포함된 리스트를 생성한다.
    
    ```python
    numbers = [x**2 if x % 2 == 0 else x for x in range(1, 11)]
    print(numbers)  # 출력: [1, 4, 3, 16, 5, 36, 7, 64, 9, 100]
    ```
    
    여기서 x**2 if x % 2 == 0 else x 표현식은 x가 짝수인 경우 제곱을 계산하고, 홀수인 경우 그대로 x를 반환한다.
    

1. **중첩 리스트 컴프리헨션**
    
    2차원 리스트의 요소를 평탄화(flatten)하여 1차원 리스트로 변환한다.
    
    ```python
    matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    flattened = [num for row in matrix for num in row]
    print(flattened)  # 출력: [1, 2, 3, 4, 5, 6, 7, 8, 9]
    ```
    
    여기서 for row in matrix는 2차원 리스트의 각 행을 반복하고, for num in row는 각 행의 요소를 반복한다.
    

### **리스트 컴프리헨션의 장점**

- **간결성**: 리스트 컴프리헨션을 사용하면 더 간결하고 읽기 쉬운 코드를 작성할 수 있다.
- **가독성**: 한 줄로 작성된 컴프리헨션은 반복문을 사용한 코드보다 가독성이 높다.
- **성능**: 리스트 컴프리헨션은 일반적인 for 루프보다 더 빠르게 실행될 수 있다. 이는 컴프리헨션이 내부적으로 최적화되어 있기 때문이다.
- **리스트 컴프리헨션의 주의사항**
- **과도한 사용 지양**: 너무 복잡한 리스트 컴프리헨션은 오히려 가독성을 떨어뜨릴 수 있습니다. 간단한 경우에만 사용하는 것이 좋다.
- **디버깅 어려움**: 한 줄로 작성된 리스트 컴프리헨션은 디버깅이 어려울 수 있으므로 복잡한 로직은 여러 줄로 나누어 작성하는 것이 좋다.

### **5. 예제 및 실습 문제**

1. **문제 1: 리스트에서 중복된 요소를 제거하는 함수 작성**
    
    ```python
    def remove_duplicates(lst):
        return list(set(lst))
    
    numbers = [1, 2, 3, 2, 4, 5, 3, 6]
    unique_numbers = remove_duplicates(numbers)
    print(unique_numbers)  # 출력: 중복이 제거된 리스트
    ```
    

1. **문제 2: 리스트에서 가장 큰 값과 가장 작은 값을 찾는 함수 작성**
    
    ```python
    def find_max_min(lst):
        return max(lst), min(lst)
    
    numbers = [1, 2, 3, 4, 5]
    max_value, min_value = find_max_min(numbers)
    print(f"최대값: {max_value}, 최소값: {min_value}")  
    # 출력: 최대값과 최소값
    ```
    

1. **문제 3: 리스트의 모든 요소를 제곱한 값을 갖는 새 리스트를 생성하는 함수 작성**
    
    ```python
    def square_elements(lst):
        return [x**2 for x in lst]
    
    numbers = [1, 2, 3, 4, 5]
    squared_numbers = square_elements(numbers)
    print(squared_numbers)  
    
    # 출력: [1, 4, 9, 16, 25]
    ```
    

## Dictionary

java의 Map과 비슷

- 키-값 쌍으로 데이터를 저장하는 자료형
- **딕셔너리**는 중괄호 `{}`를 사용하여 정의하며, 각 키와 값은 콜론 `:`으로 구분되고, 쌍은 쉼표 `,`로 구분된다.
- **딕셔너리**의 키는 불변(immutable) 자료형이어야 하며, 값은 어떤 자료형도 가능
- **딕셔너리**는 해시 테이블을 사용하여 구현되므로, 키를 통해 값을 빠르게 검색할 수 있다.

### 딕셔너리 생성

```python

# 빈 딕셔너리 생성
empty_dict = {}

# 키-값 쌍을 포함한 딕셔너리 생성
person = {
    "name": "홍길동",
    "age": 27,
    "city": "서울"
}

# dict() 함수를 사용하여 딕셔너리 생성
person = dict(name="홍길동", age=27, city="서울")
```

### 딕셔너리 접근 및 수정

 키를 사용하여 접근하고 수정가능

```python

person = {"name": "홍길동", "age": 27, "city": "서울"}

# 키를 사용한 값 접근
print(person["name"])  # 출력: 홍길동
print(person["age"])  # 출력: 27

# 값 수정
person["age"] = 28
print(person)  # 출력: {'name': '홍길동', 'age': 28, 'city': '서울'}

# 새 키-값 쌍 추가
person["job"] = "개발자"
print(person)  # 출력: {'name': '홍길동', 'age': 28, 'city': '서울', 'job': '개발자'}
```

### 딕셔너리 메서드

1. 요소 추가 및 업데이트
    - `update()` 메서드를 사용하여 다른 딕셔너리의 키-값 쌍을 추가하거나 업데이트할 수 있다.
    
    ```python
    person = {"name": "홍길동", "age": 27}
    person.update({"city": "서울", "job": "개발자"})
    print(person)  # 출력: {'name': '홍길동', 'age': 27, 'city': '서울', 'job': '개발자'}
    ```
    
2. 요소 삭제
    - `del` 키워드를 사용하거나 `pop()` 메서드를 사용하여 요소를 삭제할 수 있다.
    - `popitem()` 메서드는 마지막 키-값 쌍을 삭제하고 반환한다..
    - `clear()` 메서드는 딕셔너리의 모든 요소를 삭제한다.
3. 요소 검색
    - `keys()`, `values()`, `items()` 메서드를 사용하여 딕셔너리의 키, 값, 키-값 쌍을 반환할 수 있다.
4. 요소 존재 여부 확인
    
    ```python
    
    person = {"name": "홍길동", "age": 27, "city": "서울"}
    
    # 키 존재 여부 확인
    print("name" in person)  # 출력: True
    print("job" in person)   # 출력: False
    ```
    

### 딕셔너리 컴프리헨션(Dictionary Comprehensions)

- 기존의 리스트 컴프리헨션과 유사한 방식으로 딕셔너리를 간결하고 직관적으로 생성하는 방법입니다.
- 반복문과 조건문을 결합하여 새로운 딕셔너리를 생성할 수 있다
- 이 방법을 사용하면 한 줄의 코드로 복잡한 딕셔너리를 생성할 수 있어 코드의 가독성과 효율성을 높인다.
1. 기본 형태
    
    ```python
    {키_표현식: 값_표현식 for 아이템 in 반복가능객체 if 조건}
    ```
    
    - **키_표현식**: 딕셔너리의 키를 생성하는 표현식
    - **값_표현식**: 딕셔너리의 값을 생성하는 표현식
    - **아이템**: 반복 가능한 객체(예: 리스트, 튜플, 문자열 등)에서 가져온 요소
    - **조건**: 선택 사항으로, 이 조건이 참인 경우에만 아이템이 딕셔너리에 포함된다.
2. 예제
    
    ```python
    text = "hello world"
    frequency = {char: text.count(char) for char in set(text)}
    print(frequency)  
    # 출력: {'d': 1, 'h': 1, 'e': 1, 'o': 2, 'l': 3, 'r': 1, ' ': 1, 'w': 1}
    ```
    
    - `set(text)`는 문자열 `text`에서 중복된 문자를 제거하여 유일한 문자만을 포함하는 집합을 만든다.
    - `text.count(char)`는 문자열 `text`에서 문자 `char`의 빈도수를 계산
    - 결과적으로, 각 문자의 빈도수를 나타내는 딕셔너리가 생성된다.