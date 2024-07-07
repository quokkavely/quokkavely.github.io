---
layout : single
title : "[Security] Spring Security-Flow"
categories: Spring
tag : [Spring, Practice, Security]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


# Spring Security의 흐름

## 보안이 적용된 웹 요청의 일반적인 처리 흐름

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b6366c57-83bb-4f21-8eae-756e7a4a09c9"/>

- (1)에서 사용자가 보호된 리소스를 요청
- (2)에서 인증 관리자 역할을 하는 컴포넌트가 사용자의 크리덴셜(Credential)을 요청
    - **사용자의 크리덴셜(Credential)**이란 해당 사용자를 증명하기 위한 구체적인 수단을 의미. 일반적으로는 **사용자의 패스워드가 크리덴셜에 해당**
- (3)에서 사용자는 인증 관리자에게 크리덴셜(Credential)을 제공
- (4)에서 인증 관리자는 크리덴셜 저장소에서 사용자의 크리덴셜을 조회
- (5)에서 인증 관리자는 사용자가 제공한 크리덴셜과 크리덴셜 저장소에 저장된 크리덴셜을 비교해 검증 작업을 수행
- (6) 유효한 크리덴셜이 아니라면 Exception을 throw 한다.
- (7) 유효한 크리덴셜이라면 (8)에서 접근 결정 관리자 역할을 하는 컴포넌트는 사용자가 적절한 권한을 부여받았는지 검증
- (9) 적절한 권한을 부여받지 못한 사용자라면 Exception을 throw한다.
- (10) 적절한 권한을 부여받은 사용자라면 보호된 리소스의 접근을 허용한.

## 웹 요청에서의 서블릿 필터와 필터 체인의 역할

### **서블릿 필터(Servlet Filter)**

- 서블릿 필터(Servlet Filter)는 서블릿 기반 애플리케이션의 엔드포인트에 요청이 도달하기 전에 중간에서 요청을 가로챈 후 어떤 처리를 할 수 있도록 해주는 Java의 컴포넌트

> 서블릿 필터는 자바에서 제공하는 API이며, javax.servlet 패키지에 인터페이스 형태로 정의되어 있습니다.
> 
> 
> `javax.servlet.Filter` 인터페이스를 구현한 서블릿 필터는 웹 요청(request)을 가로채어 어떤 처리(전처리)를 할 수 있으며, 또한 엔드포인트에서 요청 처리가 끝난 후 전달되는 응답(reponse)을 클라이언트에게 전달하기 전에 어떤 처리(후처리)를 할 수 있다.
> 

 **Filter에서의 처리가 모두 완료되면** DispatcherServlet에서 클라이언트의 요청을 핸들러에 매핑하기 위한 다음 작업을 진행


### **필터체인**

Spring Security에서 **보안을 위한 작업을 처리하는 필터의 모음**

서블릿 필터는 하나 이상의 필터들을 연결해 필터 체인(Filter Chain)을 구성

- 각의 필터들이 `doFilter()`라는 메서드를 구현해야 하며, `doFilter()` 메서드 호출을 통해 필터 체인을 형성
- Filter 인터페이스를 구현한 다수의 Filter 클래스를 구현했다면 서블릿 필터에서 **특별한 작업을 수행한 뒤, HttpServlet을 거쳐 DispatcherServlet에 요청이 전달**되며, 반대로 DispatcherServlet에서 전달한 **응답에 대해 역시 특별한 작업을 수행할  수 있다.**
- Servlet FilterChain은 요청 URI path를 기반으로 HttpServletRequest를 처리 ⇒ 따라서 클라이언트가 서버 측 애플리케이션에 요청을 전송하면 서블릿 컨테이너는 요청 URI의 경로를 기반으로 어떤 Filter와 어떤 Servlet을 매핑할지 결정한다.
- Filter Chain에서 Filter의 순서는 매우 중요하며 Spring Boot에서 여러 개의 Filter를 등록하고 순서를 지정하기 위해서는 다음과 같은 두 가지 방법을 적용가능
    - Spring Bean으로 등록되는 Filter에 `@Order` 애너테이션을 추가하거나 `Orderd` 인터페이스를 구현해서 Filter의 순서를 지정할 수 있다.
    - `FilterRegistrationBean`을 이용해 Filter의 순서를 명시적으로 지정할 수 있다.

### **Spring Security에서의 필터 역할**

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/76b2b680-1c43-431c-9829-02196772248e"/>

