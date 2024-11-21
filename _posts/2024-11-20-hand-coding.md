---
layout : single
title : "[Java] 1차면접 : 손코딩 회고"
categories: Programmers
tag : [CodingTest, 실습]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}



## 손코딩 테스트 회고 : 배열 입력과 최소 루프 찾기

### 1️⃣문제
이름이 서로다른 1000명의 이름과 출생년도를 배열에 입력

1. 첫 번째 문제: 1000명의 이름과 출생년도를 배열에 어떤 방식으로 입력할 것인가?
2. 두 번째 문제:
    - 이름과 출생년도를 찾을 때 최소 루프 횟수를 계산하고 답을 구하라.

면접관님이 문제를 2번 말해주는데 떨려서 제대로 필기하지 못했고 요구사항이 명확히 이해하지 못한 채로 (다시 질문해서 문제를 들었음에도 불구하고😂) 답은 제출하지않고 풀이과정만 제출했다....

### 2️⃣실수했던 부분
당시 내가 작성한 코드는 아래와 같다

```java

public class Member {
    String name;
    int year;
}

//이름과 출생년도에 대한 배열이 따로 주어진다고 생각했고
//임의로 이름 배열을 nameArr, 출생년도 배열을 yearArr로 정함

List<Member> members = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    members.add(nameArr[i], yearArr[i]); 
    //new Memeber 로 해당 값을 넣었어야 했는데 그냥 add 만 했다.
}

for (Member member : members) {
    if (member.getName().equals("Any")) { // 특정 조건
        // 처리 로직
    }
}
```

이 코드는 풀이 과정만 적었고, 문제가 요구했던 답인 배열 입력 방법과 최소 루프 횟수를 명시적으로 적지 않았다.
즉, 문제의 핵심인 "답"을 적지 않고 과정만 제출한 것이 실수였다.

### 3️⃣ 다시 푼 풀이

테스트가 끝난 후, 문제를 다시 차분히 읽고 다시 풀어봤다.

1. 첫 번째 문제: 배열 입력 방식
    1000명의 이름과 출생년도를 배열에 입력하는 가장 단순한 방법은 for 루프를 사용하는 것이다.

```java
public class Member {
    String name;
    int year;

    public Member(String name, int year) {
        this.name = name;
        this.year = year;
    }
}

List<Member> members = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    members.add(new Member("Name" + i, 1980 + (i % 40))); // 이름과 출생년도 생성
}
```
이 방식은 루프를 한 번만 돌면서 데이터 입력을 완료하기 때문에 시간 복잡도가 O(N)이다.


2. 두 번째 문제: 최소 루프 횟수
    - 이름과 출생년도를 찾기 위해 배열을 탐색할 때, 최소 루프 횟수는 데이터의 크기인 N이다.
    - 따라서 루프를 한 번만 사용하여 조건을 검사할 수 있다:

```java
for (Member member : members) {
    if (member.year > 2000) { // 2000년 이후 출생자 찾기
        System.out.println(member.name);
    }
}
```
- 최소 루프 횟수: 1000번 (배열 크기만큼 한 번의 탐색만 필요)
- 시간 복잡도: O(N)

<br>

---

긴장하기도 했고 손코딩문제가 나올거라고 잠깐 예상했지만 <br> 논술에 꽂혀서 게스티메이션이 나올거라고 생각했는데 <br> 문제가 2개 일 것이라고는 더더욱 상상도 못했다. <br> 긴장하느라 답도 안적어서 제출한 것이 너무 어이가 없고 속상하다..<br>
좋은 결과를 기대하긴 어렵지만, <br> 한 번도 손코딩을 안해봐서 조금은 대비를 해두어야 하는 것 같다.


<br>
<br>
<br>
<br>
<br>