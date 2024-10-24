---
layout : single
title : "[Project] netflix- eureka"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# netflix- eureka

오늘 MSA 적용하기로 결정하고 회의 후에 각자 주말 동안 MSA에 대해 공부해오기로 했다.

지금 한참 이력서랑 포트폴리오 낸다고 다들 바빠서 프로젝트에 오히려 집중 못하고 있는데

그래도 마지막 프로젝트 엄청 중요하니까, 프로젝트 잘 나와서 다들 도움이 되면 좋으니까 공부하고 로컬에서 환경 구성해보기로 했다.

# **Netflix Eureka**

1. **Netflix Eureka란?**
    - **Netflix Eureka**는 서비스 레지스트리(Server-side Discovery)로 동작하는 시스템
    - 각 마이크로서비스는 자신을 **Eureka Server**에 등록하고, 다른 서비스가 **Eureka Client**로서 해당 서비스의 위치 정보를 조회할 수 있다.
    - 기본적으로 Eureka는 **Eureka Server**와 **Eureka Client**로 나뉘어 동작하며, Eureka Server는 서비스의 등록과 조회를 관리하고, Eureka Client는 다른 서비스에 대한 정보를 얻고, 통신할 수 있게 해준다.
2. **Eureka의 구성 요소**
    - **Eureka Server**
        - **Eureka Server**는 모든 마이크로서비스가 등록되는 **중앙 서비스 레지스트리** 역할을 g한다.
        - 마이크로서비스는 시작할 때 **Eureka Server**에 자신의 위치(IP, 포트 등)를 등록하고, 주기적으로 하트비트(Heartbeat)를 보내서 살아 있음을 확인
        - **Eureka Server**는 각 서비스의 인스턴스 정보를 저장하고, 서비스가 중단되면 이 정보를 제거
    - **Eureka Client**
        - **Eureka Client**는 다른 서비스와 통신하기 위해 **Eureka Server**로부터 필요한 서비스를 조회해.
        - 서비스 클라이언트는 요청을 보내기 전에 **Eureka Server**에서 서비스를 찾아 해당 서비스의 IP 주소와 포트를 가져오고, 이를 사용해 통신을 할 수 있다.
        - 클라이언트는 또한 **Eureka Server**에 자신을 등록할 수도 있다.
3. **Eureka 동작 방식**
    - **서비스 등록**
        - 서비스가 시작되면 **Eureka Client**는 Eureka Server에 **HTTP 요청**을 통해 자신의 정보(IP, 포트, 상태)를 등록
        - 이때 서비스는 하트비트(Heartbeat)를 주기적으로 보내서 서비스가 여전히 살아 있음을 확인한다.
    - **서비스 조회**
        - 서비스가 다른 서비스에 요청을 보내기 전에, **Eureka Client**는 **Eureka Server**에서 요청할 서비스의 위치 정보를 조회한다.
        - Eureka Server는 등록된 서비스의 IP와 포트 정보를 반환하고, 클라이언트는 해당 서비스로 요청을 보낼 수 있다.
    - **하트비트 및 서비스 제거**
        - Eureka Client는 주기적으로 **하트비트**를 보내 서비스가 정상 동작 중임을 Eureka Server에 알림.
        - Eureka Server는 일정 기간 하트비트가 없으면, 해당 서비스를 **등록 목록에서 제거**하고 더 이상 요청을 받을 수 없게 한다.

4. **Netflix Eureka의 주요 기능**

- **서비스 레지스트리 관리**
    - Eureka Server는 **중앙 집중식 서비스 레지스트리** 역할을 하며, 모든 서비스의 IP 주소, 포트, 상태 정보를 관리한다.
    - 마이크로서비스 환경에서는 서비스가 동적으로 추가되고 제거될 수 있기 때문에, Eureka는 이러한 변화를 즉시 반영한다.
- **고가용성 지원**
    - Eureka는 **Failover** 기능을 제공하여, 만약 한 서비스 인스턴스가 다운되더라도 다른 인스턴스로 자동으로 요청을 보낼 수 있다.
    - 이를 통해 고가용성(High Availability)을 보장하고, 서비스 장애 시에도 다른 인스턴스를 통해 요청을 처리할 수 있다.
- **로드 밸런싱**
    - Eureka는 서비스의 여러 인스턴스를 관리하고, 클라이언트가 Eureka Server에서 여러 인스턴스 중 하나를 선택해 로드 밸런싱을 할 수 있게 해준다.
    - 클라이언트는 **Ribbon** 같은 로드 밸런서를 사용해 로드를 분산시킬 수 있다.
