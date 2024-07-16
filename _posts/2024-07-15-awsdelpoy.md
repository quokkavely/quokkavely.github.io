---
layout : single
title : "aws로 deploy 실습"
categories: cloud
tag : [AWS, Practice, cloud]
author_profile: true
---


# 배포 실습

### EC2 생성

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled.png)

키 페어생성 누르면 key가 다운 받아짐. 내가 알고 있는 폴더에 넣어두기.

다른 것 안건들이고 인스턴스 시작

### 버킷 생성

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%201.png)

리전 확인해서 서울이 아니면 다시 설정.

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%202.png)

퍼블릭 액세스 차단 설정 - 모든 퍼블릭 액세스 차단 체크 해제, 노란박스는 체크하기.

### 데이터베이스생성

1. 표준생성 > MySQL > 템플릿 프리티어  설정
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%203.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%204.png)
    
    - 템플릿은 무조건 프리티어로 설정한다. 특히 RDS는 비쌈  프리티어 표시 꼭 확인하기.
2. 인스턴스식별자 (내가 구분할 수 있게 ) , 자격증명 설정에 사용자 이름은 admin으로 설정, 암호 설정
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%205.png)
    
    - 암호 적기 기억하기 쉬운 것으로!
3. 기본 설정으로 유지하고 퍼블릭 액세스만 예 체크하기
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%206.png)
    

1. 추가 구성 토글 열어서  13306 설정하기 (3306으로 하면 보안에 좋지 않다..)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%207.png)
    
2. 모니터링 아래에 추가 구성 토글 열어서  데이터페이스 옵션에 이름 생성
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%208.png)
    

1. 그다음 데이트베이스 생성 클릭
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%209.png)
    

### EC2에서 인스턴스 연결 규칙 추가 및 연결

EC2에서 인스턴스 연결 규칙 추가

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2010.png)

EC2는 껏다 키면 IP변경 되므로 껏다키지말기

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2011.png)

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2012.png)

여기 적혀있는 거 복사 후 명령프롬프트로 이동해서 아래  1번처럼  key 실행

1. C:\AWS>attrib +r AWS_KEY.pem (AWS_KEY 는 생성한 키 이름 , 새로 만들었던 이름으로 변경 해서 넣을 것)
2. Warning 발생하면 아래 명령어 입력.
3. icacls.exe AWS_KEY.pem /reset
4. icacls.exe AWS_KEY.pem /grant %username%:(R)
5. icacls.exe AWS_KEY.pem /inheritance:r
6. 작성 후 ssh -i 로 시작하는 위의 인스턴스 주소를 복사해서 붙여넣기.
7. 예시
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2013.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2014.png)
    

1. apt
2. sudo apt update
3. sudo apt install openjdk-11-jre-headless 후에
4. java -version 명령어 입력 후 자바버전 확인하기.
5. 예시
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/cf8fb6d3-7f1d-4984-b951-45c678e0563c.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2015.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2016.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2017.png)
    

### 배포 파일-resource-property 수정

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2018.png)

1. database (spring.datasource.url에  RDS 엔드포인트 복사해서 붙여넣기.)
    - RDS-데이터베이스- 만들었던 DB 클릭 후 엔드포인트 복사하기.
        
        ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2019.png)
        
2.  config.domain에 S3 - 속성 - 정적웹사이트호스팅 활성화 후 버킷 웹사이트 엔드포인트 복사 후 붙여넣기 
    - S3 - 속성 - 정적웹사이트호스팅 편집
        
        ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2020.png)
        
        활성화하고 인덱스 문서에 페이지 지정하기.
        
        ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2021.png)
        
        이 주소를 config.domain에 붙어넣기하기
        

1. Gradle-build-bootJar 클릭
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2022.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2023.png)
    
2. build 한 파일 AWS 키 있는 위치로 옮기기.
    
    ```java
    C:\AWS>scp -i AWS_KEY.pem DeployServer-0.0.1-SNAPSHOT.jar ubuntu@ec2-3-38-246-253.ap-northeast-2.compute.amazonaws.com: DeployServer-0.0.1-SNAPSHOT.jar  
    ```
    
    //내 키이름 +  배포파일 이름 + EC2 인스턴스연결 주소 
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2024.png)
    
    우분투에서 ls 해서 파일이 이동했는지 확인 한 다음
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2025.png)
    
    파일 실행 명령어 입력 →  java -jar DeployServer-0.0.1-SNAPSHOT.jar
    

### 포스트맨 실행

EC2의 퍼블릭IPv4 복사 후  > 주소+ :8080 으로 요청하면 된다.

![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2026.png)

1. /signin 으로 Post시 나온 token을 복사
2. /status 입력 후 key에 Authorization, value 에 token 값 붙여넣기  후 get요청
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2027.png)
    
3. 결과
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2028.png)
    

### html에 EC2 주소 넣고 버킷 정책 편집

1. index.html 파일에  url 넣기.
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2029.png)
    
    파일을 모두 S3에 업로드 해야한다.
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2030.png)
    
2. 업로드
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2031.png)
    
3. 버킷 > 권한 > 버킷 정책 편집
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2032.png)
    
    - ARN 복사 > 정책 생성기
        
        ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2033.png)
        

1. 정책생성기 
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2034.png)
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2035.png)
    
    - 생성 확인 후 Generate Policy 클릭
    
    ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2036.png)
    
    - 뜨는 JSON 복사 후 정책에 붙여 넣기
        
        ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2037.png)
        
        ![Untitled](%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%20%E1%84%8B%E1%85%A5%E1%86%B9%E1%84%8B%E1%85%B3%E1%86%B7%2040446f8c5e394a999d9d9c7dc422bdb2/Untitled%2038.png)