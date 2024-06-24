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

### Part 2.

```java
public class part2 {
  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-1. user 테이블에 존재하는 모든 컬럼을 포함한 모든 데이터를 확인하기 위한 SQL을 작성해주세요.
  */
  public static final String PART2_1 = "SELECT * from user";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-2. user 테이블에 존재하는 모든 데이터에서 name 컬럼만을 확인하기 위한 SQL을 작성해주세요.
  */
  public static final String PART2_2 = "SELECT name from user";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-3. user 테이블에 데이터를 추가하기 위한 SQL을 작성해주세요.
          - 원하는 name, email을 사용하시면 됩니다.
  */
  public static final String PART2_3 = "INSERT INTO user(name,email) VALUES('jerry','jerry@gmail.com');";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-4. user 테이블에서 특정 조건을 가진 데이터를 찾기위한 SQL을 작성해주세요.
          - 조건 : name이 luckykim이여야 합니다.
  */
  public static final String PART2_4 = "SELECT NAME FROM user WHERE NAME='luckykim';";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-5. user 테이블에서 특정 조건을 가진 데이터를 찾기위한 SQL을 작성해주세요.
          - 조건 : name이 luckykim이 아니여야 합니다.
  */
  public static final String PART2_5 = "SELECT NAME FROM user WHERE NOT NAME='luckykim';";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-6. content 테이블에 존재하는 모든 데이터에서 title 컬럼만을 찾기 위한 SQL을 작성해주세요.
  */

  public static final String PART2_6 = "SELECT title FROM content;";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-7. content의 title과 그 컨텐츠를 작성한 user의 name을 찾기 위한 SQL을 작성해주세요.
          - 저자가 없더라도, 켄턴츠의 title을 모두 찾아야합니다.
  */
  public static final String PART2_7 = "SELECT content.title, user.name FROM content LEFT JOIN user ON user.Id=content.userId;";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-8. content의 title과 그 컨텐츠를 작성한 user의 name을 찾기 위한 SQL을 작성해주세요.
          - 저자가 있는 컨텐츠의 title만 찾아야합니다.
  */
  public static final String PART2_8 = "SELECT content.title, user.name FROM content JOIN user ON user.Id=content.userId;";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-9. content의 데이터를 수정하기 위한 SQL을 작성해주세요.
          - title이 database homework인 content 데이터에서 body를 database is very easy로 수정해야합니다.
  */
  public static final String PART2_9 = "UPDATE content SET body = 'database is very easy' where title = 'database homework'; ";
  

  /* 강사님 답변 : UPDATE content AS c SET c.body = 'database is very easy' where c.title = 'database homework';
  ----------------------------------------------------------------------------------------------
      TODO: Q 2-10. content의 데이터를 추가하기 위한 SQL을 작성해주세요.
          - luckykim이 작성한 컨텐츠를 추가해주세요. 제목과 본문은 자유입니다. (참고: luckykim의 아이디는 1입니다.)
  */
  public static final String PART2_10 = "INSERT INTO content(title,body,userID) VALUES ('free','luckykim is free',1);";
}
```

### Part 3.

```java
public class part2 {

	/*
			TODO: Q 3-2-1. category 테이블에 존재하는 데이터에서 id, name을 찾는 SQL을 작성해주세요.
  */

  public static final String PART3_2_1 = "SELECT id, name FROM category ;";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-2. user의 name과 email 그리고 그 user가 속한 role name(컬럼명: roleName)을 찾기 위한 SQL을 작성해주세요.
          - 속한 role이 없더라도, user의 name과 email,role name을 모두 찾아야합니다.
  */
  public static final String PART3_2_2 = "SELECT user.name, user.email, role.name from user left join role on user.roleId=role.id;";

  /* user쪽으로 outer join 해야함.
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-3. 어느 role에도 속하지 않는 user의 모든 컬럼 데이터를 찾기위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_3 = "SELECT *from user WHERE roleId IS NULL; ";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-4. content_category 테이블에 존재하는 모든 칼럼의 데이터를 찾기위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_4 = "SELECT * from content_category;";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-5. minsanggu이 작성한 content의 title을 찾기위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_5 = "SELECT content.title from content JOIN user ON user.id =content.userId WHERE user.name='minsanggu';";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-6. minsanggu이 작성한 content의 category name을 찾기위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_6 = 
  
  "SELECT category.name from category  JOIN content_category 
  ON content_category.categoryId =category.id JOIN content ON content.id= content_category.contentId 
  JOIN user ON user.id= content.userId WHERE user.name='minsanggu';";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-7. category의 name이 java인 content의 title, body, created_at을 찾기위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_7 = 
  
  "SELECT content.title, content.body, content.created_at FROM content 
  JOIN content_category ON content.id=content_category.contentId 
  JOIN category ON category.id = content_category.categoryId 
  WHERE category.name ='java';";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-8. category의 name이 java인 content의 title, body, created_at, user의 name을 찾기위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_8 = "SELECT content.title,content.body,content.created_at,user.name from content 
  join content_category on content.id = content_category.contentId "+
          "join category on content_category.categoryId=category.id  
          join user on user.id = content.userId  WHERE category.name='java'; ";

  /*
  ----------------------------------------------------------------------------------------------
      TODO: Q 3-2-9. teawoongna가 작성한 글의 개수 (컬럼명: ContentCount)를 출력하기 위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_9 = "SELECT COUNT(*)AS ContentCount FROM CONTENT JOIN user ON user.id=content.user
Id WHERE user.name='teawoongna';";

  TODO: Q 3-2-10. 각 user(컬럼명: name)가 작성한 글의 개수 (컬럼명: ContentCount)를 출력하기 위한 SQL을 작성해주세요.
  */
  public static final String PART3_2_10 = "SELECT  user.name,COUNT(content.id) AS ContentCount FROM content RIGHT JOIN user ON user.id=content.userId GROUP BY user.name; ";
}
```

