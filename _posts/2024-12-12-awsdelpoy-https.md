---
title : "[aws] https 적용기 1탄 : AWS를 활용한 도메인 및 인증서 설정 (with 가비아)"
categories: cloud
tag : [AWS, 실습, cloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# HTTPS 적용기: AWS를 활용한 도메인 및 인증서 설정

AWS를 사용하여 S3 버킷의 정적 웹사이트에 **HTTPS**를 적용하는 과정을 단계별로 정리했다.

이를 통해 보안 인증서를 설정하고, 구매한 도메인과 CloudFront를 연동하여 HTTPS 기반의 안정적인 웹사이트를 구축할 수 있다.

## 1. 도메인 구매

**HTTPS 적용을 위해서는 SSL 인증서가 필요하다.** S3 버킷의 웹사이트 엔드포인트는 AWS 소유이기 때문에 인증서를 직접 발급받을 권한이 없다. 따라서 도메인을 구매하여 HTTPS 설정을 진행해야 함.

- 나는 **가비아**에서 도메인을 구매했고  닷컴은 비싸서 .site로 할인가에 구매

## 2. Route53 호스팅 영역 생성

### 2.1. 호스팅 영역 생성

- AWS **Route53 > 호스팅 영역 > 호스팅 영역 생성**에서 구매한 도메인으로 호스팅 영역을 생성
- 도메인 이름과 호스팅 영역 유형 선택
    
    <img src="https://github.com/user-attachments/assets/02380b72-5db7-4ceb-a8e1-abbcfcd79297" width =500/>
    
- 선택 후 아래 호스팅 영역 생성 버튼 눌러주기

### 2.2. 네임서버 설정

- 생성된 호스팅 영역에는 기본적으로 **NS 레코드** 유형의 네임서버 4개가 발급된다.
    
    <img src="https://github.com/user-attachments/assets/d51b564e-90e0-4b97-bf24-836e64b2b748" width =500/>
    
- 구매한 도메인의 네임서버를 **가비아 사이트에서 Route53 네임서버로 변경**
    - 네임서버 1,2,3,4 를 순서대로 1차 ~ 4차에 복사해서 붙여 넣어주면 된다. (단, 뒤에 . 은 빼야 함)
    
    <img src="https://github.com/user-attachments/assets/6b781d1e-52e5-450b-be89-a8643857e352" width =500/>
    

## 3. 인증서 발급

**AWS Certificate Manager**를 사용하여 SSL 인증서를 요청한다.

**주의**: 리전은 반드시 **버지니아 북부(N. Virginia)로 설정**해야 CloudFront에서 사용할 수 있다.

### 3.1. 인증서 요청

1. AWS **Certificate Manager > 인증서 > 인증서 요청**으로 이동
2. **퍼블릭 인증서 요청**을 선택
    
    <img src="https://github.com/user-attachments/assets/a9674e0c-a8fa-4510-9c28-43484ad4db43" width =500/>
    
3. **도메인 이름 입력**
    - 기본 도메인(e.g., `example.com`)과 함께 서브도메인(e.g., `www.example.com`)도 추가한다.
        
      <img src="https://github.com/user-attachments/assets/482660c7-ec2b-47fd-b265-f88ef8812420" width =500/>
        
    - 나는 아래와 같이 3개의 도메인을 추가해주었다.
        - (1) **가비아에서 구매한 도메인**과
        - (2) **[www.도메인]**으로 요청해도 사이트가 나오게 하고 싶을 경우 www도 추가로 적어준다.
        - (3) 그다음 **서버에서 요청할 때 받을 주소**를 지정해준다.
    - **요청 버튼 클릭**
        
        도메인이름, 검증방법, 키 알고리즘을 선택 후 태그 아래에 요청 버튼을 클릭한다
        
    
4. **Route53 레코드 생성**
    
    인증서 요청으로 생성된 도메인들을 Route53에 레코드 생성 버튼을 통해 Route53에 추가하고 조금 기다리면 발급이 완료된다.
    
    <img src="https://github.com/user-attachments/assets/77c6090b-fd31-44fa-8249-7ad2bb57efa3" width =500/>
    
    - 요청한 인증서에 대해 Route53에서 자동으로 레코드를 생성한다.
    - Route S3에서 레코드 생성 클릭 > 바코드생성

## 4. CloudFront 생성

CloudFront는 S3 버킷과 도메인을 연결하고 HTTPS를 설정하는 역할을 한다.

### 4.1. CloudFront 배포 생성

1. **Origin Domain**에 S3 버킷을 선택
2. **Viewer Protocol Policy**를 `Redirect HTTP to HTTPS`로 설정
3. 대체 도메인(CNAME)에 구매한 도메인을 입력
4. **인증서 선택**에서 발급받은 인증서를 연결
    
    <img src="https://github.com/user-attachments/assets/2a02eb34-161f-4477-96fa-02c79a6a8855" width =500/>
    

## 5. Route53과 CloudFront 연결

### 5.1. Route53에서 레코드 생성

1. Route53으로 돌아와 **레코드 생성** 버튼을 클릭합니다.
2. **별칭**을 활성화하고, 트래픽 라우팅 대상에서 CloudFront 배포를 선택합니다.
    - **클라이언트**: 브라우저에 도메인을 입력.
    - **Route53**: 도메인을 CloudFront 배포로 라우팅.
    - **CloudFront**: HTTPS 인증서를 사용해 S3 콘텐츠를 안전하게 제공.
    
    <img src= "https://github.com/user-attachments/assets/e6fdd5f4-9690-4933-90ba-7da5e7f29ce2" width =500/>
    
3. 이제 도메인과 HTTPS 설정이 완료! 지만 아직 남았지 ⇒ **서버와 HTTPS 통신!**
    
    S3 버킷에 HTTPS를 적용했더라도, 서버와의 통신이 HTTP라면 보안이 완전하지 않다.
    
    다음 단계로는 **AWS EC2에서 로드밸런서를 통해 HTTPS를 설정**하여 서버와의 통신도 HTTPS로 보호할 계획이다.