---
layout : single
title : "[Project] 근태관리 : ip를 가져와서 출근/퇴근 구현"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

사내에 있는 컴퓨터로만 출/퇴근을 한다고 가정하고 ip가 등록된 ip일 경우에만 출근/퇴근 버튼을 누를 수 있게 구현하려고 했다.

어떻게 ip주소를 가져 오는 지에 대해서 클라이언트에서 요청할 때 추출할 수 있다고 해서 찾아보니 꽤 정리된 글들이 많았다.

그래서 ip를 어떻게 가져오는지 이런 거에 대해서는 처음 아는 것들이라 정리해보려고한다.!  

IP 기반 근태관리를 구현하기 위해 다음과 같은 흐름으로 진행하였다:

1. **IP 주소 저장**: 사내 네트워크에서 사용하는 IP 대역을 미리 시스템에 등록하거나, 개별 사용자의 허용된 IP를 저장하였다.
2. **출근 시 IP 확인**: 사용자가 출근 기록을 시도할 때, 요청을 보내는 컴퓨터의 IP 주소를 `HttpServletRequest` 객체에서 추출하였다.
3. **IP 검증**: 추출한 IP 주소가 사전에 등록된 회사 네트워크 IP 대역 또는 개인에게 할당된 IP와 일치하는지 확인하였다.
4. **출근 기록 처리**: IP 검증에 성공하면 출근 시간을 기록하고, 실패할 경우에는 오류 메시지를 반환하거나 출근 기록을 거부하였다.
5. **CRUD 구현**: Feign 클라이언트를 활용하여 근태 기록에 대한 CRUD(Create, Read, Update, Delete) 기능을 구현하였다.

## 구현 내용

### 1. IP 주소 저장

먼저, 사내 네트워크에서 사용하는 IP 대역을 시스템에 등록하였다. 이는 주로 `application.yml` 파일이나 데이터베이스에 저장하여 관리할 수 있었다. 예를 들어, 다음과 같이 허용된 IP 목록을 저장하였다:

```yaml
attendance:
  authorized-ips:
    - "192.168.0.1"
    - "192.168.0.2"
    - "192.168.0.3"

```

또는 데이터베이스 테이블을 생성하여 관리할 수도 있었다. 이를 통해 관리자는 필요한 경우 허용된 IP 목록을 쉽게 업데이트할 수 있었다.

### 2. 출근 시 IP 확인

사용자가 출근 기록을 시도할 때, 요청을 보내는 컴퓨터의 IP 주소를 `HttpServletRequest` 객체를 통해 추출하였다. 이를 위해 `AttendanceController`에서 다음과 같은 메서드를 작성하였다:

```java
@PostMapping("/check-in")
public ResponseEntity checkIn(@RequestParam Long employeeId, HttpServletRequest request) {
    Attendance attendance = attendanceService.checkIn(employeeId, ipConvertString(request));
    return ResponseEntity.ok(attendance);
}

```

### 3. IP 검증

추출한 IP 주소가 사전에 등록된 허용된 IP인지 확인하는 로직을 `AttendanceService`에 구현하였다:

```java
public boolean isVerifiedIp(String ipAddress) {
    // 사내 IP 리스트
    List<String> companyIps = List.of("192.168.0.1", "192.168.0.2", "192.168.0.3");
    return companyIps.contains(ipAddress);
}

```

### 4. 출근 기록 처리

IP 검증에 성공하면 출근 시간을 기록하는 로직을 구현하였다. 이는 주로 데이터베이스에 출근 시간을 저장하는 방식으로 이루어졌다. 예를 들어, `Attendance` 엔티티를 생성하고 이를 저장하는 방식이었다:

```java
public Attendance checkIn(Long employeeId, String ipAddress) {
    attendanceRepository.findByEmployeeIdAndAttendanceStatus(employeeId, Attendance.AttendanceStatus.CLOCKED_IN)
        .ifPresent(a -> { throw new BusinessLogicException(ExceptionCode.CAN_NOT_WORK_IN); });

    if (!isVerifiedIp(ipAddress)) {
        throw new BusinessLogicException(ExceptionCode.NOT_ALLOWED_IP);
    }

    verifiedEmployee(employeeId);

    Attendance attendance = new Attendance();
    attendance.setEmployeeId(employeeId);
    attendance.setCheckInTime(LocalDateTime.now());
    attendance.setAttendanceStatus(Attendance.AttendanceStatus.CLOCKED_IN);

    return attendanceRepository.save(attendance);
}

```

## IP 주소 가져오는 방법

IP 주소는 인터넷 프로토콜(IP)을 사용하는 네트워크에서 각 장치가 통신을 위해 부여받는 고유한 주소이다. IP 주소는 주로 IPv4와 IPv6 두 가지 버전이 존재하며, 네트워크 내에서 데이터가 올바른 대상에 도달할 수 있도록 경로를 지정하는 역할을 한다.

### IP 주소의 종류

1. **공인 IP 주소**: 인터넷 상에서 유일하게 식별 가능한 IP 주소로, 외부 네트워크와의 통신에 사용된다.
2. **사설 IP 주소**: 내부 네트워크에서 사용되는 IP 주소로, 외부에서는 직접 접근할 수 없다. 일반적으로 NAT(Network Address Translation)를 통해 공인 IP 주소와 매핑된다.
    - 사설 IP 대역은 다음과 같다:
        - **10.0.0.0 ~ 10.255.255.255 (10.0.0.0/8)**
        - **172.16.0.0 ~ 172.31.255.255 (172.16.0.0/12)**
        - **192.168.0.0 ~ 192.168.255.255 (192.168.0.0/16)**

### IP 주소 검증

- 웹 애플리케이션에서 클라이언트의 IP 주소를 추출하는 방법은 주로 `HttpServletRequest` 객체를 통해 이루어진다.
- 그러나 클라이언트가 프록시 서버나 로드 밸런서를 통과하는 경우, 실제 클라이언트의 IP 주소가 `X-Forwarded-For` 헤더에 포함될 수 있다.
- 따라서 IP 주소를 정확히 추출하기 위해서는 이러한 헤더를 고려해야 한다.
    
    ```java
    private String ipConvertString(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
    
        if (ip != null && !ip.isEmpty() && !"unknown".equalsIgnoreCase(ip)) {
            if (ip.contains(",")) {
                ip = ip.split(",")[0].trim();
            }
        } else {
            ip = request.getHeader("Proxy-Client-IP");
            if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("WL-Proxy-Client-IP");
            }
            if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_CLIENT_IP");
            }
            if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_X_FORWARDED_FOR");
            }
            if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getRemoteAddr();
            }
        }
    
        if (isInternalIp(ip)) {
            return String.format("내부 IP: %s", ip);
        }
        return ip;
    }
    ```
    
1. **`X-Forwarded-For` 헤더 확인**
    - 클라이언트가 프록시 서버를 통해 요청을 보냈을 때, 원래의 IP 주소는 `X-Forwarded-For` 헤더에 포함된다.
    - 여러 개의 IP 주소가 콤마(,)로 구분되어 있을 수 있는데, 이 경우 첫 번째 IP 주소가 실제 클라이언트의 IP이다.
    - 따라서, `X-Forwarded-For` 헤더가 존재하고, 값이 비어 있지 않으며, `"unknown"`이 아닌 경우 이를 사용한다.
2. **기타 헤더 확인:**
    - `X-Forwarded-For` 헤더가 없거나 유효하지 않은 경우, 다른 헤더들을 순차적으로 확인한다.
    - `Proxy-Client-IP`, `WL-Proxy-Client-IP`, `HTTP_CLIENT_IP`, `HTTP_X_FORWARDED_FOR` 헤더들을 순서대로 확인하여 IP 주소를 추출한다.
    - 모든 헤더가 유효하지 않을 경우, `request.getRemoteAddr()`를 사용하여 IP 주소를 가져온다.
