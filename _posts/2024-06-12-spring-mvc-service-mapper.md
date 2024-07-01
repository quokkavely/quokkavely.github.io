---
layout : single
title : "[Spring MVC] 서비스계층(DI를 통해 API계층과 연동)"
categories: MVC
tag : [MVC,실습,Spring,개념정리]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# 서비스계층(DI를 통해 API계층과 연동)

## DI를 통해 서비스 계층 연동

Service

애플리케이션의 비즈니스 로직을 처리하기 위한 서비스 계층은 대부분 도메인 모델을 포함하고 있음. 

도메인 모델은 DDD와 관련이 깊음.

아직 DDD는 어려워서 도메인은 우리가 사용하고 있는 애플리케이션의 기능 축소단위라고 생각해도 됨..

## Service class 작성 실습

어제 했던 실습 파일에서 작성하기. [DTO 실습]

1. MemberController -  총 다섯 개 핸들러 메서드의 역할 요약
    - `postMember()` : 1명의 회원 등록을 위한 요청을 전달받음
    - `patchMember()` : 1명의 회원 수정을 위한 요청을 전달받음
    - `getMember()` : 1명의 회원 정보 조회를 위한 요청을 전달받음
    - `getMembers()` : N명의 회원 정보 조회를 위한 요청을 전달받음
    - `deleteMember()` : 1명의 회원 정보 삭제를 위한 요청을 전달받음
    
2. MemberService Class  기본구조 작성 [MemberController와 1:1로 매칭하기]
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9b068e3d-666c-4a05-bbc7-0a54f0c63bc7" width=300/>
    
    1. 특별한 예외상황이 없으면 controller와 1:1 로 메서드를 매칭해서 만듬.
   
   
        | Controller | MemberService |
        | --- | --- |
        | postMember | createMember |
        | patchMember | updateMember |
        | getMember | findMember |
        | getMembers | findMembers |
        | deleteMember | deleteMember |
    
    2. Controller 계층은 httpMethod 기준으로 이름 생성

    3. 서비스 계층은 CRUD 로 이름 생성

3. Member , MemberService 개선.
    1. Member 클래스
       
        ```java
        @Getter
        @Setter
        @NoArgsConstructor
        @AllArgsConstructor
        public class Member {
            private long memberId;
            private String email;
            private String name;
            private String phone;
        }
        
        ```
        
        `@NoArgsConstructor`  -  기본생성자를 만듬
        
        `@AllArgsConstructor`  - 생성자, 아래 박스
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/fc2189be-1b38-4fa0-9e83-7b39fdc8b82f" width=500/>
        
    2. MemberService 클래스
        1. createMember 
           
            ```java
            public Member createMember(Member member){
               Member createdMember = member;
               return createdMember;
            }
            ```
            
            - 회원정보 생성 → 데이터베이스에 해당 Member 객체를 보내서 정보를 저장해야 함, DB는 아직 학습하지 않았으므로, 나중에 변경해야 함
            - createdMember 라는 변수에 멤버의 속성을 저장
            - Id는 따로 안 받음(DB  불필요하게 접속 안함 ⇒ 비용증가방지.)
            - v1/members/1에서 1은 primariy key (PK), QueryParmeter
            
        2. updateMember
           
            ```java
            @PostMapping
                public ResponseEntity postMember(@Valid @RequestBody MemberPostDto memberPostDto) {
                    Member member = new Member();
                    member.setEmail(memberPostDto.getEmail());
                    member.setName(memberPostDto.getName());
                    member.setPhone(memberPostDto.getPhone());
                 Member response = memberService.createMember(member);
               return new ResponseEntity<>(response, HttpStatus.CREATED);   
              }
            ```
            
        3. findMember, findMembers
           
            ```java
            public Member findMember(long memberId){
                Member member =
                new Member(memberId,"제리","lucky-kor@cat.house","010-1234-5768");
            					//DB에 요청했다 치고 이런 데이터가 들어올 것 이다.라는 것.
                    return member;
            }
            public List<Member> findMembers(){
               List<Member> members=List.of(
            			  new Member(1,"제리","jerry@gamil.com","010-1234-1234"),
                    new Member(2, "쿼카", "quokka@naver.com","010-0312-0312"),
                    new Member(3, "통통", "tong@kakao.com","010-1111-1111")
                    );
                    
                    return members;
                }
            ```
            
            - 회원구조를 여러개 담기위해 List에 담음. =  더미데이터.
            - 각자 더미데이터가 범위가 달라서(findMember와 findMembers) findMember에 3을 넣는다고 통통이가 나오지 않음. ⇒ 3을 넣으면 3번의 제리 가 나오게 됨.
              
                <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5f1b537a-9e83-4782-a59b-df1d11853748" width=300/>
            
            
            
            
    
