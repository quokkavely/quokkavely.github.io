---
layout : single
title : "[MVC] API 문서화 실습"
categories: Practice
tag : [Spring, 실습, Testing, 문서화]
author_profile: true
---


# API 문서화 실습

### 구현 조건

1. **`MemberController` 클래스에 대한 API 문서 생성용 테스트 케이스 작성**
    
    각 핸들러 메서드의 API 스펙 정보에 맞게 API 문서를 잘 생성하는지 확인
    
    [https://quokkavely.github.io/spring/TDD-Docs/](https://quokkavely.github.io/spring/TDD-Docs/) 여기에서 작성했던 Slice Test의 get/patch에 이어 controller의 핸들러 메서드 테스트 케이스 작성 후 문서화 하기
    
    - getMemberTest()
    - getMembersTest()
    - deleteMemberTest()
2. **생성한 문서 스니펫의 내용을 웹 브라우저로 확인하기.**
    
    

## 구현 코드

### getMemberTest()

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d4c8f05c-59db-451d-9f80-4b9f6b23457f"/>

- 지난 실습때 배웠던 것 처럼 given에서 memberService와 mapper는 Mockito이기 때문에 굳이 member에 하나하나 데이터를 담아서 넣을 필요 없이
- getMember는 urI에 memberId만 있으면 되기 때문에 requestBody에는 아무것도 담지 않아도 되고 pathVariable에 해당하는 memberId만 넣어주었다.

### deleteMemberTest()

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/1e1d5abe-10fb-4790-9227-bed47bd9d15a"/>

- **pathParameters**는 기존에 List.of로 작성했었는데 하나 밖에 없으면 굳이 안써도 될 것 같아서 사용하지 않았지만 구현체로 들어가보면  1개여도 여러개여도 반환 값은 List라서 명시적으로 사용해도 무방하다.
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/cad5adf7-7e5c-41e4-94ec-c996e62000b2"/>
    

### getMembersTest()

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3f80ca14-d480-4fca-8d21-52f638d58eab">

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c09aa3b3-a8a6-4ce4-9c0c-01a66c8ef219">

- 처음에는 index 넣으려다가 찾아보니까 data는 배열형태이고 이 배열 안에 요소를 가져얼 때는 data[ ]로 가져올 수 있다. 이건 문서화 할 때 문법이라고 생각하면 된다. 배열 안에 객체의 요소를 가져올 때 사용!

1. 오류 발생 
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/47c635b5-7c91-4b82-a423-1eb6d309e7e7">
    
    - pageInfo를 문서화할 때 하나하나 다 넣어주어야 하는데 넣지 않아서 생기는 오류

### 문서 스니펫을 웹 브라우저로 확인

1. src-docs-asciidoc에 index.adoc 생성
    
   <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b6794c7c-17dd-4686-93a0-ccaf329e18d2">
    

1. Testing 코드에서 and Do(Document ~ 로 생성된 파일을 index.adoc에 넣어준다.
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/ce8cb04f-fd21-4266-8564-ebaa2c17a72b" width=300>
    
2. index.adoc → 템플릿 문서에 내용 추가
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/1776a92a-e818-4fac-98a4-c60262f6f2eb" width=400>
    
3.  `:build` 또는 `:bootJar` task 명령을 실행해서 `index.adoc` 파일을 `index.html` 파일로 변환
    
    <img scr="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/dbd4fa65-beca-474b-bf3f-4ba74c0adf8a" width=400>
    
4. 완성
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/784cf966-fcd8-4f93-b901-9f1c796e6a22" width=400>