다른 것은 길이가 길어서 힘들고 오탈자만 아니면 금방 통과했는데

3-2-9랑 10은 COUNT를 안배워서 조금 생각하고 찾아봐야 했다.

**3-2-9 의 내 코드**

```sql
SELECT COUNT(*) AS ContentCount FROM CONTENT
JOIN user
ON user.id=content.userId
WHERE user.name='teawoongna';
```

- 3-2-9  Reference
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled.png)
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%201.png)
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%202.png)
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%203.png)
    

강사님은 대부분 AS로 약어 처럼 엔터티명을 사용함.

대부분 실무에 가면 이렇게 작성 많이 한다고 해서 이렇게 쓰는 것을 권장.

**3-2-10 내코드**

```sql
SELECT  user.name, COUNT(content.id) AS ContentCount
FROM content RIGHT JOIN user
ON user.id=content.userId
GROUP BY user.name;
```

- **3-2-10 Reference**
    
    count는 보통 GROUP BY 와 같이 사용되고
    
    처음에 9번처럼 count(*)사용해서
    
    SELECT u.name, count(*)FROM content RIGHT JOIN ~
    
    처럼 작성했는데 u.name과 content가 같이 사용될 수 없어서 *를 사용하면
    
    안된다고 한다.
    
    count 쓸때 별을 넣으면 null일 경우 1을 반환.
    
    특정 컬럼을 기준으로 해야 0을 반환함.
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%204.png)
    
    c.* 해도 안됨.!
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%205.png)
    
    정답
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%206.png)
    

- 이외 참고해서 볼 것. = TEST CODE
    
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%207.png)
    
    Mysql - single ton pattern
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%208.png)
    
    null이면 새로만들고 널이 없으면 반환하겠따.
    
    데이터 베이스 접속  JDBC drivers.
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%209.png)
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2010.png)
    
    원래는 Result set으로 변환해서 받지만
    
    테이블에서는 map과 비슷 해서 내부에는 결과를 MAP에 맵핑 , key value 사용
    
    FactoryService 검증하는 클래스
    
    ![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2011.png)
    
    preparedStatement 
    
    DTO
    
    FACTORY SERVICE 뜯어보는 것 좋음.
    

## Comment

대부분의 문제풀고 테스트하는 시간이라서

문제 많이 못풀었으면 현타왔을텐데

이제는 이런 문제들에 단련이 되어서 그런가

풀만해서 기분이 좋다....😊

---

### **GROUP BY, ORDER BY**

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2012.png)

GROUP BY는 그룹컬럼

ORDER BY는 정렬컬럼

---

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2013.png)

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2014.png)

직업(job) 별로 급여(sal) 총합계를 구하는 예제이다.

---

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2015.png)

그룹 칼럼이 여러 개인 경우 첫 번째 칼럼(deptno)으로 먼저 그룹이 묶이고, 두 번째 칼럼(job)으로 집계가 된다.

COUNT

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2016.png)

***✨별표 *는 모든 행 , NULL이 있으면 값을 1로 가져옴***

컬럼을 사용할 경우 NULL이 0으로 가져오거나 제외 될 수도 있음.

SUM, AVG

SUM 함수는 해당 칼럼의 모든 값을 합산하며, 수치형 칼럼에만 사용할 수 있다.

AVG 함수는 칼럼 값의 평균을 구하며, 칼럼의 값이 NULL인 경우 제외를 하고 연산을 하니 주의해야 한다.

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2017.png)

MIN, MAX

MIN 함수는 해당 칼럼의 최솟값, MAX 함수는 해당 칼럼의 최댓값을 구한다.

![Untitled](blog%203ab7de4abe5149f98951e906bcacbbb1/Untitled%2018.png)