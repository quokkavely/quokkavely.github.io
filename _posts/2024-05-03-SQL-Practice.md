---
layout : single
title : "[SQL] Practice"
categories: Databass
tag : [DBMS, SQL, 실습]
author_profile: true
---

## SQL 연습문제

[https://www.w3schools.com/sql/exercise.asp?filename=exercise_select1](https://www.w3schools.com/sql/exercise.asp?filename=exercise_select1)

기본적인 문제이고 칸크기도 답에 맞춰져 있어서

문제풀면서 아 이렇게 쓰는거구나 하기 좋다...!!!

GROUP BY랑 와일드카드는 나중에 배운다해서

잘 모르겠지만 GROUP BY는 풀어볼만해서 풀어봄...

그리고 연습문제 맨마지막에 GROUP BY 써야 될 수도 있어서 참고하기!
<img src="" width=500/>

## 연습문제

![https://postfiles.pstatic.net/MjAyNDA1MDNfMjIg/MDAxNzE0NzI3ODEzNjI5.hizrZ-w2O-lF9KDtG-mFyzPviUFqF1C9BzaTKw0nnyIg.12F_xCQlOIAOt3dKau-5EdR6rBQ1ciqL4kVuTKK8he4g.PNG/image.png?type=w580](https://postfiles.pstatic.net/MjAyNDA1MDNfMjIg/MDAxNzE0NzI3ODEzNjI5.hizrZ-w2O-lF9KDtG-mFyzPviUFqF1C9BzaTKw0nnyIg.12F_xCQlOIAOt3dKau-5EdR6rBQ1ciqL4kVuTKK8he4g.PNG/image.png?type=w580)

### schema 작성 : ERD보고 테이블 만들기

```java
CREATE TABLE `user` (
  `id` int PRIMARY KEY AUTO_INCREMENT,
  `name` varchar(255) not NULL,
  `email` varchar(255) not NULL
);

CREATE TABLE `content` (
  `id` INT AUTO_INCREMENT  PRIMARY KEY ,
  `title`  varchar(255) ,
  `body` varchar(255),
  `created_at` timestamp not NULL DEFAULT CURRENT_TIMESTAMP, // DEFAULT CURRENT_TIMESTAMP : 아무값도 없으면 현재시간이 들어옴
  `userId` int,
  FOREIGN KEY (`userId`) REFERENCES `user` (`id`)
);

CREATE TABLE `category` (
  `id` INT PRIMARY KEY AUTO_INCREMENT NOT NULL ,
  `name`  varchar(255) NOT NULL
);
CREATE TABLE `content_category` (
  `id` INT PRIMARY KEY AUTO_INCREMENT NOT NULL ,
    `contentId` INT NOT NULL,
     `categoryId` INT NOT NULL,
  FOREIGN KEY (`contentId`) REFERENCES `content` (`id`),
  FOREIGN KEY (`categoryId`) REFERENCES `category` (`id`)
);
CREATE TABLE `role` (
 `id` INT AUTO_INCREMENT  PRIMARY KEY NOT NULL,
   `name`  varchar(255) NOT NULL
);

ALTER TABLE `user` ADD roleId int;
ALTER TABLE `user` ADD FOREIGN KEY (`roleId`) REFERENCES `role` (`id`);
```

- 사실 조인테이블 작성은 많이 해봤는데 CREATE까지는 많이 안해봐서 하다가 좀 헷갈렸는데 헷갈렸던 부분은 FOREIGN KEY 작성하는 것.
- 참조키 생성하고 해당 키가 어디서 오는 참조키라고 다는 건데 참조키 속성에 안만들고 바로 FOREIGN KEY부터 만들었다가 테스트 통과 안돼서 다시 생각해서 만들었다..

### PART 1.

모든 테이블의 정보를 불러올 때는 **SHOW TABLES;**

어떤 테이블의 구조를 볼 때는 **DESC 테이블명;**

DESC는 DESCRIBE의 약어이다.

헷갈려서 찾아봤더니 **내림차순 명령어랑 똑같음**...> DESC (Descending)

### part 2

part 2는 간단해서 금방 품.

### Part 3.

다른 것은 길이가 길어서 힘들고 오탈자만 아니면 금방 통과했는데

3-2-9랑 10은 COUNT를 안배워서 조금 생각하고 찾아봐야 했다.

**3-2-9 **

3-2-9. teawoongna가 작성한 글의 개수 (컬럼명: ContentCount)를 출력하기 위한 SQL을 작성

- **내 코드**

```sql
SELECT COUNT(*) AS ContentCount FROM CONTENT
JOIN user
ON user.id=content.userId
WHERE user.name='teawoongna';
```

- **Reference**
  
    <img src="" width=400>
    
    <img src="" width=400>
    
    <img src="" width=400>
    
    <img src="" width=400>
    

강사님은 대부분 AS로 약어 처럼 엔터티명을 사용함.

대부분 실무에 가면 이렇게 작성 많이 한다고 해서 이렇게 쓰는 것을 권장.

**3-2-10 **

각 user(컬럼명: name)가 작성한 글의 개수 (컬럼명: ContentCount)를 출력하기 위한 SQL을 작성

- **내코드**

```sql
SELECT  user.name, COUNT(content.id) AS ContentCount
FROM content RIGHT JOIN user
ON user.id=content.userId
GROUP BY user.name;
```

- **Reference**
  
    count는 보통 GROUP BY 와 같이 사용되고
    
    처음에 9번처럼 count(*)사용해서
    
    SELECT u.name, count(*)FROM content RIGHT JOIN ~
    
    처럼 작성했는데 u.name과 content가 같이 사용될 수 없어서 *를 사용하면
    
    안된다고 한다.
    
    count 쓸때 별을 넣으면 null일 경우 1을 반환.
    
    특정 컬럼을 기준으로 해야 0을 반환함.
    
    <img src="" width=400>
    
    c.* 해도 안됨.!
    
    <img src="" width=400>
    
    정답
    
    <img src="" width=400>
    
- 이외 참고해서 볼 것. = TEST CODE
  
  
    <img src="" width=400>
    
    Mysql - single ton pattern
    
    <img src="" width=400>
    
    null이면 새로만들고 널이 없으면 반환하겠따.
    
    데이터 베이스 접속  JDBC drivers.
    
    <img src="" width=400>
    
    <img src="" width=400>
    
    원래는 Result set으로 변환해서 받지만
    
    테이블에서는 map과 비슷 해서 내부에는 결과를 MAP에 맵핑 , key value 사용
    
    FactoryService 검증하는 클래스
    
    <img src="" width=400>
    
    
    

## Comment

대부분의 문제풀고 테스트하는 시간이라서

문제 많이 못풀었으면 현타왔을텐데

이제는 이런 문제들에 단련이 되어서 그런가

풀만해서 기분이 좋다....😊

---

### **GROUP BY, ORDER BY**

<img src="" width=400>

GROUP BY는 그룹컬럼

ORDER BY는 정렬컬럼

---

<img src="" width=400>

<img src="" width=400>

직업(job) 별로 급여(sal) 총합계를 구하는 예제이다.

---

<img src="" width=400>

그룹 칼럼이 여러 개인 경우 첫 번째 칼럼(deptno)으로 먼저 그룹이 묶이고, 두 번째 칼럼(job)으로 집계가 된다.

COUNT

<img src="" width=400>

***✨별표 *는 모든 행 , NULL이 있으면 값을 1로 가져옴***

컬럼을 사용할 경우 NULL이 0으로 가져오거나 제외 될 수도 있음.

SUM, AVG

SUM 함수는 해당 칼럼의 모든 값을 합산하며, 수치형 칼럼에만 사용할 수 있다.

AVG 함수는 칼럼 값의 평균을 구하며, 칼럼의 값이 NULL인 경우 제외를 하고 연산을 하니 주의해야 한다.

<img src="" width=400>

MIN, MAX

MIN 함수는 해당 칼럼의 최솟값, MAX 함수는 해당 칼럼의 최댓값을 구한다.

<img src="" width=400>