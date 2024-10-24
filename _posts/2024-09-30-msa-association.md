---
layout : single
title : "[Project] MSA -연관관계 고민"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

### MSA에서 연관관계 관리하기

이번 프로젝트에서 MSA를 도입하면서 가장 큰 고민이 생긴 건 **연관관계를 어떻게 처리할지**였다. 기존 모놀리식 아키텍처에서는 JPA로 간단하게 @ManyToOne, @OneToMany 등을 통해 엔티티 간의 관계를 바로 맺고, 필요한 데이터를 쉽게 가져올 수 있었다. 그런데 MSA에서는 이런 방식이 불가능했다. 각 서비스가 독립적으로 운영되기 때문에 서비스 간 데이터를 직접 참조할 수 없다는 점이 가장 큰 차이였다.

### 느슨한 연관관계로 서비스 독립성 유지

MSA에서는 중요한 것이 바로 서비스 간의 **느슨한 결합**이다. 기존의 모놀리식 구조처럼 서비스 간에 강한 연관관계를 맺으면, 서비스 하나가 변경될 때 다른 서비스도 영향을 받게 된다. 하지만 MSA에서는 각 서비스가 서로 독립적으로 운영되기 때문에 **ID만으로 연관관계를 관리하고 필요한 데이터는 주고받는 방식**이 더 적합하다.

특히 이 방식은 서비스 간의 **독립 배포**가 가능하다는 큰 장점이 있었다. 하나의 서비스가 변경되더라도 다른 서비스에는 영향을 주지 않기 때문에, 더 안정적이고 유연한 구조가 될 수 있었다.

 서비스 간에 데이터를 직접 참조하지 않고, **ID로만 연관관계를 관리**하면서 필요한 데이터를 주고받는 방식으로 해결해야 했다. 여기서 내가 많이 생각했던 부분은, 서비스 간에 데이터를 주고받는 방법으로 주로 **API 호출**이나 **메시지 큐**를 사용하는 것이었다.

### 연관관계는 ID로만 관리

먼저, 각 서비스가 다른 서비스의 엔티티와 연관관계를 맺는 방식 대신, 단순히 **ID만 저장**하는 방식으로 구현했다. 예를 들어 `Document` 서비스와 Employee 서비스 간의 연관관계를 생각해보면, `Document` 엔티티에는 employeeID만 저장되고, 그 외의 Employee 정보는 저장하지 않는다. 

그럼 employee와 관련된 정보가 필요할 때는 어떻게 할지에 대해도 고민했는데 이때는 employeeId를 기반으로 Employee 서비스에 **API를 호출**해서 데이터를 가져오는 것이다.

```java
@Entity
public class Document {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String content;
    private Long employeeId;
```

이 방식은 처음엔 조금 불편하게 느껴졌다. 왜냐면, 기존에는 JPA로 연관관계를 맺어두면 바로 데이터를 가져올 수 있었는데, 이제는 항상 API 호출을 통해 데이터를 가져와야 하기 때문이다. 하지만 이 방식은 각 서비스의 **독립성**을 유지하면서도 필요한 데이터를 가져올 수 있다는 큰 장점이 있었다.

---

### 서비스 간 통신 방식: REST API

**서비스 간의 통신을 어떻게 할지**는 또 다른 중요한 문제였다. 여기서 두 가지 주요 방식이 나왔는데, 하나는 **REST API**를 이용한 동기 방식이고, 다른 하나는 **메시지 큐**를 이용한 비동기 방식이다.

**REST API 방식**은 필요한 데이터를 실시간으로 가져와야 할 때 유용하다. 예를 들어, `Document` 서비스에서 Employee 에 대한 정보를 바로 가져와야 할 때, 아래와 같이 REST API를 호출해서 해결할 수 있다.

```java
@RestController
@RequestMapping("/documents")
public class DocumentController {

    @Autowired
    private EmployeeServiceClient employeeServiceClient;

    @GetMapping("/{documentId}")
    public DocumentResponse getDocument(@PathVariable Long documentId) {
        Document document = documentService.getDocumentById(documentId);

        // Workflow 서비스에 API 호출해서 Workflow 정보 가져오기
        Employee employee = EmployeeServiceClient.getEmployee(document.getEmployeeId());

        //...
    }
}
```

이 코드는 `Document` 서비스가 employeeId를 이용해 Employee 서비스에 있는 데이터를 가져오는 방식이다. 간단하게 말해서, **필요할 때만 다른 서비스의 데이터를 가져오는 방식**으로 통신을 처리하는 것이다.

