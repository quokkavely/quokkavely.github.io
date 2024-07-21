---
layout : single
title : "[SQL] Practice 1"
categories: Database
tag : [DBMS, SQL, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## SQL 연습문제

[https://www.w3schools.com/sql/exercise.asp?filename=exercise_select1](https://www.w3schools.com/sql/exercise.asp?filename=exercise_select1)

기본적인 문제이고 칸크기도 답에 맞춰져 있어서

문제풀면서 아 이렇게 쓰는거구나 하기 좋다...!!!

GROUP BY랑 와일드카드는 나중에 배운다해서

잘 모르겠지만 GROUP BY는 풀어볼만해서 풀어봄...

그리고 연습문제 맨마지막에 GROUP BY 써야 될 수도 있어서 참고하기!
<img src="https://github.com/user-attachments/assets/2ee0818f-f355-41ca-880d-c3e303dc67d0" width=200/>

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

**3-2-9**

3-2-9. teawoongna가 작성한 글의 개수 (컬럼명: ContentCount)를 출력하기 위한 SQL을 작성

- **내 코드**

```sql
SELECT COUNT(*) AS ContentCount FROM CONTENT
JOIN user
ON user.id=content.userId
WHERE user.name='teawoongna';
```

- **Reference**
  
    <img src="https://github.com/user-attachments/assets/e25f7076-7f12-4433-8f9c-e7db6e43cb33" width=400>
    
    <img src="https://github.com/user-attachments/assets/3327a1bc-928e-424e-97c4-18cfdbb3f24f" width=400>
    
    <img src="https://github.com/user-attachments/assets/84f519a4-76ed-4d30-b892-b0b65d6ccdf5" width=400>
    
    <img src="https://github.com/user-attachments/assets/2d329e3d-6f78-46ba-8f62-fe2505e969d5" width=400>
    

AS로 약어 처럼 엔터티명을 사용함.
대부분 실무에 가면 이렇게 작성 많이 한다고 해서 이렇게 쓰는 것을 권장.

**3-2-10**

각 user(컬럼명: name)가 작성한 글의 개수 (컬럼명: ContentCount)를 출력하기 위한 SQL을 작성

- **내코드**

```sql
SELECT  user.name, COUNT(content.id) AS ContentCount
FROM content RIGHT JOIN user
ON user.id=content.userId
GROUP BY user.name;
```

- **Reference**
  
    count는 보통 GROUP BY 와 같이 사용되고 처음에 9번처럼 count(*)사용해서 <br/>
    SELECT u.name, count(*)FROM content RIGHT JOIN ~처럼 작성했는데 <br/>
    u.name과 content가 같이 사용될 수 없어서 *를 사용하면 안된다고 한다. <br/>
    count 쓸때 별을 넣으면 null일 경우 1을 반환. <br/>
    특정 컬럼을 기준으로 해야 0을 반환함. <br/>
    
    <img src="https://github.com/user-attachments/assets/93141232-534f-49e2-8fb6-7eb81514e116" width=400>
    
    c.* 해도 안됨.!
    
    <img src="https://github.com/user-attachments/assets/5772e6be-ded5-40e9-8d23-947ae8848019" width=400>
    
    정답
    
    <img src="https://github.com/user-attachments/assets/7aa7ee44-af5f-49a5-b563-696f8dbae3c9" width=400>
    

    
    
    

## Comment

대부분의 문제풀고 테스트하는 시간이라서 문제 많이 못풀었으면 현타왔을텐데 <br/>
이제는 이런 문제들에 단련이 되어서 그런가 풀만해서 기분이 좋다....😊 <br/>

---