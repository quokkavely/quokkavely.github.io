---
layout : single
title : "[Spring MVC] DTO 실습"
categories: MVC
tag : [MVC,실습,Spring]
author_profile: true
---

# [Spring MVC] DTO 실습


### 처음작성 - 오류발생

1. CoffeePostDto
    
    ```java
    public class CoffeePostDto {
        @NotBlank
        private String korName;
    
        @NotBlank
        @Pattern(regexp ="^[A-Za-z]+(\\s[A-Za-z]+)*$", message = "영문만 가능합니다.")
        private String engName;
    
        @Min(100)
        @Max(50000)
        private int price;
        }
    ```
    
2. CoffeePatchDto
    
    ```java
    package com.springboot.coffee;
    import com.springboot.member.NotSpace;
    import javax.validation.constraints.*;
    
    @Getter @Setter
    public class CoffeePatchDto {
    		@Positive    
        private long coffeeId;
    
        @NotBlank
        private String korName;
    
        @NotSpace
        @Pattern(regexp ="^[A-Za-z]+(\\s[A-Za-z]+)*$", message = "영문만 가능합니다.")
        @NotBlank
        private String engName;
    
        @Min(100)
        @Max(50000)
        private int price;
    }
    ```
    
3. Coffeecontroller 클래스의 patchCoffee Method
    
    ```java
    
    @PatchMapping("/{coffee-id}")
    public ResponseEntity<CoffeePatchDto> patchCoffee(@PathVariable("coffee-id") @Positive long coffeeId,
                                                      @RequestBody @Valid CoffeePatchDto coffeePatchDto) {
        coffeePatchDto.setCoffeeId(coffeeId);
    
        return new ResponseEntity<>(coffeePatchDto, HttpStatus.OK);
    }
    ```
    