- Spring Security에서 지원하는 Filter 종류는 무수히 많다.
- **Spring Security 필터 중 특별한 필터 :**
- ****`DelegatingFilterProxy` ,  `FilterChainProxy`
    
    `DelegatingFilterProxy`와 `FilterChainProxy` 클래스는 Filter 인터페이스를 구현하기 때문에 서블릿 필터로써의 역할을 수행한다.
    
    ### **1. DelegatingFilterProxy** ⭐
    
    Spring Security는 Spring의 핵심인 ApplicationContext를 이용한다.
    
    서블릿 필터와 연결되는 Spring Security만의 필터를 ApplicationContext에 Bean으로 등록한 후에 이 Bean들을 이용해서 보안과 관련된 여러 가지 작업을 처리하게 되는데 `DelegatingFilterProxy` 가 Bean으로 등록된 Spring Security의 필터를 사용하는 시작점이라고 생각하면 된다.
    
    그런데 `DelegatingFilterProxy` 는 보안과 관련된 어떤 작업을 처리하는 것이 아니라 **서블릿 컨테이너 영역의 필터와 ApplicationContext에 Bean으로 등록된 필터들을 연결해 주는 브리지 역할을 한다.**
    
    ### 2. **FilterChainProxy** ⭐
    
    - **Spring Security의 Filter를 사용하기 위한 진입점**
    - **FilterChainProxy부터 Spring Security에서 제공하는 보안 필터들이 필요한 작업을 수행한다**
    - Spring Security의 Filter Chain은 URL 별로 여러 개 등록할 수 있으며, Filter Chain이 있을 때 어떤 Filter Chain을 사용할지는 FilterChainProxy가 결정하며, 가장 먼저 매칭된 Filter Chain을 실행
        - `/api/**` 패턴의 Filter Chain이 있고, `/api/message` URL 요청이 전송하는 경우
            - `/api/**` 패턴과 제일 먼저 매칭되므로, 디폴트 패턴인 `/**`도 일치하지만 가장 먼저 매칭되는 `/api/**` 패턴과 일치하는 Filter Chain만 실행합니다.
        - /message/** 패턴의 Filter Chain이 없는데 `/message/` URL 요청을 전송하는 경우
            - 매칭되는 Filter Chain이 없으므로 디폴트 패턴인 `/**` 패턴의 Filter Chain을 실행합니다.
- 예제
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/2cac7052-6ef8-4ddd-b084-2052846f6138"/>
    
    - 여기서 하나하나가 다 필터가 관리하고 있다. 내부적으로 각각 하나의 필터에 맵핑된다.
    - SpringSecurity에서 지원하는 Filter의 종류가 정말 많고 내가 새롭게 구현할 일은 잘 없다. → 문제가 있을 때 어느부분이 잘못되었는지만 찾아볼 수 있으면 됨.

### Filter 인터페이스

filter는 init으로 초기화하고 추상메서드 구현해야한다.

```java
public class FirstFilter implements Filter {
     // (1) 초기화 작업
     public void init(FilterConfig filterConfig) throws ServletException {

     }

     // (2)  해당 Filter가 처리하는 실질적인 로직을 구현
     public void doFilter(ServletRequest request,
                          ServletResponse response,
                          FilterChain chain)
                          throws IOException, ServletException {
        // (2-1) 이곳에서 request(ServletRequest)를 이용해 다음 Filter로 넘어가기 전처리 작업을 수행한다.

         // (2-2)
        chain.doFilter(request, response);

        // (2-3)에서 response를 이용해 (2-2)의 chain.doFilter(request, response)가 호출된 이후에 할 수 있는 후처리 작업에 대한 코드를 구현할 수 있
     }

    
     public void destroy() {
        // (5) Filter가 사용한 자원을 반납하는 처리
     }
  }

```

## Spring Security의 권한 부여 처리 흐름

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/b75a718b-2e16-4cb8-8097-3be2fee83f87"/>

- (1)에서 사용자가 로그인 폼 등을 이용해 Username(로그인 ID)과 Password를 포함한 request를 Spring Security가 적용된 애플리케이션에 전송
- 사용자의 로그인 요청이 Spring Security의 Filter Chain까지 들어오면 여러 Filter들 중에서 `UsernamePasswordAuthenticationFilter`가 해당 요청을 전달받는다.
    - 사용자의 로그인 요청을 처리하는 Spring Security Filter는 UsernamePasswordAuthenticationFilter
- spring Security에서의 실질적인 인증 처리는 지금부터 시작
- 로그인 요청을 전달받은 UsernamePasswordAuthenticationFilter는 Username과 Password를 이용해 (2)와 같이 `UsernamePasswordAuthenticationToken`을 생성
    - UsernamePasswordAuthenticationToken은 Authentication 인터페이스를 구현한 구현 클래스이며, 여기에서 Authentication은 ⭐ **`아직 인증이 되지 않은 Authentication`이다.**
- 아직 인증되지 않은 Authentication을 가지고 있는 **UsernamePasswordAuthenticationFilter**는 (3)과 같이 해당 **Authentication**을 AuthenticationManager에게 전달
- AuthenticationManager는 인증 처리를 총괄하는 매니저 역할을 하는 인터페이스이고, AuthenticationManager를 구현한 구현 클래스가 바로 ProviderManager 이다. 즉, `ProviderManager가 **인증이라는 작업을 총괄하는 실질적인 매니저**`인 것
- (4)와 같이 `ProviderManager`로부터 **Authentication**을 전달받은 `AuthenticationProvider`는 (5)와 같이 `UserDetailsService`를 이용해 `UserDetails`를 조회
    - `UserDetails`는 **데이터베이스 등의 저장소에 저장된 사용자의 Username과 사용자의 자격을 증명해 주는 `크리덴셜(Credential)`인 Password, 그리고 사용자의 권한 정보를 포함하고 있는 컴포넌트**
- `UserDetailsService`는 (5)에서 처럼 데이터베이스 등의 저장소에서 사용자의 크리덴셜(Credential)을 포함한 사용자의 정보를 조회
- 데이터베이스 등의 저장소에서 조회한 사용자의 크리덴셜(Credential)을 포함한 사용자의 정보를 기반으로 (7)과 같이 `UserDetails`를 생성한 후, 생성된 `UserDetails`를 다시 `AuthenticationProvider`에게 전달
- **UserDetails**를 전달받은 `AuthenticationProvider`는 **PasswordEncoder**를 이용해 `UserDetails`에 포함된 **암호화된 Password**와 **인증을 위한** `Authentication`**안에 포함된 Password**가 일치하는지 검증
    - 검증에 성공하면 **UserDetails를 이용해 인증된 Authentication을 생성**합니다(9).
    - 만약 검증에 성공하지 못하면 Exception을 발생시키고 인증 처리를 중단합니다
- `AuthenticationProvider`는 **인증된 Authentication을** `ProviderManager`에게 전달합니다(10).
    - (2)에서의 `Authentication`은 **인증을 위해 필요한 사용자의 로그인 정보**
    - `ProviderManager`에게 전달한  **Authentication**은 **인증에 성공한 사용자의 정보**
- **Authentication**을 전달받은 **UsernamePasswordAuthenticationFilter**는 마지막으로 (12)와 같이 `SecurityContextHolder`를 이용해 `SecurityContext`에 **인증된 Authentication**을 저장
- ⭐ 그리고 SecurityContext는 이후에 Spring Security의 세션 정책에 따라서 HttpSession에 저장되어 사용자의 인증 상태를 유지하기도 하고, HttpSession을 생성하지 않고 무상태를 유지하기도 함

<br/>


## Spring Security의 컴포넌트로 보는 권한 부여(Authorization) 처리 흐름

아래는 사용자가 로그인 인증에 성공한 이후,  Spring Security에서 인증된 사용자에게 어떻게 권한을 부여하는지 그 처리 흐름을 나타내는 그림이다.

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/6f880dee-f3e6-47f5-a5d4-555901d1761f"/>

- Spring Security Filter Chain에서 URL을 통해 사용자의 액세스를 제한하는 **권한 부여 Filter**는 바로 `AuthorizationFilter` 이다.
- `AuthorizationFilter`는 먼저 (1)과 같이 SecurityContextHolder로부터 Authentication을 획득한다.
- 그리고 (2)와 같이 SecurityContextHolder로부터 획득한 Authentication과 HttpServletRequest를 `AuthorizationManager`에게 전달한다.
- `AuthorizationManager`는 권한 부여 처리를 총괄하는 매니저 역할을 하는 인터페이스이고, `RequestMatcherDelegatingAuthorizationManager`는 `AuthorizationManager`를 구현하는 구현체 중 하나이다.
    
    `RequestMatcherDelegatingAuthorizationManager`는 RequestMatcher 평가식을 기반으로 해당 평가식에 매치되는 `AuthorizationManager`에게 권한 부여 처리를 위임하는 역할을 한다
    
    ⭐ `RequestMatcherDelegatingAuthorizationManager`가 직접 권한 부여 처리를 하는 것이 아니라 `RequestMatcher`를 통해 매치되는 `AuthorizationManager` 구현 클래스에게 위임만 한다
    
- `RequestMatcherDelegatingAuthorizationManager` 내부에서 매치되는 `AuthorizationManager` 구현 클래스가 있다면 해당 `AuthorizationManager` 구현 클래스가 사용자의 권한을 체크한다(3).
- 적절한 권한이라면 (4)와 같이 다음 요청 프로세스를 계속 이어간다
- 만약 적절한 권한이 아니라면 (5)와 같이 `AccessDeniedException`이 throw되고 **ExceptionTranslationFilter**가 `AccessDeniedException`을 처리하게 된다.

<Br/>
<br/>


## Spring Security의 권한 부여 컴포넌트

### AuthorizationFilter

`AuthorizationFilter`는 URL을 통해 사용자의 액세스를 제한하는 권한 부여 Filter이며, Spring Security 5.5 버전부터 FilterSecurityInterceptor를 대체한다.

```java
public class AuthorizationFilter extends OncePerRequestFilter {

	private final AuthorizationManager<HttpServletRequest> authorizationManager;

  ...
  ...

  // (1)
	   //AuthorizationFilter 객체가 생성될 때, AuthorizationManager를 DI 받는다
	   //DI 받은 AuthorizationManager를 통해 권한 부여 처리를 진행
	public AuthorizationFilter(AuthorizationManager<HttpServletRequest> authorizationManager) {
		Assert.notNull(authorizationManager, "authorizationManager cannot be null");
		this.authorizationManager = authorizationManager;
	}

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		AuthorizationDecision decision = this.authorizationManager.check(this::getAuthentication, request); 
		// (2)  DI 받은 AuthorizationManager의 check() 메서드를 호출해 적절한 권한 부여 여부를 체크
		// URL 기반으로 권한 부여 처리를 하는 **AuthorizationFilter**는 AuthorizationManager의 구현 클래스로 
		// RequestMatcherDelegatingAuthorizationManager를 사용
		
		this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, decision);
		if (decision != null && !decision.isGranted()) {
			throw new AccessDeniedException("Access Denied");
		}
		filterChain.doFilter(request, response);
	}

  ...
  ...

}

```

### 

### AuthorizationManager

이름 그대로 권한 부여 처리를 총괄하는 매니저 역할을 하는 인터페이스

```java
@FunctionalInterface
public interface AuthorizationManager<T> {
  ...
  ...

	@Nullable
	AuthorizationDecision check(Supplier<Authentication> authentication, T object);
	 
	 // AuthorizationManager 인터페이스는 check() 메서드 하나만 정의되어 있으며, 
	 // Supplier<Authentication>와 제너릭 타입의 객체를 파라미터로 가	
}

```

### RequestMatcherDelegatingAuthorizationManager

`RequestMatcherDelegatingAuthorizationManager`는 AuthorizationManager의 구현 클래스 중 하나이며, 직접 권한 부여 처리를 수행하지 않고 `RequestMatcher`를 통해 매치되는 `AuthorizationManager` 구현 클래스에게 권한 부여 처리를 위임한다.

```java
public final class RequestMatcherDelegatingAuthorizationManager implements AuthorizationManager<HttpServletRequest> {

  ...
  ...

	@Override
	public AuthorizationDecision check(Supplier<Authentication> authentication, HttpServletRequest request) {
		if (this.logger.isTraceEnabled()) {
			this.logger.trace(LogMessage.format("Authorizing %s", request));
		}

    // (1) check() 메서드의 내부에서 (1)과 같이 루프를 돌면서 RequestMatcherEntry 정보를 얻은 후에
    // (2)와 같이 RequestMatcher 객체를 얻습니다
		for (RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>> mapping : this.mappings) {

			RequestMatcher matcher = mapping.getRequestMatcher(); // (2)
			MatchResult matchResult = matcher.matcher(request);
			if (matchResult.isMatch()) {   
					// (3)  MatchResult.isMatch()가 true이면 AuthorizationManager 객체를 얻은 뒤, 
					// AuthorizationManager 구현 클래스에게 권한 부여 처리를 위임한다.
				AuthorizationManager<RequestAuthorizationContext> manager = mapping.getEntry();
				if (this.logger.isTraceEnabled()) {
					this.logger.trace(LogMessage.format("Checking authorization on %s using %s", request, manager));
				}
				return manager.check(authentication,
						new RequestAuthorizationContext(request, matchResult.getVariables()));
			}
		}
		this.logger.trace("Abstaining since did not find matching RequestMatcher");
		return null;
	}
}

```

⭐ 여기서의 `RequestMatcher`는 SecurityConfiguration에서 `.antMatchers("/orders/**").hasRole("ADMIN")` 와 같은 메서드 체인 정보를 기반으로 생성된다



### Comment
너무 어려워서 정리하는데 한참 걸렸다,, 근데도 이해가 안가는 것은 함정;;
어서 이해해야하는데 답이없다 정말 ㅠㅠㅠ