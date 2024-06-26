---
layout : single
title : "[MVC] 서블릿 실습"
categories: Practice
tag : [MVC,실습,Spring]
author_profile: true
---

## 서블릿실습

### get

1. 인텔리제이에서 프로젝트 생성 → build.gradle 파일 수정해주기 (위 사진에서 아래 빨간색 박스의 내용 추가해주기)
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d2b08633-9f0e-419c-8d40-deb7d70a11fd" width=500/>
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6c655469-94a9-403b-a6dd-4c729a1a3ccd" width=500/>
    
    - 코드
      
        ```groovy
        plugins {
            id 'java'
            id 'war'
        }
        
        group = 'org.example'
        version = '1.0-SNAPSHOT'
        
        repositories {
            mavenCentral()
        }
        
        dependencies {
            implementation group : 'javax.servlet', name: 'javax.servlet-api', version : '3.1.0'
            testImplementation platform('org.junit:junit-bom:5.10.0')
            testImplementation 'org.junit.jupiter:junit-jupiter'
        }
        
        test {
            useJUnitPlatform()
        }
        tasks.withType(JavaCompile){
            options.encoding = 'UTF-8'
        }
        ```
        

1. 작성이 완료되면 코끼리 아이콘이 뜸
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6c76fbca-4f96-454a-86d2-7791eeba10e4" width=100/>
    
    빌드가 설정되었으니 리로드하라는 아이콘
    
2. 서블릿 작성
    1. code
       
        ```groovy
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        import java.io.PrintWriter;
        
        @WebServlet("/hello")
        public class TestServlet extends HttpServlet{
            protected  void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
                response.setContentType("text/html");
                PrintWriter out = response.getWriter();
                out.println("<h1>Hello, World!</h1>");
            }
        }
        ```
        
        - path에 오류 생길 경우 annotation 작성하면 해결
    
3. xml 작성
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/31b2c2e7-6238-43a1-8e12-51df4abe9c27" width=500/>
    
1. 완료 후 clean 누르고 war 눌러서 build 해줌
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5ef01bb1-66ed-43a1-923c-34faad06fda7" width=300/>
    
1. 빌드 후 bulid/ lips의 폴더 열어보기
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/33c59b40-fb9d-4ca9-bb28-850ed5a5d433" width=300/>
    
1. 폴더에 war파일 생성되었으면 잘라내기 누르기
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/e5c457d1-2baf-4ab8-8470-49ad47fc5966" width=500/>
    
2. tomcat> webapps에 가서 기존 파일 삭제하고 붙여넣기(7에서 잘라낸 파일) 한 후 폴더 더블 클릭하면 압축이 풀림
3. tomcat은 항상 켜두어야 함 (켜져 있을 경우 더블 클릭하면 자동으로 압축이 풀림)
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/c1bbeb58-5534-4043-be3a-a5666a15a43b" width=500/>
    
