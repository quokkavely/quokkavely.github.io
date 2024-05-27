# [JS] ES6 신규문법 (spread/rest, 구조분해, 화살표함수)

태그: js
날짜: 2024/05/23
목표량: No
복습: No
소목표 DB: 프론트 [html , css , js] (https://www.notion.so/html-css-js-f707bd4d7be549b5aca0b62caff4cfbb?pvs=21)
최종 목표: https://www.notion.so/b7df7eb5912743c5bef6b0851dfc0c4c

*arguments와 rest parameter를 통해 배열로 된 전달인자(args)의 차이를 확인하시기 바랍니다.*

## **spread/rest 문법**

### spread 문법

주로 배열을 풀어서 인자로 전달하거나, 배열을 풀어서 각각의 요소로 넣을 때에 사용

```jsx
function sum(x, y, z) {
  return x + y + z;
}

const numbers = [1, 2, 3];
sum(...numbers);
```

1. …numbers : spread문법으로 배열 numbers를 풀어서 매개인자로 넣어줌
2. 결과는 6

![Untitled](%5BJS%5D%20ES6%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%B2%E1%84%86%E1%85%AE%E1%86%AB%E1%84%87%E1%85%A5%E1%86%B8%20(spread%20rest,%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%E1%84%87%E1%85%AE%E1%86%AB%E1%84%92%E1%85%A2,%20%E1%84%92%E1%85%AA%E1%84%89%E1%85%A1%2098fe8bf1d51244c3bafb0287ca8e038a/Untitled.png)

매개인자를 1개만 넣으면 나머지는 undefined로 출력됨

### rest 문법

파라미터를 배열의 형태로 받아서 사용 가능. 파라미터 개수가 가변적일 때 유용

```jsx
function sum(...theArgs) {
  let result = 0;
	for(let idx = 0; idx < theArgs.length; idx++) {
		result = result + theArgs[idx];
	}
	return result;
}

sum(1,2,3)
sum(1,2,3,4)
```

1. sum(1,2,3) → …theArgs 는 매개인자를 배열로 받아서 ( [1,2,3] ) 사용하게 됨
2. 결과는 6, 10

Rest는 남는 것만 가져오는 것임

따라서 다음과 같은 문법은 사용할 수 없습니다. 

const [first, ...middle, last] = array //**할당하기 전 왼쪽에는, rest 문법 이후에 쉼표가 올 수 없습니다**

### spread 문법 응용 ✨✨✨

spread 문법은 기존 배열을 변경하지 않으므로(immutable)

[전개 구문 - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax)

[나머지 매개변수 - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/rest_parameters)

### 배열

1. 배열 합치기
    
    ```jsx
    let parts = ['shoulders', 'knees'];
    let lyrics = ['head', ...parts, 'and', 'toes'];
    
    console.log(lyrics);
    // ['head', 'shoulders', 'knees', 'and', 'toes']
    ```
    
    ```jsx
    let arr1 = [0, 1, 2];
    let arr2 = [3, 4, 5];
    arr1 = [...arr1, ...arr2];  
    // 참고: spread 문법은 기존 배열을 변경하지 않으므로(immutable), 
    //arr1의 값을 바꾸려면 새롭게 할당해야 합니다.
    
    // 질문: arr1의 값은 무엇인가요?
    [0,1,2,3,4,5]
    ```
    
2. 배열 복사
    
    ```jsx
    let arr = [1, 2, 3];
    let arr2 = [...arr]; // arr.slice() 와 유사
    arr2.push(4); 
     // 참고: spread 문법은 기존 배열을 변경하지 않으므로(immutable),
     // arr2를 수정한다고, arr이 바뀌지 않습니다.
    
    // 질문: arr와 arr2의 값은 각각 무엇인가요?
    ```
    
    ![Untitled](%5BJS%5D%20ES6%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%B2%E1%84%86%E1%85%AE%E1%86%AB%E1%84%87%E1%85%A5%E1%86%B8%20(spread%20rest,%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%E1%84%87%E1%85%AE%E1%86%AB%E1%84%92%E1%85%A2,%20%E1%84%92%E1%85%AA%E1%84%89%E1%85%A1%2098fe8bf1d51244c3bafb0287ca8e038a/Untitled%201.png)
    
    원본안바뀌고 딥카피가 됨.
    
    but, 다차원 배열 복사시 내부의 배열은 얕은복사가 됨
    
    아래는 보완할 수 있는 방법
    
    다차원 배열을 풀어서 넣
    
    ![Untitled](%5BJS%5D%20ES6%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%B2%E1%84%86%E1%85%AE%E1%86%AB%E1%84%87%E1%85%A5%E1%86%B8%20(spread%20rest,%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%E1%84%87%E1%85%AE%E1%86%AB%E1%84%92%E1%85%A2,%20%E1%84%92%E1%85%AA%E1%84%89%E1%85%A1%2098fe8bf1d51244c3bafb0287ca8e038a/Untitled%202.png)
    
3. 문제
    
    ```jsx
    const array = ['java', 'spring', 'im', 'course']
        const [start, ...rest] = array
        console.log(start) //'java'
        console.log(rest)//['spring', 'im', 'course']
        console.log(array) //[ 'java', 'spring', 'im', 'course' ]
    ```
    

### 객체

1. 객체에서 사용
    
    ```jsx
    let obj1 = { foo: 'bar', x: 42 };
    let obj2 = { foo: 'baz', y: 13 };
    
    let clonedObj = { ...obj1 };
    let mergedObj = { ...obj2, ...obj1 };
    ```
    
    ![Untitled](%5BJS%5D%20ES6%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%B2%E1%84%86%E1%85%AE%E1%86%AB%E1%84%87%E1%85%A5%E1%86%B8%20(spread%20rest,%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%E1%84%87%E1%85%AE%E1%86%AB%E1%84%92%E1%85%A2,%20%E1%84%92%E1%85%AA%E1%84%89%E1%85%A1%2098fe8bf1d51244c3bafb0287ca8e038a/Untitled%203.png)
    
    기능 비슷하게 구현
    
    ---
    
    ```jsx
    let obj1 = { foo: 'bar', x: 42 };
    let obj2 = { foo: 'baz', y: 13 };
    
    let tempObj={};
    concatObj(tempObj,obj1);
    concatObj(tempObj,obj2);
    
    console.log(tempObj);
    function concatObj(obj1,obj2){
        let arr =Object.keys(obj2);
        for(let i =0 ;i<arr.length;i++){
            obj1[arr[i]]=obj2[arr[i]];
        }
        return obj1;
    }
    ```
    
    [출력결과]
    
    ```jsx
    > concatObj(tempObj,obj1);
    { foo: 'bar', x: 42 }
    > concatObj(tempObj,obj2);
    { foo: 'baz', x: 42, y: 13 }
    ```
    
2. 함수에서 나머지 파라미터 받아오기
    
    ```jsx
    function myFun(a, b, ...manyMoreArgs) {
      console.log("a", a);
      console.log("b", b);
      console.log("manyMoreArgs", manyMoreArgs);
    }
    
    myFun("one", "two", "three", "four", "five", "six");
    
    // 질문: 콘솔은 순서대로 어떻게 찍힐까요?
    
    a one
    b two
    manyMoreArgs [ 'three', 'four', 'five', 'six' ]
    ```
    

### 구조분해 destructing

### **분해 후 새 변수에 할당**

1. 배열✨✨✨
    1. 예제 1
        
        ```jsx
        const [a, b, ...rest] = [10, 20, 30, 40, 50];
        
        // 질문: a, b, rest는 각각 어떤 값인가요?
        console.log(a); // 10
        console.log(b); // 20
        console.log(rest); // [30, 40, 50]
        ```
        
    2. 예제 2
        
        ```jsx
        const foo = ["one", "two", "three"];
        
        const [red, yellow] = foo;
        console.log(red); // "one"
        console.log(yellow); // "two"
        console.log(green); // "three"
        console.log([red, yellow]);// ["one", "two"] 
        
        ```
        
    
2. 객체
    
    ```jsx
    const {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40}
    // 질문: a, b, rest는 각각 어떤 값인가요?
    console.log(a); // 10
    console.log(b); // 20
    console.log(rest); // {c: 30, d: 40}
    ```
    
    1. a와 b뒤에는 잘라서 객체로 받음
    2. 객체에서 구조 분해 할당을 사용하는 경우, 선언(`const`, `let`, `var`)과 함께 사용하지 않으면 에러가 발생 할 수 있음
3. 유용한 여제
    1. 함수에서 객체 분해 
        
        ```jsx
        function whois({displayName: displayName, fullName: {firstName: name}}){
          console.log(displayName + " is " + name);
        }
        
        let user = {
          id: 42,
          displayName: "jdoe",
          fullName: {
              firstName: "John",
              lastName: "Doe"
          }
        };
        
        whois(user) // 질문: 콘솔에서 어떻게 출력될까요?
        ```
        
        ![Untitled](%5BJS%5D%20ES6%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%B2%E1%84%86%E1%85%AE%E1%86%AB%E1%84%87%E1%85%A5%E1%86%B8%20(spread%20rest,%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%E1%84%87%E1%85%AE%E1%86%AB%E1%84%92%E1%85%A2,%20%E1%84%92%E1%85%AA%E1%84%89%E1%85%A1%2098fe8bf1d51244c3bafb0287ca8e038a/Untitled%204.png)
        
    
    문제 1. 
    
    ```jsx
    const student = { name: '박해커', major: '물리학과' }
     const { name } = student
     
     console.log(name) // '박해커'  
       
    ```
    
    문제 2. **객체 분해 및 덮어쓰기 : 객체는 나중에 들어오는 속성이 같은 키를 가지면 덮어쓴다.**
    
    ```jsx
    const user = {
          name: '김코딩',
          company: {
            name: 'Javascript',
            department: 'Development',
            role: {
              name: 'Software Engineer'
            }
          },
        }
    
        const changedUser = {
          ...user,
          name: '박해커',
          age: 20
        }
    
        const overwriteChanges = {
          name: '박해커',
          age: 20,
          ...user
        }
    
        const changedDepartment = {
          ...user,
          company: {
            ...user.company,
            department: 'Marketing'
          }
        }
    ```
    
    1. **changedUser**
        
        ```jsx
         {name: '박해커',**company: {name: 'Javascript',department: 'Development',role:{ name: 'Software Engineer'}}**,age: 20}
        ```
        
        1. **왜 …user가 먼저 나오는데 name:박해커 와 age:20 사이에 들어오는지?**
            
            
            1. **`...user`**는 **`user`** 객체의 모든 속성을 복사
                
                ```jsx
                {name: '김코딩',
                  company: {
                    name: 'Javascript',
                    department: 'Development',
                    role: {
                      name: 'Software Engineer'
                    }
                  }
                }
                ```
                
            
             그 다음, **`name: '박해커'`** 속성이 추가됨. 기존의 **`name`** 속성이 '김코딩'이었지만, 이제 '박해커'로 덮어씀
            
            ```jsx
            {name: '박해커',
              company: {
                name: 'Javascript',
                department: 'Development',
                role: {
                  name: 'Software Engineer'
                }
              }
            }
            ```
            
            마지막으로, 새로운 **`age: 20`** 속성이 추가됩니다
            
            ```jsx
            {name: '박해커',
              company: {
                name: 'Javascript',
                department: 'Development',
                role: {
                  name: 'Software Engineer'
                }
              },
              age: 20
            }
            ```
            
    2. **overwriteChanges**
        
        ```jsx
        {name: '김코딩',age: 20, company: {name: 'Javascript',department: 'Development', role: { name: 'Software Engineer' }}}
        ```
        
    3. **changedDepartment**
        
        ```jsx
        {name: '김코딩', company:{name: 'Javascript', department: 'Marketing', role: {name: 'Software Engineer'}},}
        ```
        
    

### 화살표함수(**arrow function)**

```jsx
// 함수선언문
function sum (x, y) {
	return x + y;
}

// 함수표현식
const subtract = function (x, y) {
	return x - y;
}
```

화살표함수는 fuction이 빠지고 화살표로 대체

```jsx
// 화살표 함수
const multiply = (x, y) => {
	return x * y;
}
```

자바에서 람다는 →

js는 ⇒

1. 매개변수가 한 개일 때 소괄호 생략가능
    
    ```jsx
    // 매개변수가 한 개일 때, 소괄호를 생략할 수 있습니다.
    const square = x => { return x * x }
    
    // 위 코드와 동일하게 동작합니다.
    const square = ( x ) => { return x * x }
    
    // 단, 매개변수가 없는 경우엔 소괄호를 생략할 수 없습니다.> 함수임을 나타내야해
    const greeting = () => { return 'hello world' }
    ```
    
2. 함수 코드 블록 내부가 하나의 문으로 구성되어 있다면 중괄호(`{}`)를 생략할 수 있다. 이때 코드 블록 내부의 문이 값으로 평가될 수 있으면 `return` 키워드를 생략할 수 있다**.**
    
    ```jsx
    const squre = x => x * x
    
    // 위 코드와 동일하게 동작합니다.
    const square = x => { return x * x }
    
    // 위 코드와 동일하게 동작합니다.
    const square = function (x) {
    	return x * x
    }
    ```
    

람다와 동일, 람다와 똑같이사용