---
layout : single
title : "[aws] deploy 실습"
categories: cloud
tag : [AWS, 실습, cloud]
author_profile: true
---


# 배포 실습

### EC2 생성

<img src = "https://github.com/user-attachments/assets/22a2801a-5ed8-4f62-a437-73bdafa09c35" width =500/>

키 페어생성 누르면 key가 다운 받아짐. 내가 알고 있는 폴더에 넣어두기.

다른 것 안건들이고 인스턴스 시작

### 버킷 생성

<img src = "https://github.com/user-attachments/assets/600f248e-7406-47b2-b26c-484a394ff594" width =500/>

리전 확인해서 서울이 아니면 다시 설정.

<img src = "https://github.com/user-attachments/assets/f780d19b-a601-4fc1-9445-10a5a53744be" width =500/>

퍼블릭 액세스 차단 설정 - 모든 퍼블릭 액세스 차단 체크 해제, 노란박스는 체크하기.

### 데이터베이스생성

1. 표준생성 > MySQL > 템플릿 프리티어  설정
    
    <img src="https://github.com/user-attachments/assets/a3423fbf-f9a7-4b85-bc3b-e67b57444cc6" width =500/>
    
    <img src="https://github.com/user-attachments/assets/295d9d7b-f2dc-4c17-8262-cf2351e16e0d" width =500/>
    
    - 템플릿은 무조건 프리티어로 설정한다. 특히 RDS는 비쌈  프리티어 표시 꼭 확인하기.
2. 인스턴스식별자 (내가 구분할 수 있게 ) , 자격증명 설정에 사용자 이름은 admin으로 설정, 암호 설정
    
    <img src="https://github.com/user-attachments/assets/55356514-8ab8-4dec-bf06-611b4cbc7707" width =500/>
    
    - 암호 적기 기억하기 쉬운 것으로!
3. 기본 설정으로 유지하고 퍼블릭 액세스만 예 체크하기
    
    <img src="https://github.com/user-attachments/assets/cf1f78bf-3639-40ee-9858-4810860f8edb" width =500/>
    

1. 추가 구성 토글 열어서  13306 설정하기 (3306으로 하면 보안에 좋지 않다..)
    
    <img src="https://github.com/user-attachments/assets/577e1459-ada6-43b8-902d-4f7764bc0b0a" width =500/>
    
2. 모니터링 아래에 추가 구성 토글 열어서  데이터페이스 옵션에 이름 생성
    
    <img src="https://github.com/user-attachments/assets/612f428a-db17-4f5f-a907-8c576ac4091a" width =500/>
    

1. 그다음 데이트베이스 생성 클릭
    
    <img src="https://github.com/user-attachments/assets/2a94ac96-ba64-4070-b795-3b6b9b8ee371" width =500/>
    

### EC2에서 인스턴스 연결 규칙 추가 및 연결

EC2에서 인스턴스 연결 규칙 추가
<img src="https://github.com/user-attachments/assets/d4c65420-2202-41be-8be9-81dd6c1db9e1" width =500/>


EC2는 껏다 키면 IP변경 되므로 껏다키지말기

<img src="https://github.com/user-attachments/assets/7b6844b3-e4e3-4921-b531-6a3b91431c63" width =500/>

여기 적혀있는 거 복사 후 명령프롬프트로 이동해서 아래  1번처럼  key 실행

1. C:\AWS>attrib +r AWS_KEY.pem (AWS_KEY 는 생성한 키 이름 , 새로 만들었던 이름으로 변경 해서 넣을 것)
2. Warning 발생하면 아래 명령어 입력.
3. icacls.exe AWS_KEY.pem /reset
4. icacls.exe AWS_KEY.pem /grant %username%:(R)
5. icacls.exe AWS_KEY.pem /inheritance:r
6. 작성 후 ssh -i 로 시작하는 위의 인스턴스 주소를 복사해서 붙여넣기.
7. 예시
    
    <img src="https://github.com/user-attachments/assets/bbfd33c3-8ad1-415e-a2b7-28b12e09bc5b" width =500/>

    <img src="https://github.com/user-attachments/assets/8d691c6b-bbe7-449e-adb1-8e4522de4fdc" width =500/>

1. apt
2. sudo apt update
3. sudo apt install openjdk-11-jre-headless 후에
4. java -version 명령어 입력 후 자바버전 확인하기.
5. 예시
    
    
    <img src="https://github.com/user-attachments/assets/b4d33437-48be-4f8a-9892-304338811a1c" width =500/>
   
    <img src="https://github.com/user-attachments/assets/5deeb5d7-98d5-4598-af0c-dc9287731da6" width =500/>
    <img src="https://github.com/user-attachments/assets/3a5f6a81-a332-4c0c-bb12-79ce8ae5eea2" width =500/>
    <img src="https://github.com/user-attachments/assets/a905b0e8-5cb6-4bb6-9038-2da7eec244b9" width =500/>

### 배포 파일-resource-property 수정