- **Self-Preservation Mode**
    - Eureka는 **Self-Preservation Mode**라는 기능을 제공
    - 네트워크 장애나 성능 이슈로 인해 일부 서비스의 하트비트가 Eureka Server에 도달하지 않더라도, 해당 서비스를 레지스트리에서 곧바로 제거하지 않고, 일정 기간 유지함으로써 시스템의 신뢰성을 높인다.
1. **Netflix Eureka의 작동 흐름**
    - **서비스 등록**
        - **마이크로서비스**가 시작되면 **Eureka Client**는 자신의 IP와 포트 정보를 **Eureka Server**에 등록.
        - 서비스는 주기적으로 Eureka Server에 **하트비트**를 보내서, 서비스가 아직 동작 중임을 알림.
    - **서비스 조회**
        - 서비스 A가 다른 서비스 B에 요청을 보내려면, **Eureka Client**는 먼저 **Eureka Server**에서 서비스 B의 위치 정보를 조회.
        - Eureka Server는 서비스 B의 **IP와 포트**를 반환하고, 서비스 A는 그 정보를 바탕으로 서비스 B로 요청을 보냄.
    - **Self-Preservation 모드**
        - 만약 서비스 B의 하트비트가 일정 시간 동안 수신되지 않으면, Eureka Server는 **Self-Preservation** 모드로 전환
        - 이 모드에서는 네트워크 문제가 일시적인 장애일 수 있기 때문에, 서비스 B를 곧바로 삭제하지 않고 일정 시간 동안 유효성을 유지함.
        - 네트워크가 복구되면, 서비스 B는 하트비트를 재개하고 다시 정상 상태로 등록될 수 있다.

## **Netflix Eureka** 실습

