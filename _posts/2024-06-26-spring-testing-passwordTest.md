---
layout : single
title : "[Spring]JUnit 활용한 Test Case 작성"
categories: Practice
tag : [Spring, 실습, JPA, Testing]
author_profile: true
---

## **`RandomPasswordGenerator` 클래스의 단위 테스트 케이스 작성**

### 구현되어있는 코드에 Test 코드 구현하기

```java
public class RandomPasswordGenerator {
    public static String generate(int numberOfUpperCaseLetters,
                                  int numberOfLowerCaseLetters,
                                  int numberOfNumeric,
                                  int numberOfSpecialChars) {
        String upperCaseLetters = RandomStringUtils.random(numberOfUpperCaseLetters, 65, 90, true, false);
        String lowerCaseLetters = RandomStringUtils.random(numberOfLowerCaseLetters, 97, 122, true, false);
        String numbers = RandomStringUtils.randomNumeric(numberOfNumeric);
        String specialChars = RandomStringUtils.random(numberOfSpecialChars, 33, 47, false, false);

        String combinedLetters = combineLetters(upperCaseLetters, lowerCaseLetters, numbers, specialChars);
        List<Character> shuffledLetters = shuffleLetters(combinedLetters);
        return shuffledLetters.stream()
                .collect(StringBuilder::new, StringBuilder::append, StringBuilder::append)
                .toString();
    }

    private static List<Character> shuffleLetters(String combinedLetters) {
        List<Character> shuffledLetters = combinedLetters.chars().mapToObj(c -> (char) c).collect(toList());
        Collections.shuffle(shuffledLetters);
        return shuffledLetters;
    }

    private static String combineLetters(String upperCaseLetters, String lowerCaseLetters, String numbers, String specialChars) {
        return upperCaseLetters.concat(lowerCaseLetters).concat(numbers).concat(specialChars);
    }
}
```

### 구현조건

- `RandomPasswordGeneratorTest`내의 비어 있는 한 개의 테스트 케이스를 JUnit을 사용해서 작성
    - 비어 있는 한 개의 테스트 케이스
        - `generateTest()`
- 제한 사항
    - JUnit의 Assertion 메서드를 이용해야 합니다.
    - `RandomPasswordGenerator.generate()` 파라미터의 입력 값으로 다양한 숫자를 입력해서 테스트

### 처음 구현한 코드

```java
 public class RandomPasswordGeneratorTest {
 
    @DisplayName("랜덤 패스워드 생성 테스트")
    @Test
    public void generateTest() {
 
			//given
        int numberOfNumeric= 2; //0 이상의 숫자개수
        int numberOfUpperCaseLetters=3;
        int numberOfLowerCaseLetters=4;
        int numberOfSpecialChars=1; //특수문자 1개
        int length = numberOfLowerCaseLetters+numberOfNumeric+numberOfUpperCaseLetters+numberOfSpecialChars;

      //when
        String password = RandomPasswordGenerator.generate(numberOfUpperCaseLetters,numberOfLowerCaseLetters,numberOfNumeric,numberOfSpecialChars);

      //then
        Assertions.assertEquals(length, password.length());
        Assertions.assertEquals(countOfLower(password),numberOfLowerCaseLetters);
        Assertions.assertEquals(countOfUpper(password),numberOfUpperCaseLetters);
        Assertions.assertEquals(countOfNumber(password), numberOfNumeric);
        Assertions.assertEquals(length-countOfLower(password)-countOfUpper(password)-countOfNumber(password),numberOfSpecialChars);
```

- 임의로 변수에 Password 길이를 할당함 (숫자/대문자/소문자/특수문자 개수)
- when에서 테스트 할 해당 메서드에 파라미터를 넣고
- then에서 Assertions.assertEqauls로 생성된 비밀번호의 개수가 일치하는 지 테스트

나는 따로  for문을 사용한 메서드를 private로  아래에 생성함.

```java
    private int countOfUpper(String password){
        int count = 0;
        for(char c: password.toCharArray()){
            if (Character.isUpperCase(c)) {
                count++;
            }
        }
      return count;
    }

    private int countOfLower(String password){
        int count=0;
        for(char c: password.toCharArray()){
            if(Character.isLowerCase(c)){
                count++;
            }
        }
        return count;

    }

    private int countOfNumber(String password){
        int count =0;
        String number="123456789";
        for(char c : password.toCharArray()){
            if(number.indexOf(c)!=-1){
                count++;
            }
        }
        return count;
    }
```

- Stream으로 하려다가 헷갈려서 for문 먼저 구현하였다.
- Characater.isUpperCase와 Character.isLowerCase는  소문자 / 대문자 일 경우 true 아니면 false 를 반환함.
- 특수 문자는 처음에 구현 못해서 총 길이에서 나머지 값(대문자,소문자,숫자로 구성된 문자열의 길이)을 뺀 길이로 계산함.

### 스트림 구현

Stream 에서 map만 사용해봤는데 chars 사용하면 문자열 내의 각 문자를 `IntStream`으로 반환

```java
return (int) password.chars().filter(Character::isUpperCase).count();
return (int) password.chars().filter(Character::isLowerCase).count();
return (int) password.chars().filter(Character::isDigit).count();
return (int) password.chars().filter( c -> !Character.isLetterOrDigit(c)).count();
```

- isDigit  :  `boolean isDigit(char ch)`
- 특수문자는 letter와 Digit을 제외한 것을 제외한 것만 갯수 세도록 함.,

### 개선 포인트

