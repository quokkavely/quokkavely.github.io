---
layout : single
title : "[MVC] 애플리케이션 빌드/실행/배포 실습"
categories: Practice
tag : [Spring, Practice, JPA]
author_profile: true
---


📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}



## 애플리케이션 빌드/실행/배포 실습

1. **MySQL 로그인 후 database 생성 → example**
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b7c8ef1a-0e9d-49f3-90e8-ff365d2c9f1b" width=200 >
    
2. **Build.gradle에 의존성 추가**
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b5348ca5-f095-4caf-9b11-596218ed7036" width=200>
    
3. **application-server.yml 파일 작성 - datasorce 부분 추가**
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/2cd2cd13-5dc7-45ed-979b-c93491d94f55" width=400>
    
4. **profile 설정**
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/8f4ccafa-a6f9-4877-bd86-e995eee3f4f8" width=400>
    
5. **에러 발생**
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/bc3a746a-c305-4761-92a9-40b168203160">
    
    - 아무리 시도해도 안되고 에러메세지도 이게 다 인가? 하고 계속 원인 찾다가 위로 올려 봤더니 비밀번호에서 에러가 났다.
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/aea17c04-8743-47dc-b801-79e132e49668"witdh=400/>
    
    - sql에서는 로그인 잘하고 DB도 새로 만들었는데 왜 자꾸 비밀번호에서 안되나 했더니 첫번째 문자가 특수문자로 되어있는데 여기서 문자열로 인식을 못한다고 함…! 그래서 “”로 감싸주어야 한다.

6. **PostMan 요청** <br/>
    localhost:3306으로 요청해야 하나 고민했는데 8080으로 요청하면 된다.
    

7. **Dbeaver로 데이터 확인**