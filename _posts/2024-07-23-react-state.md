---
layout : single
title : "[react] React 흐름 - 상태끌어올리기 실습"
categories: react
tag : [Project, react, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


- 상태 끌어올리기, 데이터 흐름 개념을 활용하여 항공편 검색 기능을 구현
    
    <img src="https://github.com/user-attachments/assets/20534576-75ef-4fd4-8abd-1bf6183e533e" width=500/>
    
    검색기능을 구현해야 하는 컴포넌트는 주황색 박스이다.
    
- 항공권 검색 사이트의 구조
    
    ```java
    ├── pages
    │   ├── Main.js # 첫 화면 컴포넌트, 필터링 상태를 담고 있습니다.
    │   ├── component
    │         ├── Flight.js           # 단일 항공편
    │         ├── FlightList.js       # 항공편 목록
    │         ├── LoadingIndicator.js # 로딩 컴포넌트
    │         └── Search.js  
    ```
    
    1. Main에서 항공권리스트를 가지고 있어야 한다
        - react에서 데이터는 단방향으로 흐르기 때문에 (부모에서 자식으로만) 부모가 데이터를 가지고 있고 상태끌어올리기를 통해 상태를 관리하고, 상태변경하는 함수를 자식컴포넌트에게 전달하여 자식의 상태를 변경할 수 있게 된다.
    2. Main이 가지고 있는 FlightList는 state에 해당하지만 자식인 FlightList의 입장에서는 props에 해당한다
        - 전달인자로 전달된 값은 모두 props에 해당한다고 생각하면 된다.

### Main.js

```java
  const [condition, setCondition] = useState({
    departure: 'ICN',
  });
  
  const [flightList, setFlightList] = useState([]);
```

- 여기서 useState의 반환 값은 무조건 요소가 2개 들어있는 배열에 해당하고 , useState의 매개변수는 초기값에 해당한다.) → useState는 함수에 해당한다고 볼 수 있다.
- useState의 값이 변경되면 화면이 렌더링 된다. 결국 setCondition에 의해 새롭게 렌더링 된 화면으로 변경된다.
- flightlist의 초기값은 json인데 데이터 베이스를 사용하지 않기 때문에 더미데이터를 import로 넣어 준 것

```java
import Head from 'next/head';
import { useEffect, useState } from 'react';
import { getFlight } from '../api/FlightDataApi';
import FlightList from './component/FlightList';
import LoadingIndicator from './component/LoadingIndicator';
import Search from './component/Search';
import json from '../resource/flightList';

export default function Main() {
  // 항공편 검색 조건을 담고 있는 상태
  const [condition, setCondition] = useState({
    departure: 'ICN',
  });
  const [flightList, setFlightList] = useState(json);

  // 주어진 검색 키워드에 따라 condition 상태를 변경시켜주는 함수
  const search = ({ departure, destination }) => {
    if (
      condition.departure !== departure ||
      condition.destination !== destination
    ) {
      console.log('condition 상태를 변경시킵니다');
      
      // TODO:
      setCondition({departure, destination});
      
    }
  };

  // TODO: 테스트 케이스의 지시에 따라 search 함수를 Search 컴포넌트로 내려주세요.
  return (
    <div>
      <Head>
        <title>States Airline</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main>
        <h1>여행가고 싶을 땐, States Airline</h1>
        
        //추가
        <Search onSearch = {search}/>
            
        <div className="table">
          <div className="row-header">
            <div className="col">출발</div>
            <div className="col">도착</div>
            <div className="col">출발 시각</div>
            <div className="col">도착 시각</div>
            <div className="col"></div>
          </div>
          <FlightList list={flightList.filter(filterByCondition)} />
        </div>

        <div className="debug-area">
          <Debug condition={condition} />
        </div>
        <img id="logo" alt="logo" src="codestates-logo.png" />
      </main>
    </div>
  );
}
```

### Search.js

<img src="https://github.com/user-attachments/assets/15482062-3220-45bb-92e5-b95ee391fab1" width=500/>

- 사진 상에서는 가려져 있는데 handleChange 함수 위에 아래 코드와 같이 useState가 선언되어있다.
    
    ```java
     const [textDestination, setTextDestination] = useState('');
    ```
    
- handleChange라는 이벤트가 (검색버튼 혹은 엔터를 누를 때) 발생하면 handleSearchClick 함수가 실행되고 검색창에 있는 내용은 useState가 실행되어 setTextDestination 이 진행되어 화면이 새롭게 렌더링된다.
- 여기서 좀 헷갈렸던 부분이 오랜만에 프론트를 실습해서 그런지 key를 문자열로 구현하지 않아서 검색 버튼이 눌려지지 않았고 테스트가 통과되지 않았다.
    - 자바스크립트에서 키는 무조건 문자열로 구현해야하고 문자열로 적지 않아도 알아서 바꿔지긴 하지만 그래도 알아야 한다.

📌 짚고 가기

- js의 객체에서 키값은 무조건 문자열이다.
- 만약에 { “a” : 1 , “b” : 333 , “c” : 13} 여기서 객체와 JSON의 차이는 ??
    - JSON은 에서 양 끝에 따옴표를 붙어야 한다. → 즉  “ { “a” : 1 , “b” : 333 , “c” : 13}”