---
layout : single
title : "[Project] email 전송 EventPublisher 활용"
categories: Project
tag : [project1, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---


## EmailController의 역할을 Event로 옮긴 이유와 구현 과정

최근 프로젝트에서 기존에 EmailController가 담당하던 일을 Event로 옮기는 게 더 나을 것 같다는 생각이 들었다. 그 이유는, 기존 로직에서는 회원(member)을 등록하고 나서 인증 메일을 보내고, 사용자가 인증 코드를 입력하는 과정에서 총 세 번의 요청을 해야 했기 때문이다.

### 기존 로직의 문제점

먼저, 기존 로직을 간단히 설명하자면 이렇다:

1. 사용자가 회원 가입을 하면, 임시로 레디스(Redis)에 정보를 저장하고,
2. 그 후 EmailController를 통해 인증 메일을 발송하고,
3. 사용자가 인증 코드를 입력하면 그제야 회원 정보를 DB에 저장하는 구조다.

이 과정에서 Post 요청을 세 번이나 보내야 했다:

- 회원 등록 시 `/members`로 요청
- 이메일 전송 시 `/send-mail`로 요청
- 인증 코드 입력 시 `/verify`로 요청

물론 이 구조도 잘 작동하긴 하지만, 매번 요청을 보내야 하다 보니 번거롭고, 코드도 복잡해졌다. 그래서 이 과정을 Event로 처리하면 더 효율적이지 않을까 생각하게 됐다.

### Event로 로직을 옮긴 이유

Event로 로직을 옮기면, 회원 등록 시 한 번의 요청으로 인증 메일 발송까지 처리가 가능해진다. 이로써 요청 횟수도 줄고, 코드도 깔끔해진다.

기존에 EmailController에서 처리하던 메일 발송 로직을 Event로 옮기면, 회원(member)을 등록할 때 자동으로 이벤트가 발생하고, 그 이벤트를 통해 메일이 전송되는 구조로 변경할 수 있다. 이렇게 하면 회원 등록과 동시에 이메일이 발송되므로 별도의 메일 발송 요청을 할 필요가 없다.

### Event 구현: 간단하고 효과적인 방법

보통 Spring에서 이벤트를 처리할 때 `ApplicationEvent`를 상속받아 사용하곤 한다. 그런데 최근 스프링 버전에서는 굳이 `ApplicationEvent`를 상속받지 않아도 된다고 해서, 이번에는 상속받지 않고 간단하게 구현해봤다.

내가 선택한 방법은 다음과 같다:

1. 회원 등록 시 이벤트를 발생시키고,
2. 이벤트 핸들러가 이 이벤트를 받아서 이메일을 발송하는 로직을 실행한다.

이 과정에서 `member` 객체에서 필요한 `email` 정보만 가져와서 메일을 보내는 방식으로 구현했다. 이렇게 구현하면 기존의 로직과 크게 달라지지 않으면서도, 이메일 발송 과정을 더 효율적으로 처리할 수 있다.

### 구현 과정

1. **이벤트 클래스 생성**
    - 이메일 발송을 위한 이벤트 클래스를 생성했다. 이 클래스는 회원 등록 시 필요한 데이터를 담고 있다.
        
        ```java
        package com.springboot.helper.event;
        
        import com.springboot.member.entity.Member;
        import lombok.Getter;
        import org.springframework.context.ApplicationEvent;
        
        @Getter
        public class RegistrationEvent {
            private final Member member;
        
            public RegistrationEvent(Member member) {
                this.member = member;
            }
        }
        
        ```
        
2. **이벤트 리스너**
    - 1 에서 만든 이벤트를 처리하는 리스너(listener)를 만들어, 이벤트가 발생할 때마다 이메일을 발송하도록 설정했다.
        
        ```java
        package com.springboot.helper.event;
        
        import com.springboot.helper.email.EmailMessage;
        import com.springboot.helper.email.EmailService;
        import com.springboot.member.service.MemberService;
        import lombok.extern.slf4j.Slf4j;
        import org.springframework.context.event.EventListener;
        import org.springframework.mail.MailSendException;
        import org.springframework.scheduling.annotation.EnableAsync;
        import org.springframework.stereotype.Component;
        
        @Component
        @EnableAsync
        @Slf4j
        public class MemberRegisterEventHandler {
            private final EmailService emailService;
            private final MemberService memberService;
        
            public MemberRegisterEventHandler(EmailService emailService, MemberService memberService) {
                this.emailService = emailService;
                this.memberService = memberService;
            }
            @EventListener
            public void listen(RegistrationEvent event) {
                try{
                    EmailMessage emailMessage = EmailMessage.builder()
                            .to(event.getMember().getEmail())
                            .subject("[MeetBTI] 회원가입을 위한 인증코드 발송")
                            .build();
                    emailService.sendMail(emailMessage,"email");
        
                } catch (MailSendException e) {
                     log.error("MailSendException");
                } catch (RuntimeException e) {
                    log.error("RuntimeException : while sending");
                }
            }
        }
        
        ```
        
    - 이메일 컨트롤러에서 사용되는 것들을 listen 메서드에 적용했다.
        
        ```java
        package com.springboot.helper.email;
        
        import org.springframework.http.HttpStatus;
        import org.springframework.http.ResponseEntity;
        import org.springframework.web.bind.annotation.PostMapping;
        import org.springframework.web.bind.annotation.RequestBody;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        @RequestMapping("/send-mail")
        @RestController
        public class EmailController {
            private final EmailService emailService;
        
            public EmailController(EmailService emailService) {
                this.emailService = emailService;
            }
        
            //회원가입 이메일 인증코드 발송
            @PostMapping("/email")
            public ResponseEntity sendJoinMail(@RequestBody EmailPostDto emailPostDto) {
                EmailMessage emailMessage = EmailMessage.builder()
                        .to(emailPostDto.getEmail())
                        .subject("[MeetBTI] 회원가입을 위한 인증코드 발송")
                        .build();
        
                emailService.sendMail(emailMessage,"email");
                String message = "인증코드가 이메일로 발송되었습니다.";
                return new ResponseEntity<>(message, HttpStatus.OK);
            }
        }
        
        ```
        
3. **이벤트 발행**
    - 회원 등록이 완료되면 이 이벤트를 발행(publish)하도록 했다. 이를 통해 이메일 발송 요청이 자동으로 이루어진다.
        
        ```java
         public void createMember(Member member) {
                verifiedExistNickname(member.getNickname());
                verifiedExistEmail(member.getEmail());
        
                String key = member.getEmail() + ":email";
                redisUtil.setHashValueWithExpire(key, "memberInfo", member, 600);
                publisher.publishEvent(new RegistrationEvent(member));
            }
        ```
        
    

이렇게 로직을 Event로 옮기고 나니, 전체적인 코드가 훨씬 깔끔해졌다. 이제 한 번의 회원 가입 요청으로 이메일 발송까지 자동으로 처리되니, 사용자는 더 간편하게 가입할 수 있고 나도 요청이 줄어서 편해졌다!!!

테스트 결과 메일도 잘 온다 굿. (근데 메일 발송이 5초나 걸리는건 어떻게 하지.)