1. 압축이 잘 풀리면 
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/76e6567b-53ff-4009-aa6c-433b30f4fb1d" width=500/>
    
    1. 사이트에서 [localhost:8080/juneTest-1.0-SNAPSHOT/welcome](http://localhost:8080/juneTest-1.0-SNAPSHOT/welcome) 들어가기
       
        이렇게 들어가면 잘 나옴
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/0078b44e-52ac-4615-93cd-ce1deb3aa0b3" width=500/>
    
1. 다섯개 서블릿 만들어서 수동배포해보기
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/eb02d03d-a706-4d92-842f-05485586a2e1" width=300/>

서블릿은 하나 실수로 잘못 빌드되어 수정하려면 다시 클린 누르고 war 빌드해서

생긴 파일들을 다시 톰캣에 옮겨서 실행해야되는데 수동배포 이거 굉장히 번거롭다…

### post 실습

1. [PostHandlerServlet.java](http://PostHandlerServlet.java) 파일 생성
   
    ```groovy
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import javax.servlet.annotation.WebServlet;
    
    @WebServlet("/postHandler")
    public class PostHandlerServlet extends HttpServlet {
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            String name = request.getParameter("name");
            response.setContentType("text/html");
            PrintWriter out = response.getWriter();
            out.println("<h1>Hello, " + name + "!</h1>");
        }
    }
    ```
    
1. webapp 폴더 내에 index. html 파일 생성
   
    ```groovy
    <!DOCTYPE html>
    <html>
    <head>
        <title>Post Request</title>
    </head>
    <body>
    <form action="postHandler" method="post">
        <label for="name">Name:</label>
        <input type="text" id="name" name="name">
        <input type="submit" value="Submit">
    </form>
    </body>
    </html>
    
    ```
    
2. xml 파일
   
    ```groovy
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://java.sun.com/xml/ns/javaee"
             xsi:schemaLocation="http://java.sun.com/xml/ns/javaee  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
             version="2.5">
        <display-name>ServletTest</display-name>
        <welcome-file-list>
            <welcome-file>index.html</welcome-file>
        </welcome-file-list>
    </web-app>
    ```
    
1. clean 후 war 파일 빌드 후  빌드된 파일을 잘라내기 해서 tomcat - webapp의 기존 파일 삭제하고 다시 붙여넣기하기 
2. get요청이 아니라서 경로는 빌드된 파일명만 입력하면 됨 ([http://localhost:8080/juneTest-1.0-SNAPSHOT/](http://localhost:8080/juneTest-1.0-SNAPSHOT/))
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/7f1f462e-0e1b-4bac-83e0-99006d31194b" width=300/>
    
3. 이름 타이핑 후 제출하면   posthandler http통신은 status 200
   
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/1acf2252-3c3d-41c4-b425-8c2d8f294b18" width=500/>
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d521c91e-5ead-4b7c-92b4-27e3bfd26d3c" width=500/>
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/fbde942c-0b1a-4039-951a-3ee44bf38357" width=500/>


<br/>

## 쿠키세션 실습

서블릿 실습 후

webapp에 index.jpx 파일 생성 후 코드 생성

```groovy
<!DOCTYPE html>
<html>
<head>
    <title>Login Page</title>
</head>
<body>
    <h2>Login</h2>
    <form action="login" method="post">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username">
        <br>
        <label for="password">Password:</label>
        <input type="password" id="password" name="password">
        <br>
        <input type="checkbox" id="rememberMe" name="rememberMe">
        <label for="rememberMe">Remember Me</label>
        <br>
        <input type="submit" value="Login">
    </form>
</body>
</html>

```

xml파일에  index.html 되어있던 것을 index.jsp로 변경

그리고 LoginServelet, 및 ProfileServlet 생성

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String rememberMe = request.getParameter("rememberMe");

        HttpSession session = request.getSession();
        session.setAttribute("username", username);

        if ("on".equals(rememberMe)) {
            Cookie cookie = new Cookie("username", username);
            cookie.setMaxAge(60 * 60 * 24); // 1 day
            response.addCookie(cookie);
        }

        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>Welcome, " + username + "</h1>");
        out.println("<a href='profile'>Go to Profile</a>");
        out.println("</body></html>");
    }
}

---
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/profile")
public class ProfileServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpSession session = request.getSession(false);
        String username = null;

        if (session != null) {
            username = (String) session.getAttribute("username");
        }

        if (username == null) {
            Cookie[] cookies = request.getCookies();
            if (cookies != null) {
                for (Cookie cookie : cookies) {
                    if ("username".equals(cookie.getName())) {
                        username = cookie.getValue();
                        break;
                    }
                }
            }
        }

        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        if (username != null) {
            out.println("<h1>Profile Page</h1>");
            out.println("<h2>Welcome back, " + username + "</h2>");
        } else {
            out.println("<h1>No user information found. Please <a href='index.jsp'>login</a>.</h1>");
        }
        out.println("</body></html>");
    }
}
```

HttpSession 객체를 통해 세션가져옴

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/354b9afa-ec90-4d02-aa1b-aca7e433ce87" width=300 height=150/><br/>
<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5476b9b1-77d2-464e-9085-ddbdb85e0cfa" width=300 height=150/><br/>
<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/5ae579b1-01f2-4953-8ac9-5002c57ef0b0" width=300 height=150>


<br/>
## 계산기-에러페이지 실습

1. 에러페이지 실습
    
    ```java
    
            try {
                int number1=0;
                int number2=0;
                int calculateResult=0;
                String result="";
                    if(isValidNum(number2Str)&&isValidNum(number1Str)){
                        number1 = Integer.parseInt(number1Str);
                        number2 = Integer.parseInt(number2Str);
                    }else{
                        result="숫자만 입력하세요";
    
                    }
    
                if(!isValidOper(operatorStr)){
                    result="연산자만 입력가능합니다.";
    
                 }
                // ArithmeticException 발생 가능
                if(number2==0 && operatorStr.equals("/")) {
                   result="0으로 나눌 수 는 없습니다.";
    
                }else{
                    calculateResult = calculate(number1, operatorStr, number2);
                    result=Integer.toString(calculateResult);
                }
                response.setContentType("text/html");
                PrintWriter out = response.getWriter();
                out.println("<html><body>");
                out.println("<h1>Result: " + result + "</h1>");
                out.println("</body></html>");
            } catch (NumberFormatException | ArithmeticException e) {
                throw new ServletException("Invalid input or division by zero", e);
            }
        }
    
        public int calculate(int n1, String oper, int n2) {
            int result = 0;
            switch (oper) {
                case "+":
                    result = n1 + n2;
                    break;
                case "-":
                    result = n1 - n2;
                    break;
                case "/":
                    result = n1 / n2;
                    break;
                case "*":
                    result = n1 * n2;
                    break;
            }
            return result;
        }
    
        public boolean isValidNum(String num) {
            String numbers="0123456789";
            for(char c : num.toCharArray()){
                if(numbers.indexOf(c)==-1){
                    return false;
                }
            }
            return true;
        }
    
        public boolean isValidOper(String oper){
            String operators = "+-*/";
            int count=0;
            for(char c : oper.toCharArray()){
                if(operators.indexOf(c)==-1){
                    count++;
                }else{
                    return false;
                }
            }
            return count==1;
        }
    }
    ```
    
    1. 검증이 안됨
        1. 숫자검증
            <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3e74e239-a8bd-480c-b67b-8ae520383c83"/>
            
        2. 나누는 수 검증
            
            <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6cbf3a32-8cff-432f-ba8c-643dba2f48cb"/>
            
        3. oper  검증
            
           <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/2bb0d1d9-92f0-4086-a878-27db139410b0"/>
            
        
    2. 문자처리 해야함.
        
        1의 b 보면 문자 처리도 안된 것 같아서
        
        if문이랑 한국어 나올 수 있게 아래 코드도 수정해야 함.
        
        ```java
        response.setContentType("text/html; charset=UTF-8");
        ```
        
    3. if 문 수정
2. 수정
    
    ```java
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.io.PrintWriter;
    
    @WebServlet("/calculate")
    public class CalculateServlet extends HttpServlet {
        protected void doPost(HttpServletRequest request, HttpServletResponse response)
                throws ServletException, IOException {
            String number1Str = request.getParameter("number1");
            String operatorStr = request.getParameter("operator");
            String number2Str = request.getParameter("number2");
    
            try {
                int number1=0;
                int number2=0;
                int calculateResult=0;
                String result="";
    
                    if(isValidNum(number2Str)&&isValidNum(number1Str)){
                        number1 = Integer.parseInt(number1Str);
                        number2 = Integer.parseInt(number2Str);
                        if(isValidOper(operatorStr)){
                            calculateResult = calculate(number1, operatorStr, number2);
                            result=Integer.toString(calculateResult);
                        }else{
                            result="연산자만 입력가능합니다.";
                        }
                    }else{
                        result="숫자만 입력하세요";
                    }
                // ArithmeticException 발생
                response.setContentType("text/html; charset=UTF-8");
                PrintWriter out = response.getWriter();
                out.println("<html><body>");
                out.println("<h1>Result: " + result + "</h1>");
                out.println("</body></html>");
            } catch (NumberFormatException | ArithmeticException e) {
                throw new ServletException("Invalid input or division by zero", e);
            }
        }
    
        public int calculate(int n1, String oper, int n2) {
            int result = 0;
            switch (oper) {
                case "+":
                    result = n1 + n2;
                    break;
                case "-":
                    result = n1 - n2;
                    break;
                case "/":
                    result = n1 / n2;
                    break;
                case "*":
                    result = n1 * n2;
                    break;
            }
            return result;
        }
    
        public boolean isValidNum(String num) {
            String numbers="0123456789";
            for(char c : num.toCharArray()){
                if(numbers.indexOf(c)==-1){
                    return false;
                }
            }
            return true;
        }
    
        public boolean isValidOper(String oper){
            String operators = "+-*/";
            boolean result=false;
            for(char c : oper.toCharArray()){
                if(operators.indexOf(c)!=-1 && oper.length()==1){
                    result= true;
                }
            }
            return result;
        }
    }
    ```
    
    1. 0으로 나눌때
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/ad6eea7a-d632-4839-81e8-0466d770baab"/>
        
    2. 연산자가 아닌 문자입력
        
         <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/3c2e738c-da52-4eb1-aa68-1e30fda0f885"/>
        
    3. 숫자가 아닌 문자입력
        
        <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/d811fd35-4dd7-4720-9ac5-dfc8c5270856"/>
        
    
3. 추가 보완 해보기
    1. 에러에 맞는 에러페이지를 설정해주어야 함.
    2. operator를 select로 사용할 수 있음> 에러방지 가능  <br/>


    **index.jsp**
        
     ```java
        <body>
            <h2>Calculator</h2>
            <h4>Please enter numbers and operator</h4>
            <form action="calculator" method="post">
                <label for="number">Number1:</label>
                <input type="text" id="number" name="number">
                <select id ="operator" name="operator">
                    <option value="+">+</option>
                    <option value="-">-</option>
                    <option value="/">*</option>
                    <option value="*">/</option>
                </select>
                <label for="number2">Number2:</label>
                <input type="text" id="number2" name="number2">
                <input type="submit" value="Submit">
            </form>
        </body>
        </html>
        
    ```
    
    **CalculateServlet.java**
    
     ```java
        String number1Str = request.getParameter("number1");
        String operatorStr = request.getParameter("operator");
        String number2Str = request.getParameter("number2");
        
        try{
        
        int number1 = Integer.parseInt(number1Str);
            int number2 = Integer.parseInt(number2Str);
            int result;
            
            switch (oper) {
                    case "+":
                        result = number1 + number2;
                        break;
                    case "-":
                        result = number1 - number2;
                        break;
                    case "/":
                        result = number1 / number2;
                        break;
                    case "*":
                        result = number * number2;
                        break;
                }
        }
     ```

<br/>

### Comment

select 사용하는 건 사실 몰랐는데 대부분이 이런식으로 사용했다고 해서 조금 충격이었다,,<br/>
아직 html이나 이런건 익숙하지도 않고 try-catch문도 많이 사용해본 적이 없어서<br/>
for문과 if문으로 구현해내는 게 익숙해서 검증을 그런식으로 밖에 하지 못했던 것 같은데<br/>
강사님께서 어떻게든 구현해내는 것이 우선이라고 하셔서 그나마 위안이었다.<br/>
서블릿은 사용할 일이 많이 없다고 하셨으니,, 일단
스프링을 잘하는게 목표니까 오늘부터 스프링 화이팅...!


<br/>
<br/>
<br/>
<br/>
<br/>

---
