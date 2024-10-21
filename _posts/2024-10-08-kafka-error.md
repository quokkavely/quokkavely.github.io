---
layout : single
title : "[Project] 알림구현 : kafka+SSE 사용 중 에러"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


전자결재 거의 다 구성하고 나서 알림 기능을 붙여야하는데 kafka를 사용하는 건 처음이라 역시 초기 셋팅이 힘들었다.<br> 
또 MSA 구조이다보니 어떻게 producer랑 consumer 생성하지에 대한 고민도 있었다. <br>
어찌저찌 구현을 다하고 나서 전자결재 문서를 작성하고 올릴 시에 <br> 승인자에 해당하는 사람들에게 알림을 전송하는 기능을 구현하기 위해서

알림에서 consumer 구현 한 다음 전자결재에서 producer를 구현했는데
무한으로 연결에러가 떠서 같이 정신이 없어지는 기분이었다.

## 1. 연결 실패 문제

먼저, **전자결재(Producer)**에서 알림(Consumer)으로 메시지를 보내기 위한 설정을 진행했다. 하지만 처음 시도하자마자 무한 연결 실패 메시지가 뜨기 시작했다. 이거 구성하고 나서도 중간에 docker가 멈춰서 오잉...? 하고 다시 실행했는데 또 중단 되어있었다.

<img src= "https://github.com/user-attachments/assets/7b0b3d3a-2bca-4780-b845-34247d66c3b6" width=500/>

원인을 찾기 위해 로그를 확인해 보니 환경 변수 설정에서 문제가 있었다.

<img src= "https://github.com/user-attachments/assets/de3fed04-e27a-400e-92fd-6df70f8afe71" width=500/>

kafka-compose.yml 파일로 만들어서 docker 실행했는데
**docker-compose** 라는 이름이 아니라서 ? 발생한 문제였던 것 같다.

<img src= "https://github.com/user-attachments/assets/b4cc42ea-eeb3-48c9-b512-f0b971eb90cc" width=500/>

docker-compose.yml로 수정하고 다시 command창에 up 하니까 해결 완료

## 2. Import 실수

### StringSerializer import문  확인
org.apache.kafka.common.serialization 는 카프카에서 메시지를 직렬화할 때 사용되고
com.fasterxml.jackson.databind.ser.std.serialization 은 Java 객체의 문자열 데이터를 JSON으로 직렬화하는 역할을 하는 것이기 때문에 <br>직렬화에 대해 에러가 발생햇음을 확인하고 import 문을 수정하였다.

<img src="https://github.com/user-attachments/assets/584fc39b-8e62-46a6-b2c8-39d14adf79f1" width =500/>

1. 변경 전

    <img src= "https://github.com/user-attachments/assets/f685cf81-983b-4f33-8c93-dd0b33c0df8f" width=500/>

2. 변경 후

    <img src= "https://github.com/user-attachments/assets/dae74d5e-33ac-40af-a2c7-287033870d9b" width=500/>


### value 도 잘못된 import 문
설정을 수정한 후 다시 시도했는데, 또 다른 에러가 발생했다. 이번엔 VALUE의 IMPORT 관련 설정이 잘못되어 있었다. 

<img src ="https://github.com/user-attachments/assets/27d2cec1-1a12-4fc8-a2fc-28555c5195f5" width=500/>

다행히, 수정하고 나니 연결이 성공했고 submit은 성공을 했다, 그러나 결재자한테 알림이 가는지 확인해야하는데...

<img src="https://github.com/user-attachments/assets/96b04472-fa34-4ad6-b347-d3d9663f9134" width=500>

## 3. 패키지 문제 및 신뢰할 수 없는 패키지 에러

- 패키지 에러에서 또 실패!@

    <img src="https://github.com/user-attachments/assets/d12f27fc-3f51-45f3-ba45-73c5c0028c4f" width=500/>

- output이 엄청나다 ㅠㅠㅠ...신뢰패키지 설정 다했는데 왜 신뢰하지 못한다는 걸까.. 하고 계속 시도 했는데 안되서 여기서 좀 많이 헤맸다.
    <img src="https://github.com/user-attachments/assets/c4aafcf9-b61b-4cae-8fd7-9f7c2a6ce00a" width=500/>
    <img src="https://github.com/user-attachments/assets/28ea9fca-5b0f-4419-922b-304f53bf668f" width=500/>
    <img src="https://github.com/user-attachments/assets/fa3c49f4-9a10-4129-879c-7b6b9edd0992" width=500/>

- 와일드카드로 addTrustedPackage("*")만 적은 후 시도를 해보았는데 해결되지 않았다.

- Kafka에서 패키지 구조가 다를 때 addTrustedPackage를 설정했음에도 에러가 발생하는 경우는 보통 직렬화/역직렬화 문제와 관련이 있을 가능성이 크다고 한다.
- 특히, Kafka는 메시지를 전송할 때 직렬화된 데이터를 사용하며, Java의 기본 직렬화 방식을 사용하는 경우 패키지 정보가 직렬화된 클래스의 일부로 포함된다. 
- 따라서, consumer와 producer가 다른 패키지에서 동일한 클래스를 사용할 때 Kafka는 패키지가 다르면 해당 클래스가 동일한 클래스가 아니라고 판단할 수 있다



알림을 보내는 부분에서는 여전히 문제가 있었다. **OUTPUT 에러**가 계속해서 나타나면서 로그에 에러 메시지가 출력되었다. 이 문제는 패키지 이름이 일치하지 않아서 발생한 것으로, 패키지 경로를 제대로 설정하지 않았기 때문이었다.
그래서 alarm과 core의 패키지명을 같게 하기 위해서 com.core.kafka에 있던 kafka 관련 클래스 를 밖으로 빼서 com.alarm.kafka 패키지를 생성 후 넣어주었다.


패키지 경로를 바꾸고 나니 문제가 해결되었지만, 또 다른 에러가 발생했다.

## 4. 기본 생성자 문제

알림 메시지를 전송하는 과정에서 **기본 생성자가 없다는 에러**가 떴다.

<img src= "https://github.com/user-attachments/assets/380c5d2c-1725-4e3c-99dc-2473600f135c" width=500/>

이 문제는 간단히 해결할 수 있었다. **NotificationMessage** 클래스에 기본 생성자를 추가함으로써 문제가 해결되었다.

## 5. 문제 해결 후 성공

드디어, 기본 생성자를 추가하고 알림 메시지가 정상적으로 전송되는 것을 확인할 수 있었다. 오랜 시간 동안 씨름한 끝에 알림 시스템이 제대로 동작하기 시작했다.

<img src= "https://github.com/user-attachments/assets/09a2cc90-91c5-4fb2-a47f-3efa9df44bf1" width=500/>

---

### 결론

이번 작업을 통해 알게 된 것은 **패키지 경로의 일관성**과 **필수 생성자 설정** 등 기본적인 부분에서도 사소한 실수가 큰 문제를 일으킬 수 있다는 점이었다. 또한, 환경 변수와 설정 파일에 대한 꼼꼼한 점검이 얼마나 중요한지 다시 한번 느낄 수 있었다.