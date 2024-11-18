---
layout : single
title : "[SQLD] SQLD 55회 후기 및 정리"
categories: Database
tag : [SQLD, SQL]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


# SQLD 55회 후기

SQLD 준비할 때 기출만 보고 싶었는데 개념이랑 같이 있길래 나쁠 것 없다 생각해서 SQLD의 모든 것 책 구매했다. 사실 표지가 깔끔해서 구매했고 노랭이가 이건 줄 알았다 : ) 근데 좀 설명이 부실하다는 느낌과 내용이 깔끔하지 못하고 허술하다 생각했는데 비전공자가 노베이스로 시작하기에는 좋았다는 말이 많았다.

처음 보는 문제인데 홍쌤 기출에 있었다는 글도 많이 봤고 모든것은 답지에만 설명되어있는 개념도 있어서 전반적으로 아쉬움이 많아서 개인적으로 비추.

이번 55회는 어렵다는 후기가 다소 많았는데 다들 벼락치기로 준비하신 분들이 많은 듯 했다 (준비 기간이 일주일도 안되는 …!)

내가 느끼기에도 좀 어려움이 있었고 고사장 책상이 수평이 흐트러져서 내 집중도 좀 흐트러져서 더 그랬던 것 같다. 그래서 그런지 어처구니 없는 실수도 한듯…

무튼, 그 중 가장 헷갈렸던 개념들만 정리하고 SQLD 는 여기서 마무리하려고 한다. (합격 맞겠지….😂?? )

## 시험 내용 정리

### 1️⃣ 오라클에서는 ORDER BY 사용 시 NULL 값은 최대 값으로 처리된다.

- 오름차순(`ASC`): NULL 값이 맨 마지막에 위치
- 내림차순(`DESC`): NULL 값이 맨 앞에 위치

### 2️⃣ `REGEXP_INSTR`

1. `REGEXP_INSTR`는 정규표현식을 기반으로 문자열에서 특정 위치를 반환
2. 문제
    
    ```sql
    SELECT REGEXP_INSTR('12345678', (123)(4(56)(78))',1, 1, 0, 'i', 2);
    ```
    
    - **문자열**: `'12345678'`
    - 검색할 정규 표현식 패턴 : ‘(123)(4(56)(78))’
        - 검색 시작 위치 1,
        - 검색 반복 횟수 1,
        - 반환 방식 0 → 패턴이 일치한 시작 위치를 반환하라는 의미
        - 대소문자 무시: ‘i’
        - 그룹 번호:  2 → 패턴의 두 번째 그룹이 일치하는 시작 위치를 반환하라는 의미
    - **패턴 일치**:
        - `'123'` → 첫 번째 그룹(일치).
        - `'4'` → 두 번째 그룹의 기본 값(일치).
        - `'56'` → 두 번째 그룹의 첫 번째 하위 그룹.
        - `'78'` → 두 번째 그룹의 두 번째 하위 그룹.
    - **그룹 번호 2**:
        - 두 번째 그룹 `(4(56)(78))`의 시작 위치를 반환.
        - 두 번째 그룹은 `'12345678'`에서 **4번 문자**인 `'4'`로 시작.
3. 답 : 두 번째 그룹의 시작 위치는 **4번째 문자**.

### 3️⃣ **무결성 제약조건의 유형**

1. **도메인 무결성 (Domain Integrity)**
    - 특정 컬럼에 허용되는 데이터의 유형과 범위를 제약.
    - 데이터가 정의된 범위 내에서만 입력되도록 보장.
    - 예: 데이터 타입, NULL 여부, 기본값, 제약조건 (`CHECK`).
2. **참조 무결성 (Referential Integrity)**
    - 외래키(Foreign Key)를 통해 테이블 간 관계를 유지.
    - 외래키 값은 참조하는 테이블의 기본키 값과 일치하거나 NULL이어야 함.
    - 데이터 삭제 또는 업데이트 시 참조 관계를 유지해야 함.
3. **개체 무결성 (Entity Integrity)**
    - 기본키(Primary Key)에 대한 제약조건.
    - 기본키는 **NULL을 허용하지 않으며, 고유해야 함**.
    - 테이블의 각 행(레코드)이 유일하게 식별될 수 있도록 보장.

4. **고유 무결성 (Unique Integrity)**
    - 특정 컬럼 값이 **중복되지 않도록 보장**.
    - 기본키 제약조건과 유사하지만, NULL 값은 허용될 수 있음.
    - 예: `UNIQUE` 제약조건 사용
