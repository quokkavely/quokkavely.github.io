---
layout : single
title : "[aws] 프로젝트 ci-cd 구축"
categories: cloud
tag : [AWS, 실습, cloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 배포 실습

## EC2 생성

> 🪄  **핵심 요약** 인스턴스 시작
✅애플리케이션 - ubuntu - 22.04 (프리티어!!!)
✅키페어 - 새 키페어 생성 - 사용할 폴더로 이동
✅인스턴스 시작
> 
<img src = "https://github.com/user-attachments/assets/b2c10f7a-7a06-4cd8-b12e-c31d96d3a566" width =500/>
<img src = "https://github.com/user-attachments/assets/22a2801a-5ed8-4f62-a437-73bdafa09c35" width =500/>

키 페어생성 누르면 key가 다운 받아짐. 내가 알고 있는 폴더에 넣어두기.

다른 것 안건들이고 인스턴스 시작

## 버킷 생성

> 🪄  **핵심 요약**
✅버킷 이름은 중복이 안됨
✅ 리전은 서울이어야 한다.
✅모든 퍼블릭 액세스 차단 - 체크 해제
> 

<img src = "https://github.com/user-attachments/assets/600f248e-7406-47b2-b26c-484a394ff594" width =500/>

리전 확인해서 서울이 아니면 다시 설정.

<img src = "https://github.com/user-attachments/assets/f780d19b-a601-4fc1-9445-10a5a53744be" width =500/>

퍼블릭 액세스 차단 설정 - 모든 퍼블릭 액세스 차단 체크 해제, 노란박스는 체크하기.

## 데이터베이스생성

> 🪄 **핵심 요약**
> 
> 
> ✅엔진유형 - MySQL
> ✅템플릿 - 프리티어!!!!!!! (이거 잘못누르면 비쌈)
> ✅마스터 암호 (8자이상, 기억하기 쉬운걸로)
> ✅연결 - 퍼블릭엑세스 → 예
> ✅연결 - 추가구성 - 데이터베이스 포트 → 13306
> ✅모니터링 아래 추가구성 - 초기 데이터베이스 이름 → test
> ✅데이터베이스 생성
> 
1. 표준생성 > MySQL > 템플릿 프리티어  설정
    
    <img src="https://github.com/user-attachments/assets/a3423fbf-f9a7-4b85-bc3b-e67b57444cc6" width =500/>
        
    <img src="https://github.com/user-attachments/assets/295d9d7b-f2dc-4c17-8262-cf2351e16e0d" width =500/>
    
    - 템플릿은 무조건 프리티어로 설정한다. 특히 RDS는 비쌈  프리티어 표시 꼭 확인하기

2. 인스턴스식별자 (내가 구분할 수 있게 ) , 자격증명 설정에 사용자 이름은 admin으로 설정, 암호 설정
    
    <img src="https://github.com/user-attachments/assets/55356514-8ab8-4dec-bf06-611b4cbc7707" width =500/>
    
    - 암호 적기 기억하기 쉬운 것으로!
3. 기본 설정으로 유지하고 퍼블릭 액세스만 예 체크하기
    
    <img src="https://github.com/user-attachments/assets/cf1f78bf-3639-40ee-9858-4810860f8edb" width =500/>
    
    <br/>

4. 추가 구성 토글 열어서  13306 설정하기 (3306으로 하면 보안에 좋지 않다..)
    
    <img src="https://github.com/user-attachments/assets/577e1459-ada6-43b8-902d-4f7764bc0b0a" width =500/>
    
5. 모니터링 아래에 추가 구성 토글 열어서  데이터페이스 옵션에 이름 생성
    
    <img src="https://github.com/user-attachments/assets/612f428a-db17-4f5f-a907-8c576ac4091a" width =500/>
    

6. 그다음 데이트베이스 생성 클릭
    
    <img src="https://github.com/user-attachments/assets/2a94ac96-ba64-4070-b795-3b6b9b8ee371" width =500/>
    

## EC2에서 인스턴스 연결 규칙 추가 및 연결

### EC2

- 인스턴스 ID 클릭 - 보안 - 보안그룹 클릭 - 인바운드 규칙 편집
- 규칙 추가 (사용자 지정 TCP, 포트범위 8080, 소스 Anywhere IPv4) - 규칙저장
    
    <img src="https://github.com/user-attachments/assets/d4c65420-2202-41be-8be9-81dd6c1db9e1" width =500/>
    
- EC2는 껏다 키면 IP변경 되므로 껏다키지말기
    
    <img src="https://github.com/user-attachments/assets/5260b2a7-239a-45ff-bba7-7aa316af1d7b" width =500/>
    
- 인스턴스 - 인스턴스ID - 연결 - SSH클라이언트  - 제일 아래의 예 복사
    
   <img src="https://github.com/user-attachments/assets/7b6844b3-e4e3-4921-b531-6a3b91431c63" width =500/>

    

### 터미널

키페어 넣어둔 폴더로 이동 해서 복사 한 것 붙여 넣기 < 아래  1번처럼  key 실행

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
    

---

여기까지 EC2 준비 끝

## 배포 파일-resource-property 수정

> 🪄 **핵심 요약**
> 
> 
> ✅application.properties 로 들어가기
> ✅AWS - RDS - 데이터베이스ID - 엔드포인트 복사
> ✅datasourse.url=jdbc:mysql:// 뒤에 붙여넣기:13306/test
> ✅username=admin
> ✅password= RDS에서 설정한 암호 
> ✅config.domain = 버킷웹사이트 엔드포인트 , EC2 껏다키면 도메인 바뀜
> 

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
    
    //내 키 이름 +  배포파일 이름 + EC2 인스턴스연결 주소 
    
    <img src="https://github.com/user-attachments/assets/cfb2ecf7-1a91-4558-88f3-80278307cf87" width =500/>
    
    우분투에서 ls 해서 파일이 이동했는지 확인 한 다음
    
    <img src="https://github.com/user-attachments/assets/f931c2f9-87ee-4bcf-91e5-49b180b4789c" width =500/>
    
    파일 실행 명령어 입력 →  java -jar DeployServer-0.0.1-SNAPSHOT.jar
    

## 포스트맨 실행

EC2의 퍼블릭IPv4 복사 후  > 주소+ :8080 으로 요청하면 된다.

<img src="https://github.com/user-attachments/assets/0a7ad4fd-cefc-4fdc-a2fa-377da3d75407" width =500/>

1. /signin 으로 Post시 나온 token을 복사
2. /status 입력 후 key에 Authorization, value 에 token 값 붙여넣기  후 get요청
    
    <img src="https://github.com/user-attachments/assets/f95fdcaa-9c6c-40b0-8016-bb31f005c796" width =500/>
    
3. 결과
    
    <img src="https://github.com/user-attachments/assets/e4009645-6fe1-4259-8a64-c9a1310841e4" width =500/>
    
    - 무한로딩이 된다면 인바운드 규칙이 두 개가 아닌 하나 밖에 없는 것. 인바운드 규칙에 사용자지정 TCP,  포트범위 13306

## html에 EC2 주소 넣고 버킷 정책 편집

1. index.html 파일에  url 넣기.
    - EC2 - public DNS 주소 복사
    const url = “http://복사한 내용 :8080” 로 수정
        
    <img src="https://github.com/user-attachments/assets/79b61d70-a7dd-4bfb-8a37-778ba80f1979" width =500/>
        
    VSCode는 자동 저장이 안되기 때문에 저장 필수
        
    
2. 업로드
    - S3 - 내 버킷 - 업로드 - 파일 및 폴더 => DeployClient 폴내 파일 7개 드래그 후 업로드

        <img src="https://github.com/user-attachments/assets/8e759d91-a650-4b65-a8df-d120e27406e6" width =500/>
        
        <img src="https://github.com/user-attachments/assets/631210a9-6d22-47d3-9b4d-b318473d35fa" width =500/>
    
        
3. 버킷 - 권한 - 버킷 정책 - 편집 - 버킷ARN 복사 - 우측 정책 생성기 
    
    <img src="https://github.com/user-attachments/assets/46770870-341d-4dc6-a889-93b81debce55" width =500/>
    
    - ARN 복사 > 정책 생성기
        
        !<img src="https://github.com/user-attachments/assets/757bd193-14eb-4660-9e74-142ac442934b"  width =500/>
        

- 정책생성기
    
  <img src="https://github.com/user-attachments/assets/000e2db1-d977-48bc-a541-5f1d0ad2c0a0" width =500/>
    
    - Policy → Bucket Policy
    - principal → *
    - Actions → Get Object
    - ARN => 복사한거 붙여넣기 /*
        
     <img src="https://github.com/user-attachments/assets/e086edd9-7dc6-4b5a-91ea-b07e9f2af440" width =500/>
        
- Add Statement
- 생성 확인 후 Generate Policy 클릭

    <img src="https://github.com/user-attachments/assets/30becf42-3f44-4b43-885e-487d607238fc" width =500/>

- 뜨는 JSON 복사 후
    
   <img src="https://github.com/user-attachments/assets/08632d29-c787-47b1-99de-4fdffb5708f7 " width =500/>
    
- 정책에 붙여 넣기  - 변경사항 저장
    
    <img src="https://github.com/user-attachments/assets/8fc223e6-1ceb-4904-9440-b6d12eb1e01a" width =500/>
    
- 버킷 - 속성 - 제일 아래 정적 웹사이트 호스팅의 버킷 웹 사이트 엔드포인트 (http~) 클릭

## 지속적 배포 준비 : AWS 액세스 키 발급

Github Actions를 이용해 지속적 배포를 구성하기 위해선 AWS에 인증할 수단이 필요

외부 서비스에서 AWS의 서비스를 사용하기 위해 사용하는 수단이 바로 액세스 키, 비밀 액세스 키 페어

1. `IAM > 사용자`에서 본인이 로그인해서 서비스를 사용하는 IAM User를 선택
2.  IAM User 계정에서 **보안 자격 증명**
    
    <img src= "https://github.com/user-attachments/assets/cf5d8f39-a225-4f99-a2e0-c14a374f2806" width =500/>
    
3. 보안 자격 증명 탭에서 아래로 내려 **액세스 키**에서 **`액세스 키 만들기`** 버튼을 클릭
    
    <img src= "https://github.com/user-attachments/assets/b9642198-d0b1-4a62-ad6e-7d1b551182d7" width =500/>
    
4. 1단계 **액세스 키 모범 사례 및 대안**에선 지금 생성할 액세스 키를 어디서 사용할지 선택
    - Github Actions에서 사용하려면 `AWS 외부에서 실행되는 애플리케이션`을 선택
        
        <img src= "https://github.com/user-attachments/assets/a8df8c3e-86a6-4d63-a2b5-f241c4dfa16c" width =500/>
        
5. 2단계 설명 태그 설정은 선택 사항, 액세스 키에 대한 설명을 넣어주면 관리하기 용이
6. 액세스 키가 발급되면 키와 비밀 엑세스키를 확인할 수 있음, 완료 버튼 클릭 시 **다시는 확인할 수 없으니 반드시 다른 곳에 복사하거나 `.csv` 파일 다운로드로 키 페어를 파일로 보관해두기**
    
    

### Github Secret 설정

액세스 키와 비밀 액세스 키가 있으면 Github Actions에서 AWS의 서비스에 접근 가능

- 배포 코드가 올라가 있는 Github 리포지토리의 `Settings > Secrets and variables > Actions` 탭으로 이동하여 화면 오른쪽에 있는 **`New repository secret`** 버튼을 클릭
    
    <img src= "https://github.com/user-attachments/assets/baf232c1-44af-4c21-8cf3-6cba892392f9" width =500/>
    
    <img src= "https://github.com/user-attachments/assets/4d86384f-3d53-4fa8-a30f-055f4b43c783" width =500/>
    

## 지속적 배포

1. EC2 - 인스턴스 생성
2. EC2 인스턴스 연결 버튼 클릭 후 SSH 명령어 복사- 붙여넣고 - ubuntu 접속
    
    ```bash
    ssh -i "key.pem" ubuntu@<EC2-주소>
    ```
    
3. EC2에 SSH로 접속 후, 아래 명령어를 통해 aws-cli와 docker를 설치
    
    ```
    sudo snap install aws-cli --classic
    sudo snap install docker
    
    sudo systemctl enable snap.amazon-ssm-agent.amazon-ssm-agent.service
    sudo systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service
    ```
    
4. EC2 IAM 설정
    - **IAM → 역할 → 역할 생성**
    - AWS 서비스 선택
    - 사용 사례에서 **EC2** 선택 → 다음.
    - **AmazonSSMManagedInstanceCore** 검색 및 선택.
    - **AmazonSSMPatchBaselineForInstanceCore** 검색 및 선택.
    - 두 정책 모두 체크 후 다음으로 진행.
    - 역할 이름 - practice-deploy-role, 태그 추가 -  키 -> ci-cd (구분하기 쉬운 값 넣으면 된다)
5. IAM 역할 EC2에 연결
    - EC2 → 인스턴스 → 작업 → 보안 → IAM 역할 수정
    - 방금 생성한 IAM 역할(practice-deploy-role)을 선택.
    - IAM 역할 업데이트 클릭.

### Github Actions에 배포 자동화를 위한 Workflow 작성

Github Actions의 동작을 정의하는 yml 파일에 지속적 배포를 위한 작업을 추가

들여쓰기 유의하여 작성하기!!

### Main.yml

```bash
#위 내용은 지속적 통합을 위한 스크립트
#지속적 통합을 위한 스크립트 아래에 작성

# 1 번
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-2
# 2 번
- name: Start Session Manager session
  run: aws ssm start-session --target {인스턴스 id 값}

# 3 번
- name: Deploy to Server
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_REGION: ap-northeast-2
  run: |
    aws ssm send-command \
      --instance-ids {인스턴스 id 값} \
      --document-name "AWS-RunShellScript" \
      --parameters "commands=[
        'if sudo docker ps -a --format \\'{{.Names}}\\' | grep -q \\'^server$\\'; then',
        '  sudo docker stop server',
        '  sudo docker rm server',
        'fi',
        'sudo docker pull {도커 유저네임}/spring-cicd:${GITHUB_SHA::7}',
        'sudo docker tag {도커 유저네임}/spring-cicd:${GITHUB_SHA::7} spring-cicd',
        'sudo docker run -d --name server -p 8080:8080 spring-cicd'
      ]" \
      --output text

```

**🔹main.yml의 `-name`은 하나의 작업**

1. Configure AWS credentials
    - AWS의 액세스 키와 비밀 액세스 키의 정보를 가지고 AWS에 인증
    - AWS_Access_Key와 AWS_Secret_Access_Key는 공개 저장소에 업로드되면 안되는 값
    - 이 값을 통해 AWS 계정의 리소스에 접근할 수 있으므로 직접 main.yml에 작성하는 게 아니라 반드시 Github Secret에 저장하고 Github Actions가 등록된 Secret의 이름을 통해 값을 가져올 수 있도록 등록해야한다.
2. Start Session Manager session
    - AWS 세션매니저를 이용해 EC2에 연결.
    - `{인스턴스 id 값}` 자리엔 본인 리소스의 인스턴스 id 값을 입력
    - ex ) `aws ssm start-session --target i-03fcf92a9ea84738c`
3. Deploy to Server
    - 직전 단계에서 연결한 특정 EC2 인스턴스의 세션매니저를 통해 실행하고 싶은 명령어를 전송
    - 도커 허브에 업로드 된 웹 애플리케이션 이미지를 pull 받은 후 server라는 이름의 컨테이너로 실행
        - 이미 server라는 이름의 컨테이너가 실행 중이라면 종료 후 삭제한다
        - 지속적 통합 과정에서 도커 허브에 업로드 된 특정 태그 이미지를 EC2 인스턴스에 pull 받는다.
        - 특정 태그의 이미지를 바탕으로 태그가 없는 spring-cicd 이미지를 생성한다.
        - spring-cicd 이미지를 server라는 이름의 컨테이너로 8080번 포트를 통해 접근할 수 있도록 실행한다.

**🔹** 위 스크립트를 맨 마지막 단계로 추가하면 CI(지속적 통합) 작업을 성공적으로 마친 후 각 단계에 작성된 내용대로 Push 된 도커 이미지를 배포 대상 서버에 Pull하고 알맞게 컨테이너를 생성한 뒤 새로 만든 컨테이너를 실행

### 배포 자동화 테스트

배포 자동화를 위한 설정은 모두 끝

이제 배포 자동화가 지속적 배포를 통해 성공하는지 테스트 필요.

- Github Actions의 처리 결과를 통해 1차 확인을 하고
- Post-man을 통해 간단한 테스트 API가 정상 동작하는지 2차 확인