4. MemberController 개선
   
    ```java
    package com.springboot.member;
    
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.*;
    
    import javax.validation.Valid;
    import javax.validation.constraints.Min;
    import java.util.List;
    
    @RestController
    @RequestMapping("/v1/members")
    @Validated
    public class MemberController {
        private final MemberService memberService;
    
        public MemberController(MemberService memberService) {
            this.memberService = memberService;
        }
    
        @PostMapping
        public ResponseEntity postMember(@Valid @RequestBody MemberPostDto memberPostDto) {
            Member member = new Member();
            member.setEmail(memberPostDto.getEmail());
            member.setName(memberPostDto.getName());
            member.setPhone(memberPostDto.getPhone());
    
            Member response = memberService.createMember(member);
            return new ResponseEntity<>(memberPostDto, HttpStatus.CREATED);
        }
    
        @PatchMapping("/{member-id}")
        public ResponseEntity patchMember(@PathVariable("member-id") @Min(1) long memberId,
                                          @Valid @RequestBody MemberPatchDto memberPatchDto) {
            memberPatchDto.setMemberId(memberId);
    
            Member member= new Member();
            member.setMemberId(memberPatchDto.getMemberId());
            member.setName(memberPatchDto.getName());
            member.setPhone(memberPatchDto.getPhone());
    
            Member response = memberService.updateMember(member);
    
            // No need Business logic
    
            return new ResponseEntity<>(memberPatchDto, HttpStatus.OK);
        }
    
        @GetMapping("/{member-id}")
        public ResponseEntity getMember(@PathVariable("member-id") @Min(1) long memberId) {
            Member response = memberService.findMember(memberId);
    
            // not implementation
            return new ResponseEntity<>(response,HttpStatus.OK);
        }
    
        @GetMapping
        public ResponseEntity getMembers() {
          List<Member> response = memberService.findMembers();
    
            // not implementation
    
            return new ResponseEntity<>(response,HttpStatus.OK);
        }
    
        @DeleteMapping("/{member-id}")
        public ResponseEntity deleteMember(@PathVariable("member-id") @Min(1) long memberId) {
           memberService.deleteMember(memberId);
            // No need business logic
    
            return new ResponseEntity(HttpStatus.NO_CONTENT);
        }
    
        // 회원 정보는 구현해야 할 실습이 없습니다!
    }
    
    ```
    

### 현재 까지 문제점

1. controller 너무많은역할
2. 응답에 entity객체를 사용함.

## 매퍼(Mapper), DTO ↔ Entity

- Mapper 는 클래스를 변환하는 작업을 해줌.
  
    → DTO 클래스를 엔티티클래스 로 
    
    → 엔티티 클래스를 DTO 클래스.
    
1. **MemberMapper 생성**
   
    메서드 이름 명명할 때는 어디서 어디로 보내는지 명확히 해야 함 (메서드 명이 길어지더라도!)
    
    ```java
    package com.springboot.member;
    
    import javax.swing.plaf.metal.MetalComboBoxEditor;
    
    @Component
    public class MebmerMapper {
        public Member memberPostDtoToMember(MemberPostDto memberPostDto){ 
                      
            return new Member(0L,
                    memberPostDto.getEmail(),
                    memberPostDto.getName(),
                    memberPostDto.getPhone());
        }
        public Member memberPatchDtoToMember(MemberPatchDto memberPatchDto){
            return new Member
                    (memberPatchDto.getMemberId(),
                    null,
                    memberPatchDto.getName(),
                    memberPatchDto.getPhone());
        }
        public MemberResponseDto memberToMemberResponseDto(Member member){
            return new MemberResponseDto
                    (member.getMemberId(),
                    member.getEmail(),
                    member.getName(),
                    member.getPhone());
        }
    }
    
    ```
    
    - memberPostDtoToMember : MemberPostDto → Member 로 변환 (DTO를 엔티티로)
    - memberPatchDtoToMember : MemberPatchDto → Member 로 변환 (DTO를 엔티티로)
    - memberToMemberResponseDto :  Member → MemberResponseDto (엔티티를 DTO로)
    - @Component 붙여줌 ⇒ 빈으로 등록하기 위함.
        - MemberController에는 @RestController
        - MemberResponseDto / MemberPatchDto / MemberPostDto 는 왜 없지???
        - MemberService @Service
        - ComponentScan역할을 @SpringbootApplication 역할.
    
2. **MemberResponseDto 클래스 생성**
   
    응답 데이터의 역할을 해주는 DTO 클래스
    
    ```java
    package com.springboot.member;
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    
    @Getter
    @AllArgsConstructor
    public class MemberResponseDto {
        private long memberId;
        private String name;
        private String email;
        private String phone;
    }
    ```
    