4. 문제
    1. 오류발생 ⇒ [localhost:8080/v1/coffees/1](http://localhost:8080/v1/coffees/1) 로 patch하면  id는 0이상만 올 수 있다고 에러남
        - MethodArgumentNotValidException
        
        ```java
         Validation failed for argument [1] in 
         public org.springframework.http.ResponseEntity 
         com.springboot.coffee.CoffeeController
         .patchCoffee(long,com.springboot.coffee.CoffeePatchDto): 
         [Field error in object 'coffeePatchDto' on field 'coffeeId'
         : rejected value [0]; codes [Positive.coffeePatchDto.coffeeId,
         Positive.coffeeId,Positive.long,Positive]; 
         arguments [org.springframework.context.support
         .DefaultMessageSourceResolvable: codes [coffeePatchDto.coffeeId,coffeeId]; 
         arguments []; default message [coffeeId]]; default message [0보다 커야 합니다]] ]
        ```
        
        - CoffeePatchDto에서  @Positive    private long coffeeId; - 이게 문제가 됨.
        - 왜냐면, CoffeePatchDto.coffeeId의 초기화 값이 0이 되고 @Positive 의 유효성 검사가 실패되어  메서드가 호출되기 전에 유효성 검사에서 이미 실패했기 때문에 예외가 발생 ⇒ CoffeePatchDto의 coffeeId의 @Positive 어노테이션을 제거하면 coffeeId에 대한 유효성 검사가 수행되지 않으므로, 초기화되지 않은 상태에서도 예외가 발생하지 않습니다. 따라서, coffeePatchDto.setCoffeeId(coffeeId); 메서드 호출 후에 제대로 동작
        - @PathVariable("coffee-id") @Positive long coffeeId, 여기만 positive 작성해주면 됨.
        
        ---
        
        원시타입은 기본초기화 값이 있음.
        
        ![Untitled](%5BSpring%20MVC%5D%20DTO%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%20b608249a5e7b465689a9e6216f51ccaf/Untitled.png)
        
        Referece type은 초기화 해주어야 함.
        
        ---
        
        **추가. coffeeId 검증**
        
        ![Untitled](%5BSpring%20MVC%5D%20DTO%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%20b608249a5e7b465689a9e6216f51ccaf/Untitled%201.png)
        
        @PathVariable의 검증이 제대로 되려면 class위에 @Validated 필요
        
        ![Untitled](%5BSpring%20MVC%5D%20DTO%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%20b608249a5e7b465689a9e6216f51ccaf/Untitled%202.png)
        
        @Valid는 기본적으로 컨트롤러에서만 동작하며 기본적으로 다른 계층에서는 검증이 안됨, 다른 계층에서 파라미터를 검증하기 위해서는 @Validated와 결합되어야 함.
        
        []
        
        @Valid 동작원리
        
        모든 요청은 프론트 컨트롤러인 디스패처 서블릿을 통해 컨트롤러로 전달된다. 전달 과정에서는 컨트롤러 메소드의 객체를 만들어주는 ArgumentResolver가 동작하는데, @Valid 역시 ArgumentResolver에 의해 처리가 된다.대표적으로 @RequestBody는 Json 메세지를 객체로 변환해주는 작업이 ArgumentResolver의 구현체인RequestResponseBodyMethodProcessor가 처리하며, 이 내부에서 @Valid로 시작하는 어노테이션이 있을 경우에 유효성 검사를 진행함.
        
        @Valid는 기본적으로 컨트롤러에서만 동작하며 기본적으로 다른 계층에서는 검증이 안됨, 다른 계층에서 파라미터를 검증하기 위해서는 @Validated와 결합되어야 함.
        
        출처 : [https://mangkyu.tistory.com/174](https://mangkyu.tistory.com/174)
        
        @Validated 동작원리
        
        특정 ArgumnetResolver에 의해 유효성 검사가 진행되었던 @Valid와 달리, @Validated는 AOP 기반으로 메소드 요청을 인터셉터하여 처리된다. @Validated를 클래스 레벨에 선언하면 해당 클래스에 유효성 검증을 위한 AOP의 어드바이스 또는 인터셉터(MethodValidationInterceptor)가 등록된다. 그리고 해당 클래스의 메소드들이 호출될 때 AOP의 포인트 컷으로써 요청을 가로채서 유효성 검증을 진행한다.이러한 이유로 @Validated를 사용하면 컨트롤러, 서비스, 레포지토리 등 계층에 무관하게 스프링 빈이라면 유효성 검증을 진행할 수 있다. 대신 클래스에는 유효성 검증 AOP가 적용되도록 @Validated를, 검증을 진행할 메소드에는 @Valid를 선언해주어야 한다.이러한 이유로 @Valid에 의한 예외는 MethodArgumentNotValidException이며, @Validated에 의한 예외는  ConstraintViolationException
        

### 다시 작성- 오류 개선,  but Body의 attribute가 null일 때도 유효성 검증함

```java
    @PatchMapping("/{coffee-id}")
    public ResponseEntity patchCoffee(@PathVariable("coffee-id") @Positive long coffeeId,
            @RequestBody @Valid CoffeePatchDto coffeePatchDto) {

        coffeePatchDto.setCoffeeId(coffeeId);

        return new ResponseEntity<>(coffeePatchDto, HttpStatus.OK);
    }
```

```java
package com.springboot.coffee;
import com.springboot.member.NotSpace;

import javax.validation.constraints.*;

public class CoffeePatchDto {

    private long coffeeId;

    @NotSpace
    private String korName;

    @Pattern(regexp ="^[A-Za-z]+(\\s[A-Za-z]+)*$", message = "영문만 가능합니다.")
    @NotBlank
    private String engName;

    @Min(100)
    @Max(50000)
    private integer price;

    public String getKorName() {
        return korName;
    }

    public void setKorName(String korName) {
        this.korName = korName;
    }

    public String getEngName() {
        return engName;
    }

    public void setEngName(String engName) {
        this.engName = engName;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public long getCoffeeId() {
        return coffeeId;
    }

    public void setCoffeeId(long coffeeId) {
        this.coffeeId = coffeeId;
    }

}
```

Price에 null값이 안들어와서 @NotBlank 시도 ⇒ UnexpectedTypeException  발생

알고보니 @NotBlank도 null 못 들어오고 오히려 @min, @max는 null이 들어올 수 있음.

그래서 int는 초기화 값이 0이여서 그렇다는 걸 알게 되었고,

Integer로 바꿔도 에러 나서 왜 그런가 했더니, 위에 필드에만 변경하고

getter와 setter를 미리 만들었더니 여기는 int값으로 유지되고 있었음….!

모두 변경하니 null값이 잘 들어옴.

### 완성

[CoffeePatchDto]

```java
import lombok.Getter;
import lombok.Setter;
import com.springboot.member.NotSpace;
import javax.validation.constraints.*;

@Getter @Setter
public class CoffeePatchDto {

    private long coffeeId;
    @NotSpace //customAnnotation
    private String korName;

    @Pattern(regexp ="^[A-Za-z]+(\\s[A-Za-z]+)*$", message = "영문만 가능합니다.")
    private String engName;

    @Min(100)
    @Max(50000)
    private Integer price;
}

```

[CoffeePostDto]

```java
import lombok.Getter;
import lombok.Setter;

import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;

@Getter
@Setter
public class CoffeePostDto {
    @NotBlank
    private String korName;

    @NotBlank
    @Pattern(regexp ="^[A-Za-z]+(\\s[A-Za-z]+)*$", message = "영문만 가능합니다.")
    private String engName;

    @Min(100)
    @Max(50000)
    private int price;

}
```

[CoffeeController]

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import javax.validation.Valid;
import javax.validation.constraints.Positive;

@Validated
@RestController
@RequestMapping("/v1/coffees")
public class CoffeeController {
    // 1. DTO 클래스 및 유효성 검증을 적용하세요.
    @PostMapping
    public ResponseEntity postCoffee(@RequestBody @Valid CoffeePostDto coffeePostDto) {
        return new ResponseEntity<>(coffeePostDto, HttpStatus.CREATED);
    }

    // 2. DTO 클래스 및 유효성 검증을 적용하세요.
    @PatchMapping("/{coffee-id}")
    public ResponseEntity patchCoffee(@PathVariable("coffee-id") @Positive long coffeeId,
            @RequestBody @Valid CoffeePatchDto coffeePatchDto) {

        coffeePatchDto.setCoffeeId(coffeeId);

        return new ResponseEntity<>(coffeePatchDto, HttpStatus.OK);
    }
```

### Comment

실무에도 이렇게 null을 받나 싶어서 의문이 계속 들었고 DB를 아직 사용하지 않아서 DB에는 변경한 값만 들어가는 건지 궁금했는데

가격을 바꾸고 싶어서 PATCH에 Body 값으로 {”price”:3000} 만 넣게 되면(korName, engName

만약 DB에 null을 보내면 null이 들어가게 된다고 함.

이걸 방지하기 위해 Optional class 또는 람다식을 사용해서 대체할 수 있다고 함. (실무에서 대부분 람다식 사용)

무튼 int랑 integer때문에 애 많이 먹었는데 강사님은 NotZero라는 annotation을 custom으로 만들어 버림. 😮 ,,, 그리고 custom Annotation 실무에서도 자주 사용되는데 프로젝트 전에 한번 만들어보는 것도 좋다고 하심, !