1. 동일하게 반복문을 돌고 있어서  반복문을 하나로 돌 수 있도록 구현하는 것.
    
    ```java
    public class RandomPasswordGeneratorTest {
    
        @DisplayName("랜덤 패스워드 생성 테스트")
        @Test
        public void generateTest() {
            // given
            int numberOfNumeric = 2; // 0 이상의 숫자개수
            int numberOfUpperCaseLetters = 3;
            int numberOfLowerCaseLetters = 4;
            int numberOfSpecialChars = 1; // 특수문자 1개
            int length = numberOfLowerCaseLetters + numberOfNumeric + numberOfUpperCaseLetters + numberOfSpecialChars;
    
            // when
            String password = RandomPasswordGenerator.generate(numberOfUpperCaseLetters, numberOfLowerCaseLetters, numberOfNumeric, numberOfSpecialChars);
    
            int actualLower = 0;
            int actualUpper = 0;
            int actualNumber = 0;
            int actualSpecial = 0;
    
            for (char c : password.toCharArray()) {
                if (Character.isLowerCase(c)) actualLower++;
                else if (Character.isUpperCase(c)) actualUpper++;
                else if (Character.isDigit(c)) actualNumber++;
                else actualSpecial++;
            }
    }
    ```
    
2. 숫자/ 대문자/ 소문자/ 특수문자 별로 나눠서 통과 되었다는 Test가 필요할 것 같음. @BeforeAll 사용해서 테스트 초기화를 한번만 하도록 사용하면 가능 
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/9d18ec68-b6e9-4744-a8c4-a8b013f5255c"/>
    
3. 특수문자를 대충 검증하고 있는데 세밀하게 검증 할 필요가 있음.
    
    RandomPasswordGenerator에서 구현된 generate 메서드에서 특수문자 생성 시 사용되는 랜덤메서드에서 유니코드 33~47 사이로 구현되고 있기 때문에 해당 유니코드를 포함하는 문자만 셀 수 있도록 해야 함.
    
    ```java
    String specialChars = RandomStringUtils.random(numberOfSpecialChars, 33, 47, false, false);
    ```
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/25c1a86a-182d-4abc-a4a1-ff793f3d446b" width=400/>
    

### 개선 후 코드

```java
package com.springboot.homework;

import com.springboot.helper.RandomPasswordGenerator;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class RandomPasswordGeneratorTest {
    static int numberOfNumeric;
    static int numberOfUpperCaseLetters;
    static int numberOfLowerCaseLetters;
    static int numberOfSpecialChars;
    static int expectedLength;
    static String currentPassword;

    @BeforeAll
    public static void init(){
        numberOfNumeric= 2; //0 이상의 숫자개수
        numberOfUpperCaseLetters=3;
        numberOfLowerCaseLetters=4;
        numberOfSpecialChars=1; //특수문자 1개
        expectedLength = numberOfLowerCaseLetters+numberOfNumeric+numberOfUpperCaseLetters+numberOfSpecialChars;
        currentPassword = RandomPasswordGenerator.generate(numberOfUpperCaseLetters,numberOfLowerCaseLetters,numberOfNumeric,numberOfSpecialChars);
    }

    @DisplayName("생성된 패스워드 길이가 입력한 숫자와 일치하는지 검증하세요")
    @Test
    public void lengthOfPasswordTest() {
        Assertions.assertEquals(expectedLength, currentPassword.length());
    }

    @DisplayName("생성된 패스워드의 알파벳 대문자 개수가 입력한 파라미터의 숫자와 일치하는지 검증하세요")
    @Test
    public void numberOfUpperCaseLettersTest() {

        Assertions.assertEquals(currentPassword.chars()
                .filter(Character::isUpperCase)
                .count(), numberOfUpperCaseLetters);
    }

    @DisplayName("생성된 패스워드의 알파벳 소문자 개수가 입력한 파라미터의 숫자와 일치하는지 검증하세요")
    @Test
    public void numberOfLowerCaseLettersTest() {
        Assertions.assertEquals(currentPassword.chars()
                .filter(Character::isLowerCase)
                .count(), numberOfLowerCaseLetters);
    }

    @DisplayName("생성된 패스워드의 숫자의 개수가 입력한 파라미터의 숫자와 일치하는지 검증하세요")
    @Test
    public void numberOfNumericTest() {
        Assertions.assertEquals(currentPassword.chars()
                .filter(Character::isDigit)
                .count(), numberOfNumeric);
    }

    @DisplayName("생성된 패스워드의 특수문자의 개수가 입력한 파라미터의 숫자와 일치하는지 검증하세요")
    @Test
    public void numberOfSpecialChars() {
        Assertions.assertEquals(currentPassword.chars()
                .filter(this::isValidSpecial)
                .count(),numberOfSpecialChars);

    }

     private boolean isValidSpecial(int c){
     return c >=33 && c<=47;
    }
}

```
- 초기화 해야 할 코드 양이 늘었지만 아래와 같이 테스트 메세지가 명확하게 나옴
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/a46b9e54-ac6e-403c-9781-2e7526e39cec"/>

## COMMENT
    여태껏 하던 것 보다는 자바만 잘하면 되는 문제라고 생각되어서 쉬운듯 어려운듯 했다. 테스트 코드를 직접 내가 숫자로 할당해서 이게 맞는지 확인하는 것이었는데 여러개의 테스트 케이스가 더 들어와야 하는 것 아닌 가 싶은데 만약 그렇게 되면 랜덤으로 수를 받는 것인지 어떤지는 모르겠다! 코딩테스트 할때처럼 내가 케이스를 만드는 것과 비슷하다고 느껴져서 조금 흥미로웠다, 정말 내가 예외를 잡기위한 테스팅 코드를 짜는 것이 중요해서 코드를 쳐야할 양이 앞으로 더 많아질 것 같다.