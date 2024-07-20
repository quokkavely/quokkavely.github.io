---
layout : single
title : "[JS] Calculator 만들기"
categories: javascript
tag : [js,개념정리]
toc : true
toc_sticky : true
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}




# 계산기 구현

DOM도 배웠으니
이전 Post에서 적용했던 계산기 디자인에 실제로 계산 기능을 추가해보기로 했다.<br/>
아래 링크는 CSS 배우면서 만들었던 계산기 디자인만 구현한 것이다. <br/>
https://quokkavely.github.io/html-css/css-02/

## 구현 전에 이해하기 (DOM, JS)

```jsx
const calculator = document.querySelector('.calculator'); 
const buttons = calculator.querySelector('.calculator__buttons'); 

const firstOperend = document.querySelector('.calculator__operend--left'); 
const operator = document.querySelector('.calculator__operator'); 
const secondOperend = document.querySelector('.calculator__operend--right'); 
const calculatedResult = document.querySelector('.calculator__result'); 
```

1. **document.querySelector : html에 해당하는 클래스의 요소를 상수로 가져옴.**
    - const calculator는 .calculator가 가지고 있는 요소를 가져옴 (html의 calculator 클래스 )
    - buttons는 claculator 클래스의 .calculator_buttons클래스의 요소를 가져오는 것
    - firstOperand = .calculator__operend--left 클래스의 요소를 가져옴
    - operator =.calculator__operator 클래스의 요소
    - secondOperend= .calculator__operend--right 클래스에서 가져온 요소
    - calculatedResult =.calculator__result 클래스에서 가져온 요소
2. **나머지는 document.querySelector인데 buttons만 calculators.queryselector로 가져오는 이유?**
    1. 페이지 내에 동일한 buttons가 있거나 (페이지 전체에서 유일한 요소가 아닐경우)
    2. buttons 는 calculator 내부에 있으므로 calculator.querySelector를 사용하여 검색하는 것이 더 효율적이고 명확 ⇒ 코드 가독성을 올릴 수 있고 유지보수에도 용이
    3. 문서 전체(Document)가 아닌 특정 컨테이너에서만 검색하기 때문에 불필요한 검색을 줄일 수 있음 
3. **document는 무엇인지??? =현재 웹페이지를 나타내는 객체**
    - **`document`**는 웹 브라우저의 Document Object Model (DOM) 인터페이스를 나타내는 전역 객체
    - 현재 로드된 웹 페이지의 전체 구조와 내용을 나타내며, JavaScript에서 HTML 및 CSS와 상호작용하기 위해 사용된다
    - **`document`** 객체는 웹 페이지의 루트 요소로, 페이지의 콘텐츠에 접근하고 조작할 수 있는 다양한 메서드와 속성을 제공한다.
    

### EventListener

```jsx
buttons.addEventListener('click', function (event) {
  
  const target = event.target; 
  const action = target.classList[0]; 
  const buttonContent = target.textContent;
```

- .calculator_buttons클래스의 요소를 가져온 buttons를  클릭할 경우 함수가 실행됨.
- **event.target**은 클릭된 HTML 요소를 참조 ,
- target은 이벤트가 발생한 DOM 요소를 가리킴.
- **const action = target.classList[0];**
- **target.classList**는 클릭된 버튼의 클래스 리스트(DOMTokenList)를 반환, **`[0]`**은 첫 번째 클래스
- target.textcontent는 클릭된 버튼의 텍스트 컨텐츠를 반환
- **`buttonContent`**는 버튼에 표시된 텍스트 값을 저장한다 이를 통해 버튼에 어떤 값(예: '1', '+', '=')이 있는지 알 수 있다.

## 계산기 로직 구현하기

```jsx
const calculator =document.querySelector('.calculator');
const buttons = calculator.querySelector('.calculator__buttons');

    let firstNum, operatorForAdvanced, previousKey, previousNum;
    const display=document.querySelector(#print);

    buttons.addEventListener('click', funtion(event){
    const target = event.target;
    const action = target.classList[0];
    const buttonContent=target.textContent;
});

```

1. 작성 시 event 에서 오류남
    
    <img src="https://github.com/user-attachments/assets/809645cf-cb4c-4f42-8747-38d302392f27"/>
    
    1. fuction 오타, function 계속 c 나 n빼먹음.ㅠㅠㅠ
    2. 쿼리문 문자로 입력 querySelector (’#print’)로 작성해야됨.)
    
2. previous key 오류
    1.  operator 클릭할때 previouskey가 operator면 들어오지 못하게 막아 둔곳에 들어와서 js는 else if가 없었나 갑자기 의문이 들 정도였다. 그래서 콘솔로 다 출력되게 찍었더니 전에 number를 입력하고 operator를 클릭해도 previousKey가 operator로 인식되어서 그런거였고 왜 그런지 한참을 모르다가 위를 올려봤더니…내가 전에 previousKey를 사용했던게 문제였다. 
    2. 나는 심화를 구현할 것이라 윗 부분은 생각 안하고 있었는데 한참 걸렸지만 그래도 알아서 다행이지뭐… 했다..
3. 연산자 문제
    1. 처음에 숫자받고 (숫자 후 operator 누르기 전까지) 
    2. 그 다음은 계속 더해준 숫자는 다 previousNum으로 만들고 싶었는데
4. clear 및 undefined  에러
    1. 초기화 할 때 처음에 0으로 다 변경했음..근데 바로 이상하게 출력되서 아,,하고 바로 undefined로 변경, 근데 이상하게 undefined로 했더니 문자열로 인식해버림
    2. 근데 또 previousNum!==undefined이면 못 들어오게 막았는데 들어오길래  typeof는 문자열로 타입 입력해서 헷갈렸는데 따옴표 없애니 금방 해결됨

### 화면에 출력 안되는 문제

1. 계산기에 화면이 출력되지 않아서 콘솔 로그를 남겨서 확인해봤다.
    
    <img src="https://github.com/user-attachments/assets/668245d8-3cc2-405d-b46e-fd6792ab6f86" width=500/>
    
2. 로그 남겼는데 개발자 창에서는 로그가 다 나온다
    
    <img src="https://github.com/user-attachments/assets/f44ce825-b758-450a-8e25-b0d48f87a1fe" width=500 />
    

1. 여기가 아마 문제인 것 같다. input type과 placeholder 지움
    - 변경 전
        
      <img src="https://github.com/user-attachments/assets/68c35321-49ec-4af5-816b-1173e7e35037" width=500 />
        
    - 변경 후
        
      <img src="https://github.com/user-attachments/assets/f1e7933a-e84b-45e1-9053-f2c3ba30a3aa" width=500/>
        
    - 출력 잘 됨. 완성
        
      <img src="https://github.com/user-attachments/assets/9c9790de-ecdf-489c-a2ec-17b4a5aeb3e8" width=200/>
        

### 완성작

https://github.com/user-attachments/assets/678ddf9f-256a-4a3c-946e-5343f1767ed9