5. **키 무결성 (Key Integrity)**
    - 기본키, 외래키, 대체키와 같은 키(key)를 이용해 테이블 내에서 데이터 간의 고유성과 참조 관계를 유지.
6. **비즈니스 무결성 (Business Integrity)**
    - 특정 비즈니스 규칙에 따라 데이터의 일관성을 보장.
    - 일반적으로 애플리케이션 레벨에서 구현되며, 데이터베이스의 트리거나 프로시저로도 처리 가능.
    - 예: "재고 수량이 음수가 될 수 없다"는 규칙.

### 4️⃣ **REGEXP_SUBSTR 문제**

```sql
SELECT REGEXP_SUBSTR('abc', 'b\*c') FROM dual;
SELECT REGEXP_SUBSTR('abc', 'b*c') FROM dual 
```

- 첫 번째 패턴: `'b\*c'`
    - \*는 문자 *를 의미하며, 'abc'에는 이 패턴이 없으므로 NULL 반환.
- 두 번째 패턴: `'b*c'`
- `*` 는 바로 앞 문자 `'b'`가 0번 이상 반복됨을 의미하며, `'bc'`와 매칭되어 `'bc'` 반환.

### 5️⃣ **COUNT(*) , COUNT(표현식) 에 NULL 이 포함되어 있으면?**

- `COUNT(*)`: **NULL 포함** 모든 행의 개수를 계산.
- `COUNT(expression)`: NULL 값을 제외한 행의 개수를 계산.

### 6️⃣ (A, ROLLUP(B))이랑 같은 것은?!

1. ROLL UP
    
    ```sql
    SELECT department_id, job_id, SUM(salary)
    FROM employees
    GROUP BY ROLLUP(department_id, job_id);
    ```
    
    - ROLLUP 은 특정 열의 조합에 대해 집계를 계산하고, 계층적으로 상위 수준의 집계를 추가로 계산한다.
2. GROUPING SETS
    
    ```sql
    SELECT department_id, job_id, SUM(salary)
    FROM employees
    GROUP BY GROUPING SETS (
        (department_id, job_id), -- department_id와 job_id별 집계
        (department_id),         -- department_id별 집계
        ()                       -- 전체 집계
    );
    ```
    
    - **사용자가 원하는 그룹 조합만** 지정해서 집계를 생성할 수 있다.
    - 자동 계층 집계가 아닌 명시적으로 그룹 조합을 나열 가능
3. 문제
    
    ```sql
    (A, ROLLUP(B))와 같은 표현은?
    1. GROUPING SETS (A, B, ())
    2. GROUPING SETS ((A, B), ())
    ```
    
    - GROUPING SETS ( A, B, ()) : (A, ROLLUP(B))
    - GROUPING SETS ( (A, B), ()) : **A와 B를 결합한 결과만 포함**하며, A 또는 B 개별 집계는 포함하지 않음.

### 7️⃣ SUM(), SUM() OVER

- **`SUM()`**
    - 집계 함수로 **하나의 행 반환**.
    - ex) 전체 급여 합산 결과 한 행으로 반환.
- **윈도우 함수 `SUM()`**
    - **모든 행에 대해 반환**.
    - ex) 각 행의 누적 합계를 계산.
- 예시
    
    ```sql
    -- 집계 함수
    SELECT SUM(salary) FROM employees;  
    
    -- 윈도우 함수
    SELECT SUM(salary) OVER (PARTITION BY department_id) 
    FROM employees;
    ```
    

### 8️⃣ fetch /only / ties

- `FETCH FIRST ... ROWS ONLY`
    - 쿼리 결과에서 처음 N개의 행만 반환.
- `FETCH FIRST ... ROWS WITH TIES`
    - 동일한 값이 있을 경우 **모든 행**을 반환.
- 예시
    
    ```sql
    SELECT * 
    FROM employees 
    ORDER BY salary DESC 
    FETCH FIRST 3 ROWS WITH TIES; -- 동점 포함 3명 반환 
    ```
    

---

시험 결과를 기다리는 동안 이번 경험을 바탕으로 헷갈렸던 개념과 문제를 정리했는데, 부디 합격이기를 🤲

그리고 아래는 SQLD 시험 준비할 때 참고하기 좋은 사이트입니당!😊<br>
[네이버카페 : 데이터전문가포럼](https://cafe.naver.com/sqlpd) <br>
[기출문제 :Study with yuna🌷님 블로그](https://yunamom.tistory.com/388)
{: .notice--warning}

<br>
<br>
<br>
<br>
<br>