3. **내부 IP 주소 판별:**
    - 추출한 IP 주소가 내부 네트워크의 IP 주소인지 확인하기 위해 `isInternalIp` 메서드를 호출한다.
    - 내부 IP 주소란 주로 사내 네트워크에서 사용하는 IP 대역을 의미하며, 공인 IP 주소와는 구분된다.
    - 내부 IP 주소일 경우, `"내부 IP: "`라는 접두사를 붙여 반환한다. 이는 로깅이나 디버깅 시 유용할 수 있으나, 실제 시스템에서는 필요한 경우에만 사용하도록 주의해야 한다.]

```java
private boolean isInternalIp(String ipAddress) {
    if ("0:0:0:0:0:0:0:1".equals(ipAddress) || "127.0.0.1".equals(ipAddress)) {
        return false; //일단 허용 (test하기위해,,)
    }

    if (ipAddress.startsWith("10.") ||
            ipAddress.startsWith("192.168.") ||
            (ipAddress.startsWith("172.") && Integer.parseInt(ipAddress.split("\\.")[1]) >= 16
                    && Integer.parseInt(ipAddress.split("\\.")[1]) <= 31)) {
        return true;
    }

    return false;
}
```

- **로컬호스트 확인:**
    - `"0:0:0:0:0:0:0:1"` 또는 `"127.0.0.1"`과 같은 로컬호스트 IP 주소는 외부 네트워크에서 접근할 수 없으므로, 이를 `false`로 반환한다.
- **사설 IP 대역 확인:**
    - 사설 IP 대역인 `10.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12` 범위 내에 있는지 확인한다.
    - `ipAddress.startsWith("10.")` 또는 `ipAddress.startsWith("192.168.")`와 같이 각 사설 IP 대역의 시작 부분을 확인한다.
    - `172.`로 시작할 경우, 두 번째 옥텟이 `16` 이상 `31` 이하인지 추가로 확인하여 사설 IP 범위에 속하는지 판별한다.

---

오랜만에 ip에 대해서 공부했는데 새로 보는 것 같은 느낌이다...ㅎㅎㅎ

실제로는 어떻게 구현하는지 궁금한데 ip로 안하면 일 안하고 출근을 누를 수 있기도 하고 

요새는 좀 깐깐하게 볼 수 도 있을 것 같아 회사 컴퓨터로만 근태신청을 할 수 있게 해두었다.

다른 회사들도보면 컴퓨터 켜서 그 컴퓨터로만 연장근로나 이런 것들을 신청할 수 있는 회사도 본 적 이 있어서 생각 나서 그렇게 했는데 보안이나 이런 것은 더 찾아봐야할 것 같다.

---

- [GitHub 커밋: 근태관리 구현](https://github.com/pingpong-works/core-api/commit/cdf2246c3a4426264502aa7702131e0a9baf8830)
- [https://velog.io/@haron/Spring-최초-요청-IP-는-어떻게-가져오는-걸까-6aie0x11#-httpservletrequest의-getremoteaddr-이-클라이언트-ip를-전달하도록-스프링-설정을-하는-방법](https://velog.io/@haron/Spring-%EC%B5%9C%EC%B4%88-%EC%9A%94%EC%B2%AD-IP-%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EA%B0%80%EC%A0%B8%EC%98%A4%EB%8A%94-%EA%B1%B8%EA%B9%8C-6aie0x11#-httpservletrequest%EC%9D%98-getremoteaddr-%EC%9D%B4-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-ip%EB%A5%BC-%EC%A0%84%EB%8B%AC%ED%95%98%EB%8F%84%EB%A1%9D-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%84%A4%EC%A0%95%EC%9D%84-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
- [https://velog.io/@mardi2020/Servelet으로-외부-IP-주소-얻기](https://velog.io/@mardi2020/Servelet%EC%9C%BC%EB%A1%9C-%EC%99%B8%EB%B6%80-IP-%EC%A3%BC%EC%86%8C-%EC%96%BB%EA%B8%B0)