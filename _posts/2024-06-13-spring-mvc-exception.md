---
layout : single
title : "[spring MVC] Exception(@RestControllerAdvice,@ExceptionHandler)"
categories: MVC
tag : [MVC,실습,Spring,개념정리]
author_profile: true
---

**우리가 앞으로 만나 볼 수 있는 예외**

- 클라이언트 요청 데이터에 대한 유효성 검증(Validation)에서 발생하는 예외
- 서비스 계층의 비즈니스 로직에서 던져지는 의도된 예외
- 웹 애플리케이션 실행 중에 발생하는 예외(RuntimeException)

**예외란?** 

- cheked exception
- ⇒ 개발자가 예외처리를 꼭 해야하는 것 ! 즉 JVM이 읽을 수 있게 바꿔주는 것

## **@ExceptionHandler**

### 0. 이 때 동안 만들었던 **MemberController class에 예외처리 코드** 입력

```java
    @ExceptionHandler
    public ResponseEntity handleException(MethodArgumentNotValidException e){
        final List<FieldError>fieldErrors = e.getBindingResult().getFieldErrors();
        return new ResponseEntity(fieldErrors,HttpStatus.BAD_REQUEST);
    }

```

- MethodArgumentNotValidException은  request를 잘못 했을 때 발생 (필드 누락이나 바디가 형식에 맞지 않거나 유효성 검증 조건에 맞지 않으면 발생할 수 있음.)
    - 예를 들어, DTO클래스에서 `*@NotSpace*(message = "커피명(한글)은 공백이 아니어야 합니다.")` 를 name위에 적용하고 Controller에 @Valid를 적용했는는데 postman에서 postman에서 Request body에 attruibute에 blank 작성 하고  요청했을 경우에  발생할 수 있다.
- 매개인자로 들어온 `MethodArgumentNotValidException` 객체에서 `getBindingResult().getFieldErrors()`를 통해 발생한 에러 정보를 확인 가능
- List<<FieldError>>fieldErrros에서 FieldError는?? 저런 클래스를 작성한 것이 없는데..! 찾아보니
    - Spring Framework에서 제공하는 클래스라고 함.
    - org.springframework.validation 패키지에 속해 있으며, 유효성 검사 중 발생한 오류에 대한 정보를 포함
        1. **field**: 유효성 검사가 실패한 필드의 이름. 
        2.  **rejectedValue**: 유효성 검사가 실패한 필드의 값 
        3. **defaultMessage**: 기본 에러 메시지 
        4. **objectName**: 오류가 발생한 객체의 이름 
        5. **codes**: 오류 코드를 배열 형태로 저장
        6. **arguments**: 오류 메시지의 인자
    
