---
title : "[aws] https 적용기 2탄 : EC2와 로드밸런서를 활용한 HTTPS 적용 및 클라우드프론트 통합"
categories: cloud
tag : [AWS, 실습, cloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# AWS EC2와 로드밸런서를 활용한 HTTPS 적용 및 클라우드프론트 통합

앞서 HTTPS를 S3와 CloudFront를 통해 적용했다면, 이번에는 서버(EC2)와 클라이언트 간 HTTPS 통신을 설정하고, 이를 최적화하기 위한 로드밸런서 및 CloudFront 설정을 진행해야 한다. 

또한, CloudFront 배포 전략을 통해 최적화된 웹 애플리케이션 환경을 구축할 예정.

## 1. EC2 대상 그룹 생성

### 1.1 대상 그룹 생성

1. **EC2 > 대상 그룹 > 대상 그룹 생성**으로 이동합니다.
2. **대상 유형**과 **대상 그룹 이름**을 선택합니다.
    
    <img src="https://github.com/user-attachments/assets/ecdb2341-331e-47b3-8d8c-805e370383b5" width =700/>
    
    <img src="https://github.com/user-attachments/assets/38282edf-5278-4ddd-96ec-a7db7103c6a4" width =700/>
    

### 1.2 Health Check 설정

- 서버의 `/health` 경로를 상태 검사의 경로로 지정한다.
    
    <img src="https://github.com/user-attachments/assets/ebd6693b-3bb6-441d-bc99-56e41f4b0794" width =700/>
    
    - 서버 controller에 method가 없으면 추가해주어야 함.
        
        ```java
        package com.springboot.helper;
        
        import org.springframework.http.ResponseEntity;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        @RestController
        public class HealthCheckController {
            @GetMapping("/health")
            public ResponseEntity<String> healthCheck() {
                return ResponseEntity.ok("OK");
            }
        }
        ```
        
- 초기에는 Health Check 간격을 짧게 설정(예: 5초)하여 빠르게 상태를 확인한 후, 정상 동작이 확인되면 300초로 변경하여 비용을 절감할 수 있다.

### 1.3 대상 등록

- EC2 인스턴스를 대상 그룹에 등록합니다.
    
    <img src="https://github.com/user-attachments/assets/b0cfbc20-243f-482a-92c6-eaf9367bb8d0" width =700/>
    

## 2. 로드밸런서 생성 및 설정

1. **EC2 > 로드밸런서 > 로드밸런서 생성**으로 이동
    
    <img src="https://github.com/user-attachments/assets/9ebdd906-a903-4aa7-9d05-6219c3680101" width =700/>
    
2. **로드밸런서 이름**을 설정
3. 가용 영역(4개)을 모두 선택
    
    <img src="https://github.com/user-attachments/assets/e2589e98-197d-4fc1-8e21-a1dcab64f381" width =700/>
    
4. **EC2와 동일한 보안 그룹(launch-wizard-4)**을 설정하여 네트워크 연결을 통합 관리한다. 
5. 리스너 및 라우팅
    - HTTP와 HTTPS를 리스너로 설정
    - HTTPS 리스너에 **서울 리전에서 발급받은 인증서를 등록**  (즉, 버지니아북부와 서울(= EC2와 같은 지역) 2개의 인증서를 발급 받아야 함)

## 3. CloudFront와 로드밸런서 연동

### 3.1 CloudFront에서 새로운 배포 생성

- **원본 설정**: 오리진을 S3가 아닌 로드밸런서로 설정
    
    <img src="https://github.com/user-attachments/assets/f05908c4-92d2-49c3-9160-809aca336ef8" width =700/>
    
- **캐시 설정**
    
    <img src="https://github.com/user-attachments/assets/49895a84-0738-499c-8d2a-f7fe72742467" width =700/>
    
    - 로드밸런서는 서버 요청에 대한 것이기 때문에 허용 HTTP 메서드를 서버에 맞게 지정해준다.
    - API 요청(`/api/*`): 캐시 비활성화(CachingDisabled).
    - 정적 파일 요청(`/*`): 기본 캐싱 설정.
- 방화벽 및 설정 구성
    
    <img src="https://github.com/user-attachments/assets/6dbac7e4-8f69-4134-8096-19b043bc9c0e" width =700/>
    
    - 보안 보호 비활성화하여 비용을 절감하고
    - 대체도메인에 서버에 대한 도메인을 지정해준다.
    - 버지니아 북부에 등록된 인증서 선택하기
    

## 4. Route53 레코드 생성

<img src="https://github.com/user-attachments/assets/8bedd757-afd0-401e-ae38-633a1d0a95ea" width =700/>

- **Route53 > 호스팅 영역 > 레코드 생성**으로 이동
- 별칭을 활성화하고, CloudFront 배포를 대상으로 설정

## 5. CloudFront 배포 전략: 단일 vs 이중

### 5.1 CloudFront 단일 배포

- 정적 파일과 API 요청을 **하나의 배포**에서 처리
- `/api/*` 요청은 로드밸런서로, 그 외는 S3로 라우팅
1. **장점**
    - 관리가 간단하고 비용 절감 가능
    - 단일 도메인으로 통합(`example.com`)
2. **단점**
    - 경로 설정의 복잡성
    - 특정 원본에 트래픽 집중 시 성능 저하 가능

### 5.2 CloudFront 이중 배포

- 첫 번째 배포: S3를 원본으로 설정(정적 파일)
- 두 번째 배포: 로드밸런서를 원본으로 설정(API 요청)
1. **장점**
    - 역할 분리가 명확하여 디버깅이 쉬움
    - 각 배포의 정책 및 확장성 관리 가능
2. **단점**
    - 설정 및 유지보수 복잡
    - 비용 증가

---

https 적용하는데 제대로 된 매뉴얼이 없어서 여기저기 참고하다 보니 뒤죽박죽 섞여서 처음에 고생을 많이 했다. 그래서 다음에는 헷갈리지 않으려고 과정을 저장해두었고, 한번도 배포는 해보지 않았는데 배포부터 적용까지 하니 뭔가 뿌듯!@ 이제 부가적으로 해야할 것 정리해서 개선해봐야지