1.  https://start.spring.io/ 
    
    처음에 [https://start.spring.io/](https://start.spring.io/) 여기서 생성해도 되고 
    
    <img src="https://github.com/user-attachments/assets/01299840-4c9e-4f86-bc85-387d236db573" width=500/>
    
2. 기존에 프로젝트에서 아래 설정을 추가해주면 된다.

### 🔗Eureka Server

1. **build.gradle**
    
    ```groovy
    plugins {
    	id 'org.springframework.boot' version '2.7.0'
    	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    	id 'java'
    }
    
    javadoc.options.encoding = 'UTF-8'
    
    tasks.withType(JavaCompile) {
    	options.encoding = 'UTF-8'
    }
    
    //추가해 준 부분 > 스프링부트 버전에 따라 다름.
    ext {
    	set('springCloudVersion', "2021.0.4")
    }
    
    configurations {
    	compileOnly {
    		extendsFrom annotationProcessor
    	}
    }
    
    repositories {
    	mavenCentral()
    }
    
    dependencies {
    	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    	implementation 'org.springframework.boot:spring-boot-starter-validation'
    	implementation 'org.springframework.boot:spring-boot-starter-web'
    	compileOnly 'org.projectlombok:lombok'
    	runtimeOnly 'com.h2database:h2'
    	annotationProcessor 'org.projectlombok:lombok'
    	testImplementation 'org.springframework.boot:spring-boot-starter-test'
    	implementation 'org.mapstruct:mapstruct:1.5.1.Final'
    	annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.1.Final'
    	
    	// 의존성 추가 : 넷플릭스 유레카 서버
    	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    }
    
    tasks.named('javadoc') {
    	source = sourceSets.main.allJava
    	options.memberLevel = JavadocMemberLevel.PRIVATE
    	destinationDir = file("build/docs/javadoc")
    }
    
    //의존성 관리 추가
    dependencyManagement {
    	imports {
    		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    	}
    }
    ```
    

- 스프링 부트와 버전이 맞아야 한다. > 이거 안 맞아서 처음에 에러 엄청 났었다…ㅠㅠㅠ
    
    ```groovy
    //추가해 준 부분 > 스프링부트 버전에 따라 다름.
    ext {
    	set('springCloudVersion', "2021.0.4")
    
    ```
    
- 의존성 추가
    
    ```groovy
    // 의존성 추가 : 넷플릭스 유레카 서버
    	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    ```
    
- Spring Cloud 의존성을 관리하기 위한 dependencyManagement 섹션을 추가하는 코드
    
    ```groovy
    // Spring Cloud의 여러 모듈들을 통합 관리할 수 있다.
     dependencyManagement {
        imports {
            mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
        }
    }
    ```
    

1. **yaml**
    
    ```groovy
    spring:
      h2:
        console:
          enabled: true
          path: /h2
      datasource:
        url: jdbc:h2:mem:test
      jpa:
        hibernate:
          ddl-auto: create  # (1) 스키마 자동 생성
        show-sql: true      # (2) SQL 쿼리 출력
       
    # 추가
    server:
      port: 8761
    eureka:
      client:
        register-with-eureka: false
        service-url:
          default-zone: http://${eureka.instance.hostname}:${server.port}/eureka
        fetch-registry: false
      instance:
        hostname: localhost
    ```
    

1. **SpringStratApplication**
    
    ```groovy
    package com.springboot;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
    
    @SpringBootApplication
    @EnableEurekaServer //추가
    public class SpringStartApplication {
      public static void main(String[] args) {
        SpringApplication.run(SpringStartApplication.class, args);
      }
    }
    
    ```
    

1. [localhost:8761](http://localhost:8761) 로 접속하면 페이지가 뜬다.
    
   <img src="https://github.com/user-attachments/assets/b852839b-5ff8-4935-a75e-b31c49e01893" width=500/>
    

### 🔗 Eureka Client

1. **build.gradle**
    
    ```groovy
    plugins {
    	id 'org.springframework.boot' version '2.7.0'
    	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    	id 'java'
    }
    
    javadoc.options.encoding = 'UTF-8'
    
    tasks.withType(JavaCompile) {
    	options.encoding = 'UTF-8'
    }
    
    	//추가
    ext {
    	set('springCloudVersion', "2021.0.4")
    }
    
    configurations {
    	compileOnly {
    		extendsFrom annotationProcessor
    	}
    }
    
    repositories {
    	mavenCentral()
    }
    
    dependencies {
    	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    	implementation 'org.springframework.boot:spring-boot-starter-validation'
    	implementation 'org.springframework.boot:spring-boot-starter-web'
    	compileOnly 'org.projectlombok:lombok'
    	runtimeOnly 'com.h2database:h2'
    	annotationProcessor 'org.projectlombok:lombok'
    	testImplementation 'org.springframework.boot:spring-boot-starter-test'
    	implementation 'org.mapstruct:mapstruct:1.5.1.Final'
    	annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.1.Final'
    	
    	//client 추가
    	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    }
    
    tasks.named('javadoc') {
    	source = sourceSets.main.allJava
    	options.memberLevel = JavadocMemberLevel.PRIVATE
    	destinationDir = file("build/docs/javadoc")
    }
    
    dependencyManagement {
    	imports {
    		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    	}
    }
    
    ```
    
    - server와 동일하지만 dependency는 client 로 추가해야한다.
    
2. **yaml**
    
    ```groovy
    server:
      port: 9001
    eureka:
      client:
        register-with-eureka: true
        fetch-registry: true
        service-url:
          defaultZone: http://127.0.0.1:8761/eureka
    ```
    
3. **SpringStratApplication**
    
    ```groovy
    package com.springboot;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    
    @EnableDiscoveryClient //추가
    @SpringBootApplication
    public class SpringStartApplication {
      public static void main(String[] args) {
        SpringApplication.run(SpringStartApplication.class, args);
      }
    }
    
    ```
    

4. 주의서버 안키고 클라이언트 실행하면 아래와 같은 예외 발생 한다.
  <img src="https://github.com/user-attachments/assets/40e8bef0-f3af-4b75-bd9e-c7cf5eb28670" width=500/>
    

5. [localhost:8761](http://localhost:8761) 로 접속하면 인스턴스가 추가 되어 있다 (port : 9001)
    <img src="https://github.com/user-attachments/assets/f3b58213-0f28-4587-bfb7-fa2384b9145b" width=500/>
    
  <Br>
엄청 간단해 보이지만 생각보다 오래 걸렸다, 강의를 주고 사려니까 꽤 양이 많고 언제 다 볼 수 있을지 모르겠어서 그냥 블로그 글만 참고 했는데 다들 start.spring.io에서 파일을 많이 받아서 사용해서 시작했는지 build.gradle은 참고할 수 가없었다… 그래서 기존 파일에서 build.gradle 만 추가하려고 하니 에러가 많이 발생되었는데 그래도 하다보니 해결! 굿굿

여기서 어떻게 구현할 것인지 로컬로 여러 번 테스트 해보고 와야겠다.



<br>
<br><br><br><br><br>