- 위에서 확인한 정보를 `ResponseEntity`를 통해 Response Body로 전달
- postman을 이용해 body를 잘못 입력 후 post요청하면 하기와 같이 메세지가 뜬다.
  
    ![Untitled](%5Bspring%20MVC%5D%20Exception(@RestControllerAdvice,@Exce%2045ac29926691473b902e4cb2c5e369ba/Untitled.png)
    
- 위의 메세지는 너무 길고 필요 없는 데이터가 많음

### **1. 문제점 보완 → ErrorResponse 클래스 적용**

1. **ErrorResponse**
   
    ```java
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    
    import java.util.List;
    
    @Getter
    @AllArgsConstructor
    public class ErrorResponse {
        private List<FieldError> fieldErrors;
    
        @Getter
        @AllArgsConstructor
        public static class FieldError {
            private String field;
            private Object rejectedValue;
            private String reason;
        }
    }
    
    ```
    
    - `FieldError`라는 별도의 static class를 `ErrorResponse` 클래스의 멤버 클래스로 정의(innerclass 라기보단  ErrorResponse 클래스의 static 멤버 클래스라고 부르는 것이 적절)

### **2. MemberController의 handleException() 메서드 수정**

```java
@ExceptionHandler
public ResponseEntity handleException(MethodArgumentNotValidException e) {
   final List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();
      List<ErrorResponse.FieldError> errors =
		         fieldErrors.stream()
                      .map(error -> new ErrorResponse.FieldError(
                          error.getField(),
                          error.getRejectedValue(),
	                        error.getDefaultMessage()))
                            .collect(Collectors.toList());

	    return new ResponseEntity<>(new ErrorResponse(errors), HttpStatus.BAD_REQUEST);
   }
}
```

- 왜 new ErrorResponse(errors) 인가? 모르겟음 왜 errors가 아니라
- `ErrorResponse.FieldError` 클래스에 담아서 List로 변환 후,  `List<ErrorResponse.FieldError>`를 `ResponseEntity` 클래스에 실어서 전달
- streamd이 아닌 반복문 사용해도 됨.
  
    ```java
    List<CustomError> errors = new ArrayList<>();
    for(FieldError : fieldErrors){
    	customerError currentError = new CustomError(
    					error.getField(),
    					error.getRejectedValue(),
    					error.getDefaultMessage()
    					);
    		errors.add(currentError)
    	}
    ```
    

### 3. @ExceptionHandler의 단점

1. controller는 controller 역할(요청과 응답을 받는!)만 해야함 - 계층이 해야할 역할을 벗어남
2. **각 Controller 클래스마다 코드 중복이 발생**
3. 하나의 예외만 있는 것이 아니기 때문에 **하나의 Controller 클래스 내에서 `@ExceptionHandler`를 추가한 에러 처리 핸들러 메서드가 증가하게 됨.**

## @RestCotrollerAdvice

@RestControllerAdvice - RestController에서 발생한 예외를 처리

- exception는 공통관심사이기 때문에 member-order-coffee 모두에게 적용됨.
- `@RestControllerAdvice` 애너테이션을 추가한 클래스를 이용하면 **예외 처리를 공통화할 수 있음**
- 최상위에 advice 패키지를 만들고 controller에 있던 exception을 advice의 클래스 생성후 옮기면 order, coffee, member 모두 예외 처리가 잘됨.

### 1. MemberController의 handleException() 메서드 로직 삭제

### **2.  최상위 패키지에 advice 패키지 생성 → ExceptionAdvice 클래스 정의**

```java
package com.springboot.advice;

import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionAdvice {

}
```

### **3. ErrorResponse 수정** - GlobalExceptionAdvice 클래스에서 처리할 Exception 핸들러 메서드를 구현

```java
package com.springboot.response

import lombok.Getter;
import org.springframework.validation.BindingResult;

import javax.validation.ConstraintViolation;
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

@Getter
public class ErrorResponse {
    private List<FieldError> fieldErrors; // (1)
    private List<ConstraintViolationError> violationErrors;  // (2)

		// (3) 생성자 
    private ErrorResponse(List<FieldError> fieldErrors, List<ConstraintViolationError> violationErrors) {
        this.fieldErrors = fieldErrors;
        this.violationErrors = violationErrors;
    }

		// (4) BindingResult에 대한 ErrorResponse 객체 생성
    public static ErrorResponse of(BindingResult bindingResult) {
        return new ErrorResponse(FieldError.of(bindingResult), null);
    }

		// (5) Set<ConstraintViolation<?>> 객체에 대한 ErrorResponse 객체 생성
    public static ErrorResponse of(Set<ConstraintViolation<?>> violations) {
    
        return new ErrorResponse(null, ConstraintViolationError.of(violations));
    }

		// (6) Field Error 가공
    @Getter
    public static class FieldError {
        private String field;
        private Object rejectedValue;
        private String reason;

				private FieldError(String field, Object rejectedValue, String reason) {
            this.field = field;
            this.rejectedValue = rejectedValue;
            this.reason = reason;
        }

        public static List<FieldError> of(BindingResult bindingResult) {
            final List<org.springframework.validation.FieldError> fieldErrors =
                                                        bindingResult.getFieldErrors();
            return fieldErrors.stream()
                    .map(error -> new FieldError(
                            error.getField(),
                            error.getRejectedValue() == null ?
                                            "" : error.getRejectedValue().toString(),
                            error.getDefaultMessage()))
                    .collect(Collectors.toList());
        }
    }

		// (7) ConstraintViolation Error 가공
    @Getter
    public static class ConstraintViolationError {
        private String propertyPath;
        private Object rejectedValue;
        private String reason;

				private ConstraintViolationError(String propertyPath, Object rejectedValue,
                                   String reason) {
            this.propertyPath = propertyPath;
            this.rejectedValue = rejectedValue;
            this.reason = reason;
        }

        public static List<ConstraintViolationError> of(
                Set<ConstraintViolation<?>> constraintViolations) {
            return constraintViolations.stream()
                    .map(constraintViolation -> new ConstraintViolationError(
                            constraintViolation.getPropertyPath().toString(),
                            constraintViolation.getInvalidValue().toString(),
                            constraintViolation.getMessage()
                    )).collect(Collectors.toList());
        }
    }
}

```

코드도 길고 너무 어려워서 당황했는데 이거 2시간 넘게 보고 정리함..!

1. 우선 ErrorResponse는 Exception을 관리하는 클래스라고 볼 수 있고 더 등록할 예외가 있으면 여기에  추가해야함.
2. (1) 과 (2)의 인스턴스변수는 각각 static Member class(FieldError와 ConstraintViolationError) 객체의 List를 참조.
    - (1) 은 `MethodArgumentNotValidException`으로부터 발생하는 에러 정보를 담는 멤버 변수, 즉 DTO 멤버 변수 필드의 유효성 검증 실패로 발생한 에러정보를 담음.
    - (2)는 `ConstraintViolationException`으로부터 발생하는 에러 정보를 담는 멤버 변수이고 URI 변수 값의 유효성 검증에 실패로 발생한 에러정보를 담음
3. (3)은 ErrorResponse의 생성자인데 private가 붙어 다른 곳에서는 호출이 불가능하고 해당 클래스 내에서만 사용 가능 (=캡슐화), 외부에서 new로 객체 생성할 수 없음
4. (4)와 (5)는 메서드 오버로딩에 해당됨 (메서드 오버로딩은 메서드명이 같고 매개변수의 타입이 다르다)
5. (4)에서 of()는 메서드 이름이며 ErrorResponse의 객체를 생성함.,  bindingResult 가 들어오면 (3)을 실행하고 ErrorResponse로 반환됨.
6. BindingResult 는 스프링에서 제공하는 유효성검사 정보를 담는 객체이다. BindingResult 객체를 가지고 에러 정보를 추출하고 가공하는 일은 ErrorResponse 클래스의 static 멤버 클래스인 **FieldError 클래스에게 위임**
7. (5)에서는 of메서드에  Set<ConstraintViolation<?>>violations 가 매개변수로 들어가면 (3)을 실행하고 ErrorResponse로 반환됨, 
8.  ConstrainViolation<?>violations의 ?는 제네릭의 와일드카드로 다양한 타입에 대한 유효성검사를 진행하기 위함.
9. Set을 사용한 이유는 중복방지와 빠른 조회가 가능하기 때문, Set<ConstraintViolation<?>> 객체를 가지고 에러 정보를 추출하고 가공하는 일은 **ConstraintViolationError 클래스에게 위임**
10. (6)의 FieldError 클래스는  ErrorReponse의 static Member 클래스에 해당하고
11. FieldError의 of메서드는 BindingResult에서 필드 에러들을 추출하여 FieldError객체로 변환하고 List로 반환
12. (7)의 ConstraintViolationError는 ErrorReponse의 static Member 클래스에 해당하고 
13. ConstraintViolationError의 of메서드는 constraintViolation(유효성검사에서 발생한 위반사항을 모아둔 집합)을 ConstraintViolationError객체로 변환하고 리스트로 반환함.

> ConstrainViolation, BindingResult, FieldError 뭐가 다른지 잘 이해가 안가서 비교해봄.  
전부 유효성 검사 결과를 담는 객체인 것 같은데….다른 역할을 한다고 함.
> 

|  | ConstrainViolation | BindingResult | FieldError |
| --- | --- | --- | --- |
| 역할 | Bean Validation API에서 제약 조건을 위반했을 때 발생하는 객체. | Spring MVC에서 폼 데이터의 유효성 검사 결과를 담는 객체. | BindingResult 객체 내에서 특정 필드의 유효성 검사 오류를 나타내는 객체. |
| 사용 | 주로 Java Bean Validation (javax.validation)에서 사용. | Spring MVC 컨트롤러에서 @Valid와 함께 사용 | BindingResult 내부에서 필드 별 유효성 검사 오류를 관리. |
| 주요정보 | 유효성 검사가 실패한 객체 (getRootBean()). | 유효성 검사 결과의 오류 목록 (getFieldErrors()). | 오류가 발생한 필드 이름 (getField()). |
|  | 유효성 검사가 실패한 값 (getInvalidValue()). | 특정 필드의 오류 여부 (hasFieldErrors()). | 오류 메시지 (getDefaultMessage()). |
|  | 위반된 제약 조건의 메시지 (getMessage()). | 전체 오류 여부 (hasErrors()). | 거부된 값 (getRejectedValue()). |
|  | 유효성 검사가 실패한 속성 경로 (getPropertyPath()). |  |  |

### 4. **Exception 핸들러 메서드 수정(**GlobalExceptionAdvice)

```java
package com.springboot.advice;

import com.springboot.response.ErrorResponse;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import javax.validation.ConstraintViolationException;

@RestControllerAdvice
public class GlobalExceptionAdvice {
    @ExceptionHandler
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleMethodArgumentNotValidException(
            MethodArgumentNotValidException e) {
        final ErrorResponse response = ErrorResponse.of(e.getBindingResult());

        return response;
    }

    @ExceptionHandler
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolationException(
            ConstraintViolationException e) {
        final ErrorResponse response = ErrorResponse.of(e.getConstraintViolations());

        return response;
    }
}

```
<br/>
<br/>

## Comment

exception부터 어렵다고해서 <br/>
코드 미리 보고 갔는데 대충보고 별거없는 것 같은데<br/>
하고 다른 거 하고 있었는데 너무 대충봤나보다....!<br/>
사실 다른데서 try-catch쓰길래 그냥 예전에 했던거네 했는데<br/>
더 심화된 내용이 나올줄이야..<br/>
이렇게 어려울줄은 몰랐는데 of나와서 한번 당황하고 생성자에 private..?<br/>
그래 그건 오케이 <br/>
근데 BindingResult..? ConstraintVoilation? 이것도 다 찾아봐야하고<br/>
클래스 안에 이너 클래스가 몇개나 있는지. 근데 이건 또 이너클래스라기보다 <br/>
reponse클래스의 static Member class 라고 하네!<br/>
일단 정리는 했는데 반만 이해간다. 이해가 가면 뭐하나 코드를 외워야 하나???싶음..<br/>
자주보면 쓸 수 있겠지! ㅎㅎㅎ,,,,,,<br/>

<br/>
<br/>
<br/>
<br/>
<br/>
        
<div class="notice" markdown="1">
🍒 `공지` 
<h4> - <u>정보 공유가 아닌 개인이 공부하고 기록하기 복습하기 위한 용도입니다.</u></h4>
</div>