하지만 실시간으로 데이터를 가져올 필요가 없는 경우도 있다. 예를 들어, 결재가 완료된 후 `Document` 서비스에 그 사실을 알릴 때는, 바로 데이터를 가져올 필요가 없으니 **메시지 큐**를 사용해 비동기적으로 처리할 수 있다. 메시지 큐는 각 서비스가 독립적으로 동작할 수 있도록 해주면서도, 서비스 간의 이벤트를 효율적으로 전달해준다.

---

### 서비스 간 통신 방식: **Feign** vs **WebClient**

이제 서비스 간의 데이터를 어떻게 주고받을지 고민하게 되었다. 여기서 내가 선택한 방법은 **Feign**이었지만, **WebClient**에 대해서도 많이 고민했다. REST API 통신에는 RestTemplate도 있지만 Spring 5부터는 WebClient가 권장 되기도하고 이점이 더 많은 것 같아서 RestTemplate은 배제했다.

1. **Feign Feign**
    - API 호출을 선언적으로 처리할 수 있는 라이브러리이다.
    - API를 호출할 때 인터페이스만 정의해두고, 마치 메서드 호출하듯이 사용할 수 있다.
    - 예를 들어, **Document** 서비스에서 **Workflow** 서비스를 호출할 때
        
        ```java
        @FeignClient(name = "employee-service")
        public interface WorkflowServiceClient {
            @GetMapping("/employees/{employeeId}")
            Employee getEmployee(@PathVariable Long employeeId);
        }
        
        ```
        
    - 위 코드 처럼 Feign의 장점은 선언적으로 API를 호출할 수 있다는 점이다.
    - 별도의 로직 없이도 간단하게 서비스 간의 통신을 처리할 수 있기 때문에 코드가 깔끔하고 유지보수하기도 좋다.
2. **WebClient**
    - **Spring WebFlux**에서 도입된 **비동기** 및 **리액티브 프로그래밍**을 위한 HTTP 클라이언트
    - **비동기 처리**가 가능해서, 요청을 보내고 바로 다음 작업을 진행할 수 있어. 응답이 오면 그때 결과를 처리하는 방식
    - WebClient는 **RestTemplate의 대체**로도 많이 사용돼, Spring 5부터는 WebClient가 권장되는 HTTP 클라이언트
    - 여러 API 호출을 병렬로 처리하거나, 대규모 트래픽을 효율적으로 처리할 때 **WebClient**가 더 적합하다
    - 예제) **Document** 서비스에서 **Workflow** 서비스를 호출할 때
        
        ```java
        @Service
        public class WorkflowServiceClient {
        
            private final WebClient webClient;
        
            public WorkflowServiceClient(WebClient.Builder webClientBuilder) {
                this.webClient = webClientBuilder.baseUrl("http://employee-service").build();
            }
        
            public Mono<Employee> getEmployee(Long employeeId) {
                return webClient.get()
                        .uri("/employees/{employeeId}", employeeId)
                        .retrieve()
                        .bodyToMono(Employee.class);
            }
        }
        ```
        

### 3. 결론: Feign vs WebClient

결국 내가 선택한 건 **Feign**이었지만, 프로젝트 상황에 따라 **WebClient**를 쓰는 게 더 나은 경우도 많을 수 도 있을 것 같다. 특히 복잡한 API 호출이나 비동기 처리가 필요할 때는 **WebClient**가 더 적합하다 생각된다. 반면, API 호출이 간단하고 선언적인 코드가 필요할 때는 **Feign**을 사용하면 코드가 더 깔끔해져서 좋다 굿굿..👍

---

이번 프로젝트에서 MSA로 전환하면서 기존의 JPA 연관관계 방식을 그대로 사용할 수 없다는 것을 알게 되었다. 대신, 각 서비스는 **ID만으로 연관관계를 관리**하고, 필요한 데이터를 API 호출이나 메시지 큐로 주고받는 방식으로 처리해야 했다. 이 방식은 처음에는 복잡하게 느껴졌지만, 서비스 간의 **독립성**을 유지하면서도 확장성과 유연성을 확보할 수 있는 좋은 방법이었다.

다시 생각해보면, 이 방식은 장기적으로는 유지보수도 쉬워지고, 시스템의 복잡도도 줄여주는 효율적인 방법이라는 걸 알게 되었다.

<br>
<br><br><br><br><br>