<img src="https://github.com/user-attachments/assets/9161cf76-5382-415c-8cc3-ffd2ca82c742" width =500/>

1. database (spring.datasource.url에  RDS 엔드포인트 복사해서 붙여넣기.)
    - RDS-데이터베이스- 만들었던 DB 클릭 후 엔드포인트 복사하기.
        
       <img src="https://github.com/user-attachments/assets/eab6d13a-35e1-4b03-99c1-4f64e2cb0814" width =500/>
        
2.  config.domain에 S3 - 속성 - 정적웹사이트호스팅 활성화 후 버킷 웹사이트 엔드포인트 복사 후 붙여넣기 
    - S3 - 속성 - 정적웹사이트호스팅 편집
        
    <img src="https://github.com/user-attachments/assets/61e418ee-862b-49a9-ae34-bb6dded5e0a5" width =500/>
        
    활성화하고 인덱스 문서에 페이지 지정하기.
        
    <img src="https://github.com/user-attachments/assets/562f6a8e-78c0-4feb-a066-a26bcbeb4708" width =500/>
        
    이 주소를 config.domain에 붙어넣기하기
        

1. Gradle-build-bootJar 클릭
    
    <img src="https://github.com/user-attachments/assets/afbaa511-9125-452f-a638-a88c24391a38" width =500/>
    
    <img src="https://github.com/user-attachments/assets/3fd0de20-c48c-4797-8d0f-3be4881328bd" width =500/>
    
2. build 한 파일 AWS 키 있는 위치로 옮기기.
    
    ```java
    C:\AWS>scp -i AWS_KEY.pem DeployServer-0.0.1-SNAPSHOT.jar ubuntu@ec2-3-38-246-253.ap-northeast-2.compute.amazonaws.com: DeployServer-0.0.1-SNAPSHOT.jar  
    ```
    
    //내 키이름 +  배포파일 이름 + EC2 인스턴스연결 주소 
    
    <img src="https://github.com/user-attachments/assets/cfb2ecf7-1a91-4558-88f3-80278307cf87" width =500/>
    
    우분투에서 ls 해서 파일이 이동했는지 확인 한 다음
    
    <img src="https://github.com/user-attachments/assets/f931c2f9-87ee-4bcf-91e5-49b180b4789c" width =500/>
    
    파일 실행 명령어 입력 →  java -jar DeployServer-0.0.1-SNAPSHOT.jar
    

### 포스트맨 실행

EC2의 퍼블릭IPv4 복사 후  > 주소+ :8080 으로 요청하면 된다.

<img src="https://github.com/user-attachments/assets/0a7ad4fd-cefc-4fdc-a2fa-377da3d75407" width =500/>

1. /signin 으로 Post시 나온 token을 복사
2. /status 입력 후 key에 Authorization, value 에 token 값 붙여넣기  후 get요청
    
    <img src="https://github.com/user-attachments/assets/f95fdcaa-9c6c-40b0-8016-bb31f005c796" width =500/>
    
3. 결과
    
    <img src="https://github.com/user-attachments/assets/e4009645-6fe1-4259-8a64-c9a1310841e4" width =500/>
    

### html에 EC2 주소 넣고 버킷 정책 편집

1. index.html 파일에  url 넣기.
    
    <img src="https://github.com/user-attachments/assets/79b61d70-a7dd-4bfb-8a37-778ba80f1979" width =500/>
    
    파일을 모두 S3에 업로드 해야한다.
    
    <img src="https://github.com/user-attachments/assets/24504169-ca54-43ca-9873-35a7b3b60c07"  width =500/>
    
2. 업로드
    
    <img src="https://github.com/user-attachments/assets/631210a9-6d22-47d3-9b4d-b318473d35fa" width =500/>
    
3. 버킷 > 권한 > 버킷 정책 편집
    <img src="https://github.com/user-attachments/assets/46770870-341d-4dc6-a889-93b81debce55" width =500/>
    
    - ARN 복사 > 정책 생성기
    <img src="https://github.com/user-attachments/assets/757bd193-14eb-4660-9e74-142ac442934b"  width =500/>
        

1. 정책생성기 
    <img src="https://github.com/user-attachments/assets/000e2db1-d977-48bc-a541-5f1d0ad2c0a0" width =500/>
    <img src="https://github.com/user-attachments/assets/e086edd9-7dc6-4b5a-91ea-b07e9f2af440" width =500/>
    
    - 생성 확인 후 Generate Policy 클릭
    
    <img src="https://github.com/user-attachments/assets/30becf42-3f44-4b43-885e-487d607238fc" width =500/>
    
    - 뜨는 JSON 복사 후 정책에 붙여 넣기
        
        <img src="https://github.com/user-attachments/assets/08632d29-c787-47b1-99de-4fdffb5708f7 " width =500/>
        
        <img src="https://github.com/user-attachments/assets/8fc223e6-1ceb-4904-9440-b6d12eb1e01a" width =500/>