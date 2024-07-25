---
layout : single
title : "[react] Effect Hook"
categories: react
tag : [Project, react]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


React에서는 컴포넌트 내에서 fetch를 사용해 API 정보를 가져오거나 이벤트를 활용해 DOM 직접 조작할 때 Side Effect가 발생했다고 말한다.

### **Pure Function (순수 함수)**

- 순수 함수란, 오직 함수의 입력만이 함수의 결과에 영향을 주는 함수를 의미합니다
- 함수의 입력이 아닌 다른 값이 함수의 결과에 영향을 미치는 경우, 순수 함수가 아니다.
- 입력으로 전달된 값을 수정하지 않는다.
- 즉, 전달 인자가 주어질 경우, 항상 똑같은 값이 리턴됨을 보장 (고로 `Math.random()`은 순수 함수가 아니다)
- 순수 함수에는 네트워크 요청과 같은 Side Effect가 없다.

```jsx
function upper(str) {
  return str.toUpperCase(); // toUpperCase 메소드는 원본을 수정하지 않습니다 (Immutable)
}

upper('hello') // 'HELLO'
```

- 어떤 함수가 fetch API를 이용해 AJAX 요청을 한다고 가정했을 때 이 함수는 순수 함수가 아니다. 그 이유는?
    
    ⇒ 반환하는 값이 언제나 동일하지 않기 때문, 외부의 어떠한 값을 참조(호출)해야 해서 그 값이 달라지면 결과도 달라지게 되기 때문.
    

### **React의 함수 컴포넌트**

- React의 함수 컴포넌트는, props가 입력으로, JSX Element가 출력으로 나간다.
- 여기에는 그 어떤 Side Effect도 없으며, 순수 함수로 작동한다.

```jsx
function SingleTweet({ writer, body, createdAt }) {
  return <div>
    <div>{writer}</div>
    <div>{createdAt}</div>
    <div>{body}</div>
  </div>
  }
```

### **React 컴포넌트에서의 Side Effect**

- 타이머 사용 (setTimeout)
- 데이터 가져오기 (fetch API, localStorage)

하지만 보통 React 애플리케이션을 작성할 때에는, AJAX 요청이 필요하거나, LocalStorage 또는 타이머와 같은 React와 상관없는 API를 사용하는 경우가 발생할 수 있고 이는 React의 입장에서는 전부 Side Effect에 해당한다..

React는 Side Effect를 다루기 위한 Hook인 Effect Hook을 제공합니다.

## Effect Hook

- `useEffect`는 컴포넌트 내에서 Side effect를 실행할 수 있게 하는 Hook
- useEffect(함수, [종속성1, 종속성2, ...])
- `useEffect`의 **첫 번째 인자는 함수**, 해당 함수 내에서 side effect를 실행하면 된다.
- `useEffect`의 **두 번째 인자는 배열**,  배열은 조건을 담고 있다. 여기서 조건은 boolean 형태의 표현식이 아닌, **어떤 값의 변경이 일어날 때**를 의미한다. 따라서, 해당 배열엔 **어떤 값**의 목록이 들어갑니다. 이 배열을 특별히 종속성 배열이라고도 한다.
    - 빈 배열 넣기 `useEffect(함수, [])` : **컴포넌트가 처음 생성될 때만** effect 함수가 실행
    - 아무것도 넣지 않기(기본 형태) `useEffect(함수)` : 기본형태,
    

### 실행되는 시기

<img src="https://github.com/user-attachments/assets/c190ff55-ac85-4953-bc94-bab60bdd61ba" width=500 />

- 컴포넌트 생성 후 처음 화면에 렌더링(표시)
- 컴포넌트에 새로운 props가 전달되며 렌더링
- 컴포넌트에 상태(state)가 바뀌며 렌더링

이와 같이 매번 새롭게 컴포넌트가 **렌더링** 될 때 Effect Hook이 실행

### **Hook을 쓸 때 주의할 점**

- 최상위에서만 Hook을 호출합니다.
- React 함수 내에서 Hook을 호출합니다.
    
    [https://ko.legacy.reactjs.org/docs/hooks-rules.html#only-call-hooks-at-the-top-level](https://ko.legacy.reactjs.org/docs/hooks-rules.html#only-call-hooks-at-the-top-level)
    

### 컴포넌트 내에서  **AJAX 요청**

목록 내 필터링을 구현하기 위해서는 다음과 같은 두 가지 접근이 있을 수 있다.

1. **컴포넌트 내에서 필터링**: 전체 목록 데이터를 불러오고, 목록을 검색어로 filter 하는 방법
2. **컴포넌트 외부에서 필터링**: 컴포넌트 외부로 API 요청을 할 때, 필터링한 결과를 받아오는 방법 (보통, 서버에 매번 검색어와 함께 요청하는 경우가 이에 해당한다.)
    
    
    |  | 장점 | 단점 |
    | --- | --- | --- |
    | 컴포넌트 내부에서 처리 | HTTP 요청의 빈도를 줄일 수 있다 | 브라우저(클라이언트)의 메모리 상에 많은 데이터를 갖게 되므로, 클라이언트의 부담이 늘어난다 |
    | 컴포넌트 외부에서 처리 | 클라이언트가 필터링 구현을 생각하지 않아도 된다 | 빈번한 HTTP 요청이 일어나게 되며, 서버가 필터링을 처리하므로 서버가 부담을 가져간다 |

### AJAX 요청 보내기

임의로 구현한 storageUtil.js 대신, fetch API를 써서 서버에 요청할때,

명언을 제공하는 API의 엔드포인트가 http://서버주소/proverbs라고 가정하면  코드는 아래와 같다.

```jsx
useEffect(() => {
  fetch(`http://서버주소/proverbs?q=${filter}`)
    .then(resp => resp.json())
    .then(result => {
      setProverbs(result);
    });
}, [filter]);
```

### Loading indicator의 구현

```jsx
const [isLoading, setIsLoading] = useState(true);

// 생략, LoadingIndicator 컴포넌트는 별도로 구현했음을 가정합니다
return {isLoading ? <LoadingIndicator /> : <div>로딩 완료 화면</div>}
```

fetch 요청의 전후로 `setIsLoading`을 설정해 주어 보다 나은 UX를 구현할 수 있다.

```jsx
useEffect(() => {
  setIsLoading(true);
  fetch(`http://서버주소/proverbs?q=${filter}`)
    .then(resp => resp.json())
    .then(result => {
      setProverbs(result);
      setIsLoading(false);
    });
}, [filter]);
```