3. **MemberController의 핸들러 메서드에 매퍼(Mapper) 클래스 적용하기**
   
    ```java
    package com.springboot.member;
    
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.*;
    
    import javax.validation.Valid;
    import javax.validation.constraints.Min;
    import java.util.List;
    
    @RestController
    @RequestMapping("/v1/members")
    @Validated
    public class MemberController {
        private final MemberService memberService;
        private final MemberMapper memberMapper;
    
        public MemberController(MemberService memberService, MemberMapper mebmerMapper) {
            this.memberService = memberService;
            this.memberMapper = mebmerMapper;
        }
    
        @PostMapping
        public ResponseEntity postMember(@Valid @RequestBody MemberPostDto memberPostDto) {
    
            Member member = memberMapper.memberPostDtoToMember(memberPostDto);
            Member responsentity = memberService.createMember(member);
            MemberResponseDto responseDto= memberMapper.memberToMemberResponseDto(responsentity);
    
            return new ResponseEntity<>(responseDto, HttpStatus.CREATED);
        }
    
        @PatchMapping("/{member-id}")
        public ResponseEntity patchMember(@PathVariable("member-id") @Min(1) long memberId,
                                          @Valid @RequestBody MemberPatchDto memberPatchDto) {
    
            Member member = memberMapper.memberPatchDtoToMember(memberPatchDto);
            Member responsEntity = memberService.updateMember(member);
            MemberResponseDto responseDto = memberMapper.memberToMemberResponseDto(responsEntity);
            return new ResponseEntity<>(responseDto, HttpStatus.OK);
        }
    
    }
    ```
    
    - DTO로 받아서 Entity로 변환 후 다시 DTO로 반환. ⇒  민감한 개인정보도 막을 수 있음
    - 원래는 RequestParam으로 경로 다 지정해주었지만, DTO가 그걸 대체해 줌.
    
    ```java
    @GetMapping("/{member-id}")
        public ResponseEntity getMember(@PathVariable("member-id") @Min(1) long memberId) {
       
            Member member = memberService.findMember(memberId);
            MemberResponseDto responseDto = memberMapper.memberToMemberResponseDto(member);
    
            return new ResponseEntity<>(responseDto,HttpStatus.OK);
        }
    
        @GetMapping
        public ResponseEntity getMembers() {
          List<Member> response = memberService.findMembers();
            List<MemberResponseDto> responseDtos = memberMapper.memberToMemberResponseDtos(response);
            return new ResponseEntity<>(responseDtos, HttpStatus.OK);
        }
    ```
    
    - 컬렉션을 순회할 때는 기본적으로 forEach 사용해야 함 = 왜?? Oracle에서 권장하니까!
    - List 순회하기 위해서  controller 가 아닌 **MemberMapper 클래스**에서 작성
      
        ```java
        public List<MemberResponseDto> memberToMemberResponseDtos(List<Member> members){
            List<MemberResponseDto> list = new ArrayList<>();
            for(Member member : members){
                  list.add(memberToMemberResponseDto(member));
            }
            return list;
         }
        ```
        
    - 근데 기껏 다 구현했는데,  Mapstruct를 이용해서 Mapper를 자동 생성할 수 있다고 함.
    

### Mapstruct 이용하여 Mapper 자동생성.

1. 의존 라이브러리 설정 : build.gradle 파일에 추가 후 다시 build.
   
    ```groovy
    dependencies {
    	...
    	...
    	implementation 'org.mapstruct:mapstruct:1.4.2.Final'
    	annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
    }
    
    ```
    
2. 매퍼(Mapper) 인터페이스 생성.
   
    ```groovy
    package com.springboot.member.mapstruct;
    
    import com.springboot.member.Member;
    import com.springboot.member.MemberPatchDto;
    import com.springboot.member.MemberPostDto;
    import com.springboot.member.MemberResponseDto;
    import org.mapstruct.Mapper;
    
    import java.util.List;
    
    @Mapper(componentModel = "spring")
    public interface MemberMapper {
        Member memberPostDtoToMember(MemberPostDto memberPostDto);
        Member memberPatchDtoToMember(MemberPatchDto memberPatchDto);
        MemberResponseDto memberToMemberResponseDto(Member member);
        List<MemberResponseDto> membersToMemberResponseDtos(List<Member> members);
    }
    ```
    
    - 인터페이스 메서드는 모두 추상메서드, 시그니처만 있고 바디가 없음. 그래도 알아서 다 만듬.


## Comment

for 문돌려서 열심히 mapper클래스 만들었는데 강사님은 stream이랑 for문 둘다 보여주심. stream 배웠을 때부터 잘쓰면 좋다고 해서 써봐야지 말만하고 연습문제만
한번 더 풀어본게 다...코딩테스트도 스트림 쓰는 사람들꺼 와 멋있다..하면서
캡쳐만 해두고 직접 쳐본적은 거의 없었는데
나중에 코틀린이나 웹플럭스 사용할 때는 람다식 많이 사용한다며
공부하기를 권장하셨다...ㅠㅠ 스프링따라가기도 버거울 것 같은데
아직 was, 톰캣, Bean도 이해못해서 아...뭐지 이게 하고있음..
후.. 오늘도 불태워보자 

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
        
<div class="notice" markdown="1">
🍒 `공지` 
<h4> - <u>정보 공유가 아닌 개인이 공부하고 기록하기 복습하기 위한 용도입니다.</u></h4>
</div>


