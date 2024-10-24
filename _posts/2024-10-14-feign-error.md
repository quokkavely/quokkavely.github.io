---
layout : single
title : "[Project] feign 통신 구현 : 소통 필수@!"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## Fegin으로 통신구현

### dependency 추가

```yaml
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
```

### @EnableFeignClients 추가

Feign 클라이언트를 사용하여 다른 마이크로서비스와 통신해야 한다면, 두 서비스 모두에 `@EnableFeignClients` 어노테이션을 추가해야 한다.

나는 core-api 랑 auth-api랑 통신해야하므로@EnableFeignClients 애너테이션을 붙여주었다

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class CoreApiApplication {
  public static void main(String[] args) {
    SpringApplication.run(CoreApiApplication.class, args);
  }
}
```

### AuthServiceClient

```java
@FeignClient(name = "auth-api")
public interface AuthServiceClient {
    @GetMapping("/employees/{id}")
    EmployeeDto getEmployeeById(@PathVariable("id") Long employeeId);
}
```

### dto 추가 → service의 반환 객체와 필드가 동일해야함

```java
@Getter
public class EmployeeDto {
        private long id;
        private String name;
        private String email;
        private String phoneNumber;
        private String profilePicture;
        private String createdAt;
        private String departmentName;
}
```

모든 정보가 필요 하지는 않아서 필드 일부만 가져왔고 필드 명만 동일하게 설정했다.

---

## 문제 발생 : employeen 모두 null 값으로 들어옴

근데 잘 되다가 갑자기 employee가 null 로 들어오는 문제가 발생..

auth-api pull 받다 보니까 메서드가 변경된 걸 확인할 수 있었음. 

그래서 기존에 feign으로 employee가져오려는 부분도 문제가 생긴 것이다.

### 시도 1 : 메서드명 변경

```java
//기존
@FeignClient(name = "auth-api", url = "http://localhost:50000")
public interface AuthServiceClient {

    @GetMapping("/employees/{id}")
    EmployeeDto getEmployeeById(@PathVariable("id") Long employeeId);
}

//변경
@FeignClient(name = "auth-api", url = "http://localhost:50000")
public interface AuthServiceClient {

    @GetMapping("/user/employees/{id}")
    UserResponse getEmployeeByIdForUser(@PathVariable("id") Long id);
}
```

재빠르게 수정 한 다음에 employee를 가져오려고 하는데 모든 필드가 null로 들어왔다,,,

<img src= "https://github.com/user-attachments/assets/11f13dbd-1e99-47c3-b939-67f535dbfeef" width=500/>

그래서 postman으로도 한번 테스트 해보았는데

 [http://localhost:50000/user/employees/1](http://localhost:50000/user/employees/1)에 get 요청했을때는 문제없이 아래의 값이 잘 나왔다

```json
{
    "data": {
        "employeeId": 1,
        "name": "제리",
        "email": "abcd@gmail.com",
        "phoneNumber": "010-0020-1000",
        "employeeRank": "INTERN",
        "extensionNumber": "010-0000-0000",
        "emergencyNumber": null,
        "departmentName": "인사팀"
    }
}
```

### 시도 2 : 반환 객체 구조 동일하게 변경

**Auth-api 구조**

1. Controller
    
    ```java
     // 특정 회원 조회 - 직원용
        @GetMapping("/user/employees/{id}")
        public ResponseEntity<SingleResponseDto<EmployeeDto.UserResponse>> getEmployeeByIdForUser(@PathVariable Long id, Authentication authentication) {
    
            // Service에서 인증 및 권한 검증 수행
            Employee employee = employeeService.findEmployeeById(id, authentication);
            EmployeeDto.UserResponse response = employeeMapper.employeeToUserResponseDto(employee);
    
            return ResponseEntity.ok(new SingleResponseDto<>(response));
        }
    ```
    
2. Dto
    
    ```java
      //기존
        @Getter
        @Setter
        @NoArgsConstructor
        public static class Response {
            private long employeeId;
            private String name;
            private String email;
            private String phoneNumber;
            private String profilePicture;
            private String extensionNumber;
            private String emergencyNumber;
            private String address;
            private String vehicleNumber;
            private String createdAt;
            private Employee.EmployeeStatus status;
            private Employee.AttendanceStatus attendanceStatus;
            private String departmentName;
            private Employee.EmployeeRank employeeRank;
        }
      
      
      //변경
       @Getter
       @Setter
       @NoArgsConstructor
       public static class UserResponse {
           private long employeeId;
           private String name;
           private String email;
           private String phoneNumber;
           private Employee.EmployeeRank employeeRank;
           private String extensionNumber;
           private String emergencyNumber;
        }
    ```
    

**Core-api**

Auth에서는 해당 메서드에서 반환 할 때 Response 정보가 바껴있었고 필드도 변경되었다.

일단 구조를 비슷하게 하기 위해서 한번 감싸는 구조로 변경하였다

```java
@Getter
@Setter
public class UserResponse {
    private EmployeeData data;
}

---
@Getter
@Setter
public class EmployeeData {
    private Long employeeId;
    private String name;
    private String departmentName;
}
```

---

사실 이런 문제는 소통만 했더라면 안 겪을 수 도 있었는데  Auth를 담당하는 분이 개인 사정으로 멀리 계시고 늘 소통할 수 있었던게 아니라서 이런 문제 가 생긴 것 같다 결국 Auth API의 메서드명이 변경되면서 이를 제대로 반영하지 못해 `null` 값이 반환되는 문제가 발생했다. 이를 방지하기 위해서는 API 변경 시 반드시 문서화하고, 변경 사항을 사용하는 모든 클라이언트에 신속하게 반영해야 한다는 점을 깨달았다…

또한 kafka 할때도 데이터 구조 때문에 애먹었는데 지금도 서비스 간에 주고받는 데이터 구조가 일관되지 않으면 역직렬화 과정에서 오류가 발생할 수 있다는 것을 알게 되었다. 이를 해결하기 위해서는 클라이언트와 서버 간의 DTO(Data Transfer Object) 구조를 항상 일치시켜야 하며, 변경 시에도 이에 맞추어 코드를 수정해야 한다는 점도 알게 되었다…


<br>
<br><br><br><br><br>