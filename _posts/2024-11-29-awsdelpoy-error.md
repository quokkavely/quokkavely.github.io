---
layout : single
title : "[aws] ci-cd 중 error : git actions"
categories: cloud
tag : [AWS, 실습, cloud]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 자동 배포 에러 정리 with Git Actions

## error 1 : git action gradle class not exception

부끄러운 실패의 기록

<img src="https://github.com/user-attachments/assets/925cabb2-6771-49a1-8a17-faee0a162366" width = 500/>

1. 문제
    
    ```bash
    --output text
    mv: cannot stat 'gradle-7.6.1/gradle/wrapper/gradle-wrapper.jar': No such file or directory
    ```
    
    해당 gradle에 대한 에러를 해결하려고 시도하고 이 만큼이나 실패했다.
    
    <img src="https://github.com/user-attachments/assets/8c4bc75a-8a38-4e75-bc2f-1f6666475c24" width=500/>
    
    <img src="https://github.com/user-attachments/assets/8059dd4a-ad5c-44e9-ad64-d0996405d6b3" width=500/>
    
    <img src="https://github.com/user-attachments/assets/aec0b529-6549-4573-8219-10ab0c916336" width=500/>
    
    <img src="https://github.com/user-attachments/assets/354b6aab-e6a0-4d1e-ac14-7f77d844a219" width=500/>
    

1. 내가 작성한 main.yml 
    
    ```bash
    name: Java CI with Gradle
    on:
      push:
        branches: [ main, dev ]
    jobs:
      build:
        runs-on: ubuntu-latest
        
        steps:
          - uses: actions/checkout@v2
          
          - name: Set up JDK 11
            uses: actions/setup-java@v2
            with:
              java-version: '11'
              distribution: 'zulu'
          
          - name: Create Gradle Wrapper
            working-directory: ./server
            run: |
              mkdir -p gradle/wrapper
              wget https://services.gradle.org/distributions/gradle-7.6.1-bin.zip
              unzip gradle-7.6.1-bin.zip
              mv gradle-7.6.1/gradle/wrapper/gradle-wrapper.jar gradle/wrapper/
              mv gradle-7.6.1/gradle/wrapper/gradle-wrapper.properties gradle/wrapper/
              chmod +x gradlew
          
          - name: Build with Gradle
            working-directory: ./server
            run: ./gradlew build -x test
    
          - name: Docker build
            run: |
              docker login -u ${{ secrets.DOCKER_HUB_USERNAME }} -p ${{ secrets.DOCKER_HUB_PASSWORD }}
              docker build -t meetbti .
              docker tag meetbti hyejipark/meetbti:${GITHUB_SHA::7}
              docker push hyejipark/meetbti:${GITHUB_SHA::7}
    
          # AWS credentials configuration
          - name: Configure AWS credentials
            uses: aws-actions/configure-aws-credentials@v1
            with:
              aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
              aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
              aws-region: ap-northeast-2
    
          - name: Deploy to Server
            env:
              AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY }}
              AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
              AWS_REGION: ap-northeast-2
            run: |
              aws ssm send-command \
                --instance-ids i-0eb0e2bbd5b50659e \
                --document-name "AWS-RunShellScript" \
                --parameters "commands=[
                  'if sudo docker ps -a --format \\'{{.Names}}\\' | grep -q \\'^server$\\'; then',
                  '  sudo docker stop server',
                  '  sudo docker rm server',
                  'fi',
                  'sudo docker pull hyejipark/meetbti:${GITHUB_SHA::7}',
                  'sudo docker tag hyejipark/meetbti:${GITHUB_SHA::7} meetbti',
                  'sudo docker run -d --name server -p 8080:8080 \\
              -e DATASOURCE_URL=${{ secrets.DATASOURCE_URL }} \\
              -e DATASOURCE_USERNAME=${{ secrets.DATASOURCE_USERNAME }} \\
              -e DATASOURCE_PASSWORD=${{ secrets.DATASOURCE_PASSWORD }} \\
              -e G_CLIENT_ID=${{ secrets.G_CLIENT_ID }} \\
              -e G_CLIENT_SECRET=${{ secrets.G_CLIENT_SECRET }} \\
              -e JWT_SECRET_KEY=${{ secrets.JWT_SECRET_KEY }} \\
              -e EMAIL_USERNAME=${{ secrets.EMAIL_USERNAME }} \\
              -e EMAIL_PASSWORD=${{ secrets.EMAIL_PASSWORD }} \\
              -e ADMIN_MAIL=${{ secrets.ADMIN_MAIL }} \\
              -e AWS_ACCESS_KEY=${{ secrets.AWS_ACCESS_KEY }} \\
              -e AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }} \\
              -e AWS_S3_BUCKET=${{ secrets.AWS_S3_BUCKET }} \\
              meetbti'
                ]" \
    ```
    
2. gradle-wrapper.jar 추가
    
    gradle-wrapper.jar 파일이 당연히 있는 줄 알았는데  알고 보니 repository에 없었다..🫠
    
    너무 당연히 있을 거라 생각했고 이 때 동안 배포 과정에서도 이런 에러를 만난 적 이 없어서 내가 환경이 좀 바뀐 것이 있어서 거기에 대한 문제인가 생각했는데 무슨… 이런 기본적인 것 때문에 시간을 사르르 녹였다…ㅎ
    
    결국에는 진짜 gradle-wrapper.jar가 없어서 발생하는 문제였다.
    
    <img src="https://github.com/user-attachments/assets/366bfe1c-2322-4203-8db2-bfe5b080f267" width=500/>
    
    추가 후에는 다른 에러 발생 !
    

## error 2 : Docker build 실패 :

1. 문제
    
    <img src="https://github.com/user-attachments/assets/94895dd6-d03e-4635-8ddf-34293129e6bd" width=500/>
    

1. 원인
    
    ```bash
    ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
    Error: Process completed with exit code 1.
    ```
    
    dockerfile이 없다고 한다.
    
    다시 보니까.. Dockerfile이라고 해야되는데 Dockerflle로 되어있었다…🫠
    
2. 오타 수정 후 해결
    
    <img src="https://github.com/user-attachments/assets/56d15775-5912-4fc7-8dd8-d04a0879e3ec" width=500/>
    

## error 3: Deploy to Server : InvalidSignatureException

오타 수정했더니 이번에는 Deploy to Server 여기서 발생

1. 문제 발생 및 원인
    
    <img src="https://github.com/user-attachments/assets/77c3d1d9-8c0e-4427-8a65-109ff01fe80e" width=500/>
    아무래도 aws key가 문제인 것 같다..
    
2. 해결
    
    여기랑 Setting에서 aws accesskey랑 secretkey 다시 복사해서 붙여넣기 했더니 해결
    
    <img src="https://github.com/user-attachments/assets/6d31dd1c-46a9-4035-93bd-5284acae66af" width=500/>
    
    <img src="https://github.com/user-attachments/assets/2e77864f-bc0a-4a97-85cc-adc92feec535" width=500/>

### 완성!!

<img src="https://github.com/user-attachments/assets/694e824e-d20f-49e9-b697-30357c8a3bba" width=500/>

---

이제 client 배포랑 https 적용 남았다..