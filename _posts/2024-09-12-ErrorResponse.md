---
layout : single
title : "[Project] ConstraintViolationError NPE 발생"
categories: Project
tag : [project2, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

---


> `null` 값이 들어오는 상황을 처리하기 위해 `ConstraintViolationError` 클래스에서 `null`을 `"null"` 문자열로 변환해 예외 메시지에 포함시키도록 코드 수정. 이를 통해 `NullPointerException` 발생을 방지하고, 의미 있는 예외 메시지를 제공할 수 있었다.
> 

### 예외 처리 시 `ConstraintViolationError` 수정하기: NullPointerException 해결 과정

프로젝트 진행 중, checkbox로 한번에 수정 작업을 처리하기 위해서 Dto를 수정하고 있었다.
 `PATCH` 요청으로 List로 보낼 때 데이터가 잘 수정되는지 확인하기 위해서 PATCH 요청을 보낸다는 것이 `POST` 요청을 보내는 실수를 했고, 그로 인해 예상치 못한 예외가 발생했다. 

 <img src = "https://github.com/user-attachments/assets/a29219f0-0fe3-4b90-a623-4fbbf694c3fc
 " witdh=500 />

안그래도 발생하는 상태코드 500번 에러는 다 잡고 싶어서 잘되었다 생각하고 디버깅을 진행한 결과, 당연히  `postDto` 에 필요한 필드가 없어서 발생한 문제 였다.

근데 당연히 dto는 @Valid 애너테이션으로 유효성 검증을 해서 여기서 예외를 잡아줄 것 같은데 왜 안잡아주지? 라고 생각이 들어서 설마 @Valid 나 @Validated가 없는 걸까 하고 생각하고 확인해봤는데
그 문제는 아니였다.

<img src = "https://github.com/user-attachments/assets/7207d73b-0c57-4a34-864c-910e0b24aa2e" witdh =500/>

예외 발생 시  생기는 메세지를 다시 읽어보니 `ConstraintViolation.getInvalidValue()` 이
 `null`로 들어와서 생긴 문제임을 확인했다. 

### 발생한 문제점: `NullPointerException`과 예외 처리

- 즉 `Exception` 클래스에서 예외를 처리하던 중 `Value`가 `null`로 들어오면서 발생한 것
- 문제는 `ConstraintViolation.getInvalidValue()`가 `null` 값을 반환하는 경우, 이를 처리하지 못해 `NullPointerException`이 발생하는 것이었다. 이때 발생하는 예외를 해결하려면, `null`이 들어오는 상황에서도 적절한 예외 메시지를 반환할 수 있도록 해야 한다.
- 다시 생각해보니 postDto도 List로 들어오기 때문에 기존에는 List가 아니었기 때문에 잡아줬던 예외를 변경되고 나서는 못잡는 것 같았다. 

### 문제 해결  : `ConstraintViolationError` 클래스 수정

우선 `ErrorResponse` 내의 `ConstraintViolationError` 클래스를 수정하여, `null`이 발생할 경우에도 안전하게 처리될 수 있도록 했다. 이를 위해 `getInvalidValue()`가 `null`일 때는 이를 문자열 `"null"`로 변환해 반환하도록 수정했다.

```java
@Getter
public static class ConstraintViolationError {
    private String propertyPath;
    private String rejectedValue;  // Use String to handle null gracefully
    private String reason;

    private ConstraintViolationError(String propertyPath, String rejectedValue, String reason) {
        this.propertyPath = propertyPath;
        this.rejectedValue = rejectedValue;
        this.reason = reason;
    }

    public static List<ConstraintViolationError> of(
            Set<ConstraintViolation<?>> constraintViolations) {
        return constraintViolations.stream()
                .map(constraintViolation -> {
                // 수정한 부분
                    String invalidValue = (constraintViolation.getInvalidValue() == null)
                            ? "null" // Handle null safely
                            : constraintViolation.getInvalidValue().toString(); // Convert null values to "null"
                            
                           //여기까지
                    return new ConstraintViolationError(
                            constraintViolation.getPropertyPath().toString(),
                            invalidValue,
                            constraintViolation.getMessage()
                    );
                }).collect(Collectors.toList());
    }
}
```

- `getInvalidValue()`에서 반환되는 값이 `null`일 경우, 이를 `"null"` 문자열로 처리했고 이로 인해 `NullPointerException`이 발생하지 않고 의미 있는 오류 메시지를 전달할 수 있었다.
- 이전에는 `null` 값으로 인해 예외 메시지가 제대로 전달되지 않았다면, 이제는 오류가 발생할 때 `"null"`이라는 문자열이 메시지로 전달되어 예외 상황을 명확히 알 수 있게 되었다.
- `null` 값이 반환되더라도 오류 메시지가 정상적으로 반환되기 때문에, 코드의 유연성과 안정성이 향상되었다. 예외 처리를 더 유연하게 할 수 있고, 예상치 못한 예외 상황에서도 쉽게 문제를 파악할 수 있다.
굿굿


### 해결

<img src = "https://github.com/user-attachments/assets/d6da0cf8-88da-4255-878b-0d239dc2fe31" width  = 500/>

이러한 변경을 통해 `ConstraintViolation.getInvalidValue()`에서 `null` 값이 반환될 때 발생하는 `NullPointerException`을 방지할 수 있었다. 예외 처리 로직을 수정함으로써, 더 안정적인 코드를 작성할 수 있게 되었으며, `null` 값 처리 시에도 의미 있는 오류 메시지를 반환할 수 있도록 개선했다.

이번 문제 해결 과정은 **유효성 검사에서 발생하는 예외 상황을 더 유연하게 처리하는 방법**에 대해 고민할 기회가 되었고, 이를 통해 실수를 줄이고 예외 상황에서 더 나은 메시지를 전달할 수 있는 구조를 구현할 수 있었다.