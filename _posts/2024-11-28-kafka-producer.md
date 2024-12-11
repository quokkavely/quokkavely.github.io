---
layout : single
title : "[MSA] kafka - producer"
categories: MSA
tag : [MSA, SpringCloud, kafka]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

인프런 Dowon Lee님의 Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의를 듣고 정리한 내용입니다.😊<br>
[Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) 강의 들으러 가기👩‍🏫](https://inf.run/GHeRm) <br>
✅ 추가로 kafka 에 대해 잘 정리된 블로그가 있어 참고 하였습니다. >>
[언제나김김](https://always-kimkim.tistory.com/)<br>
{: .notice--warning}

# 카프카 프로듀서(Kafka Producer)

- 카프카 프로듀서는 데이터를 생성하여 카프카 브로커로 전송하는 역할을 한다.
- 프로듀서는 객체를 직렬화하여 바이트 배열 형태로 브로커에 전달하며, 브로커는 이를 저장하고 컨슈머는 역직렬화하여 데이터를 복원한다.

## 1️⃣ 프로듀서의 데이터 전송 방식

### 직렬화(Serialization)

- **직렬화**
    - 프로듀서는 객체를 **직렬화**하여 바이트 배열로 변환한다.
- **전송 및 저장**
    - 직렬화된 바이트 배열은 브로커의 파티션에 저장되며, 컨슈머는 이를 받아 **역직렬화**하여 사용한다.
- **직렬화 클래스 지정**
    - 프로듀서 설정 시 `key.serializer`와 `value.serializer`를 통해 직렬화 클래스를 지정해야 한다.

## 2️⃣ 메시지 키(Message Key) 사용 여부에 따른 전송

1. 메시지 키를 사용하지 않는 경우
    - **스티키 파티셔닝(Sticky Partitioning)**
        - 메시지 키를 지정하지 않으면 **스티키 파티셔닝** 방식을 사용한다
        - **특징**: 하나의 배치에 메시지를 빠르게 채워 특정 파티션으로 전송
    - **순서 보장 미흡**
        - 파티션이 여러 개일 경우 메시지의 전송 순서가 보장되지 않을 수 있다.
2. 메세지 키를 사용하는 경우
    - **해시 파티셔닝**
        - 메시지 키를 해시하여 특정 파티션에 매핑한다.
    - **순서 보장**
        - 동일한 메시지 키는 항상 같은 파티션에 저장되므로 순서가 보장됨.
    - **파티션 개수 변화 시 문제점**:
        - 파티션 개수가 변경되면 해시 결과가 달라져 메시지 순서가 깨질 수 있다.
        - **해결 방법**: 커스텀 파티셔너를 구현하거나 초기 파티션 수를 충분히 설정한다.

## 3️⃣ 프로듀서의 내부 구조와 동작 원리

### 프로듀서 데이터 전송 흐름

1. **ProducerRecord 생성**: 전송할 메시지를 생성
2. **send() 호출**: 메시지를 전송하기 위한 메서드를 호출
3. **파티셔너(Partitioner)**: 메시지를 파티션에 매핑
4. **Accumulator**: 토픽별로 배치를 생성하여 데이터를 저장한다.
5. **Sender**: 배치를 카프카 클러스터로 전송한다

### 주요 구성 요소

- **파티셔너(Partitioner)**:
    - 메시지를 특정 파티션에 할당하는 로직을 담당
    - 기본값은 `DefaultPartitioner`이며, 필요에 따라 커스텀 파티셔너를 구현할 수 있다.
- **Accumulator**:
    - 전송하기 전 메시지를 버퍼에 모아 배치를 생성
    - 효율적인 전송을 위해 배치 단위로 데이터를 관리한다.

## 4️⃣ 프로듀서 주요 옵션

### 필수 옵션

1. **bootstrap.servers**
    - 카프카 클러스터의 브로커 정보를 지정한다.
    - `'호스트이름:포트'` 형식으로 하나 이상 입력
2. **key.serializer**
    - 메시지 키를 직렬화하는 클래스
3. **value.serializer**
    - 메시지 값을 직렬화하는 클래스

### 선택 옵션

1. **acks**
    - 전송한 데이터의 확인 범위를 설정합니다.
    - **0**: 확인하지 않고 바로 다음 작업 진행.
    - **1(기본값)**: 리더 파티션에 저장되면 성공으로 판단.
    - **all(-1)**: 모든 ISR에 데이터가 저장되면 성공으로 판단.
2. **buffer.memory**
    - 배치로 전송할 데이터를 저장하기 위한 버퍼 메모리 양을 지정합니다.
    - 기본값은 **32MB**입니다.
3. **retries**
    - 전송 실패 시 재시도 횟수를 지정합니다.
    - 기본값은 **2147483647**(무제한)입니다.
4. **batch.size**
    - 배치로 전송할 레코드의 최대 크기를 지정합니다.
    - 기본값은 **16KB**입니다.
5. **linger.ms**
    - 배치를 전송하기 전까지 기다리는 최대 시간입니다.
    - 기본값은 **0ms**입니다.
6. **partitioner.class**
    - 파티셔너 클래스를 지정합니다.
7. **enable.idempotence**
    - 멱등성 프로듀서로 동작할지 여부를 설정합니다.
    - 중복 메시지 전송을 방지합니다.
8. **transactional.id**
    - 트랜잭션 프로듀서를 사용할 때 고유한 트랜잭션 ID를 지정합니다.

## 5️⃣ ISR(In-Sync Replicas)과 acks 옵션

### ISR(In-Sync Replicas)

- **정의**: 리더 파티션과 완전히 동기화된 팔로워 파티션의 집합
- **역할**: 데이터의 일관성과 가용성을 보장하며, 리더 장애 시 ISR 내 팔로워 중 하나가 리더로 승격된다.
- **모니터링**: `replica.lag.time.max.ms` 설정으로 팔로워의 동기화 상태를 확인한다.

### acks 옵션 상세 설명

1. **acks=0**
    - 리더 파티션의 응답을 기다리지 않고 전송
    - 전송 속도가 가장 빠르지만 데이터 유실 가능성이 높다.
2. **acks=1**
    - 리더 파티션에 데이터가 저장되면 성공으로 판단
    - 팔로워 파티션에 문제가 생기면 데이터 유실 가능성이 있다.
3. **acks=all(-1)**
    - ISR에 포함된 모든 파티션에 데이터가 저장되면 성공으로 판단합
    - `min.insync.replicas` 설정과 함께 사용하여 데이터의 안전성을 높인다.

### min.insync.replicas

- **역할**: ISR 내 최소 몇 개의 파티션에 데이터가 저장되어야 하는지 지정한다.
- **주의사항**
    - `min.insync.replicas`는 복제 개수보다 작아야 한다.
    - 브로커 장애 시 서비스 중단을 방지하려면 적절한 값으로 설정해야 한다.

## 6️⃣ 전송 및 재전송 메커니즘

- **Record Accumulator 가득 찬 경우**
    - `max.block.ms` 만큼 대기 후에도 전송하지 못하면 `TimeoutException`이 발생
- **Sender 스레드 동작**
    - `linger.ms` 동안 대기 후 배치를 전송
    - 브로커의 응답을 `request.timeout.ms` 만큼 기다린다.
    - 응답이 없으면 재시도하거나 `TimeoutException`이 발생한다.
- **재시도 로직**
    - `retries` 횟수만큼 재시도하며, 각 재시도 사이에 `retry.backoff.ms` 만큼 대기한다.
- **최대 전송 시간**
    - `delivery.timeout.ms` 설정으로 메시지 전송에 허용된 최대 시간을 지정한다.

---



<br>
<br>
<br>
<br>