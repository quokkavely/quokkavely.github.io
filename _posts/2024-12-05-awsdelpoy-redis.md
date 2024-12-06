---
layout : single
title : "[aws] 배포 후 redis 연결 에러 발생 및 해결"
categories: cloud
tag : [AWS, 실습, cloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# AWS EC2에서 Redis 연결 문제 해결

AWS EC2 인스턴스에 애플리케이션을 배포한 후, 회원가입과 로그인이 되지 않는 문제가 발생했다. 서버 로그를 확인해보니 Redis에 연결할 수 없다는 오류가 나타났고 Redis 연결 문제의 원인과 이를 해결한 과정을 기록하였다.

## 문제 상황

애플리케이션을 배포한 후, 클라이언트에서 회원가입과 로그인을 시도하면 실패하는 현상이 발생했고 서버에는 다음과 같은 로그가 기록되었다.

<img src= "https://github.com/user-attachments/assets/eec1f8fe-f297-4cf4-b555-98273a47086d" width = 500/>

애플리케이션이 Redis에 연결하려고 하지만 연결이 거부되고 있었다.

## 원인 분석

application.yml에는 host에 localhost로 적혀있었기에 `localhost`(127.0.0.1)에서만 수신 대기하도록 설정되어 있어 이로 인해 EC2 인스턴스의 프라이빗 IP 주소로는 연결이 거부되고 있었고 그래서 애플리케이션이 Redis에 접근할 수 없었던 것이다.

```yaml
spring:
  redis:
    host: localhost
    port: 6379

```

## 해결 방법
### 1. Redis 설정 파일 수정

Redis가 EC2 인스턴스의 프라이빗 IP 주소에서도 수신 대기하도록 설정을 변경해야 한다.

터미널이나 SSH 클라이언트를 사용하여 EC2 인스턴스에 접속

```bash
ssh -i /path/to/your/key.pem ubuntu@your-ec2-public-ip
```

redis 설정 파일 열기

```bash
sudo vi /etc/redis/redis.conf
```

`redis.conf` 파일에서 `bind` 설정을 찾고

```bash
bind 127.0.0.1 ::1
```

이 줄을 다음과 같이 변경합니다:

```bash
bind 127.0.0.1 {EC2 프라이빗 IP주소}
```

이렇게 하면 Redis가 `localhost`와 프라이빗 IP 주소 모두에서 연결을 수락하게 된다.

### 2. Redis 재시작

설정 변경 사항을 적용하기 위해 Redis 서버를 재시작한다.

```bash
sudo systemctl restart redis
```

또는 다음과 같이 재시작할 수 있다.

```bash
sudo service redis-server restart
```

### 3. Redis 수신 대기 확인

Redis가 프라이빗 IP 주소에서 수신 대기 중인지 확인

```bash
sudo netstat -tlnp | grep 6379
```

출력 결과에 `{EC2 프라이빗 IP주소}:6379`가 표시되면 설정이 올바르게 적용된 것

### 4. Redis 연결 테스트

EC2 인스턴스에서 Redis에 연결할 수 있는지 테스트한다.

```bash
redis-cli -h {EC2 프라이빗 IP주소} -p 6379 ping
```

`PONG` 응답이 오면 연결이 성공

### 5. 애플리케이션 설정 수정

`application.yml` 파일에서 Redis 호스트를 EC2 인스턴스의 프라이빗 IP 주소로 설정. 
보통 환경 변수를 사용하여 설정하므로, `EC2_IP`라는 환경 변수에 프라이빗 IP 주소를 지정

```yaml
spring:
  redis:
    host: ${EC2_IP}
    port: 6379
```

deploy.yml에도 EC2_IP 환경 변수 설정하고 

```bash
 -e EC2_IP=${{ secrets.EC2_IP }} \\
```

github repository - Settings - security - Secrets and variables - Actions - Repository secrets 에 EC2 프라이빗 주소를 저장한다.

### 6. 애플리케이션 재시작 및 테스트

애플리케이션을 재시작하여 변경된 설정이 적용 회원가입과 로그인이 정상적으로 작동하는지 확인

## 추가 사항

Redis를 여러 인터페이스에서 수신 대기하도록 설정하면 보안 위험이 증가할 수 있어서 아래와 같은 보안 사항을 추가하면 좋다고 한다.

- **방화벽 및 보안 그룹 설정**: AWS 보안 그룹에서 Redis 포트(6379)에 대한 인바운드 트래픽을 제한한다.. 필요한 경우에만 특정 IP나 서브넷에서 접근할 수 있도록 설정하기
- **Redis 인증 설정**: Redis에 비밀번호를 설정하여 인증을 요구하도록 하기. `redis.conf` 파일에서 다음과 같이 설정하면 된다.
    
    ```bash
    requirepass your_redis_password
    ```
    
    애플리케이션에서도 Redis 클라이언트 설정에 비밀번호를 추가해야 한다.
    

## 결론

Redis 연결 문제는 Redis가 `localhost`에서만 수신 대기하도록 설정되어 발생한 것이었고 Redis 설정을 수정하여 프라이빗 IP 주소에서도 수신 대기하도록 변경하고, 애플리케이션 설정에서 Redis 호스트를 프라이빗 IP 주소로 지정함으로써 문제를 해결할 수 있었다.

배포하면서 워낙 이슈가 많아서 이 정도는 에러는 감사하다..😂