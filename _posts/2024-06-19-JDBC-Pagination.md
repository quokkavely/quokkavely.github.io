---
layout : single
title : "[JDBC] Pagination 실습"
categories: Spring
tag : [Spring, 실습,JBC]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# Pagination 구현 실습

## 1차 시도

Paginaiton이 처음이라 실습 하기 전부터 미리 검색해봤는데 다 무슨 말인지 잘 모르겠어서 

한참 검색하다가 정말 이해하기 쉽게  posting 된 사이트가 있어서 공식 문서 같은 건 줄 알았는데  블로그 같은 곳에 게시된 글이었음

그래도 처음에 접근하기 쉬워서 해당 포스트를 제일 많이 참고했다.

[https://medium.com/@barisalgun/spring-boot-pagination-524083e699fc](https://medium.com/@barisalgun/spring-boot-pagination-524083e699fc)

일단 어떻게든 구현해보고 @Repository 작성 안해서 오류도 겪다가…

POSTMAN에 드디어 get요청을 했다. 

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/dc511217-52aa-42ae-a7b8-1b96d9197b32" width=500>

### Pageinfo가 안 나옴

```java
{
    "data": [
        {
            "memberId": 16,
            "email": "hgd16@gmail.com",
            "name": "홍길동16",
            "phone": "010-1616-1616"
        },
        {
            "memberId": 15,
            "email": "hgd15@gmail.com",
            "name": "홍길동15",
            "phone": "010-1515-1515"
        },
        {
            "memberId": 14,
            "email": "hgd14@gmail.com",
            "name": "홍길동14",
            "phone": "010-1414-1414"
        },
        {
            "memberId": 13,
            "email": "hgd13@gmail.com",
            "name": "홍길동13",
            "phone": "010-1313-1313"
        }
    ],
    "pageInfo": {
        "page": 0,
        "size": 0,
        "totalElements": 0,
        "totalPages": 0
    }
}
```

안그래도 작성하다가 찜찜했던 부분이 있었는데 mapper 작성 부분에서 Dto로 변환시 PageInfo의 값을 불러와야 되는데 불러올 수 없어서 기본 생성자로  넣어서 그런 것 같음

```java
  default MemberPageResponseDto membersToMemberPageResponseDto(Page<Member> members){
        //members.getContent() => List<Member>
         MemberPageResponseDto memberPageResponseDto =
                 new MemberPageResponseDto(members.getContent(),new PageInfo());
                return memberPageResponseDto;
    }
```

PageInfo는 클라이언트가 요청시 자동으로 매개변수가 들어오나,,,? 생각했는데 

당연히 내가 구현해야 하고 이걸 어떻게 구현하나 싶어서 찾아보니 

Page는 Slice를 상속받고 Slice에 이미 구현되어있어서 

Page<T>에서 가져올 수 있음. 

즉 매개변수로 들어오는 Page<Member>members에서 . 만 누르면 다나옴..!!!

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6a1c0e41-47cf-47ce-a5a0-1935a4dcd5da" width=300>

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/763b4b85-9236-4116-a28d-40187210e512" width=300>

## 2차 시도 및 다른 Entity 구현

member이외에 Order와 Coffee에도 같은 방식으로 먼저 구현 후 리팩토링 하기로 함,

coffee 구현 다하고 오류 만났는데 get 요청시 500에러가 나왔다. 

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/57ddac0f-524a-4c01-84ec-21c61b8f3de6" width=500>

당황해서 콘솔보니  HttpMediaTypeNotAcceptableException: Could not find acceptable representation 라고 오류메세지 발생.. 

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/fd89ecc7-8a05-4ed8-9b3f-6c68462fed17" width=500>

혹시 애너테이션이나 뭐 빠진게 있나 한참 찾다가 알게된 것은

DTO 클래스 만들면서 @Getter 빠져먹음…!

 

다사 다난 했지만 무튼 다 구현했는데 몇가지 문제점이 있었음.

### 문제점

1. pagination 0부터 시작하므로 수정 필요
    1. data가 20부터인데 실제로 page 1을 조회하면 자꾸 20 부터 안나오고 어중간한 숫자가 조회됨.
    2. pagination할때 가장 첫 페이지는 0인 것,  이건 항상 기억하기.
2. 일관되게 동작해야 함 : PSA ( 어떤 것이 들어오든 findAll이 일관되게 동작해야 함. → *PagingAndSortingRepository 활용해보기*
3. PageResponseDto ⇒ 전역에서 사용할 수 있게 제네릭 활용

## Refactoring

### page, size 수정

1. coffee에 데이터는 2개 들어가 있고  coffeeId를 내림차순으로 조회하고 있기 때문에 page= 1에  size=1로 요청하면 coffeeId는 2가 나오는게 맞는데 1이 나옴, 
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d0ac90fd-f9d5-4758-a7be-3149a05fba36" width=500>
    

1. 몰랐는데 Pagination에서 page는 0부터 시작한다고 함.
    
    그래서 controller에서는 page-1, pageinfo에서 .getNumber()에는 +1로 수정함. 
    
    ```java
        //Cotroller
        @GetMapping
        public ResponseEntity getCoffees(@Positive @RequestParam(name="page",defaultValue = "1") int page,
                                         @Positive @RequestParam(name="size",defaultValue = "1") int size){
            Pageable pageable = PageRequest.of(page-1, size, Sort.by("coffeeId").descending());
            CoffeePageResponseDtos response = mapper.CoffeesToCoffeePageResponseDtos(coffeeService.findCoffees(pageable));
            return new ResponseEntity<>(response, HttpStatus.OK);
        }
        
        //PageInfo
        new PageInfo(coffees.getNumber()+1,
                    coffees.getSize(),
                    coffees.getNumberOfElements(),
                    coffees.getTotalPages());
    ```
    
2. 결과 잘나옴!
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/ee16a25c-b68b-4a89-8844-95324b2757af" width=500>
    

### *PSA 적용 → PagingAndSortingRepository*

1. *PagingAndSortingRepository 은 CrudRepository를 상속 받기 때문에 쉽게 수정 가능.*
    
    실제 구현체는 SimpleJdbcRepository.class
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9f3eb958-1537-448c-85a3-d387455db4a6" width=500>
    
    findAll이 구현되어 있음.
    

1. 그래서 다른 코드 수정없이 CrudeRepository 에서 PagingAndSortingRepository만 변경하면 되는데 나같은 경우 Sort 기능 추가 하고 싶어서 cotroller에서 추가함
    
    ```java
    Pageable pageable = PageRequest.of(page-1, size, Sort.by("coffeeId").descending());
    ```
    
2. PageRequest.of의 파라미터에 Sort가 없을 때는 기본적으로 아무것도 넣지 않음
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e5926ec6-48a9-4068-9454-b527697602bf" width=300>
    

### ✨✨✨~PageResponseDto ⇒ 전역에서 사용할 수 있게 하기.

1. 비슷한 기능을 하는 클래스를 전역으로 뺀 후 모든 엔티티에 적용할 수 있게 개선하기.
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/8a30083e-418a-4440-b7c4-9d5e67b2f806" width=400>
    
2. 전역에 있는 response 패키지에 PageResponseDtos만들고 해당 객체가 있는 mapper와 cotroller 수정
    
    ```java
    @AllArgsConstructor
    @Getter
    public class PageResponseDtos <T> {
       private List<T> data;
       private PageInfo pageInfo;
    }
    ```
    
    - 사실 제네릭을 사용한 적은 잘 없어서 data에 Object 타입 사용했었다가 강사님한테 코드 검토 요청시 Object는 사용하면 안된다고 하심.
    - 그래서 List<T>로  쓰려고 했었는데 자꾸 오류나서 사용 못했었다.
    - 클래스 이름에도 <T>를 명시해 주어야 에러가 안남!
    
3. 제네릭
    - 제네릭 대신 Object  올 수 있지만 쓰면 너무 위험하기 때문에 안 쓰는 것이 좋음.
    - 제네릭은 모든 타입을 허용하는 것이 아닌 제한하는 것 ⇒ 모든 타입 허용은 Object.
    - 제네릭은 타입을 제한하기 위해 사용, 반환값이 List라는 것을 보장할 수 있음 (상하한 제한 없이도)

## Comment

오늘 거의 대부분을 Paginaition 구현하는 실습과 리팩토링 하는 시간을 가졌는데

배우지 않았던 걸 구현하는 것은 정말 쉽지 않은 일인 것 같다…

Page와 Pageable이 이미 구현되어 있어서 다행이지 실제로 내가 구현하라고 했으면 아직도 끝나지 않았을지도…? 모른다…근데 강사님은 그것도 구현할 줄 알아야 한다고 하심.

그리고 자꾸 블로그 글을 참조하게 되는데 공식문서랑 인텔리제이에서 구현체 들어가서 확인하는 것 연습해야겠다.