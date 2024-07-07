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
    - **사용자의 크리덴셜(Credential)** 이란 해당 사용자를 증명하기 위한 구체적인 수단을 의미. 일반적으로는 **사용자의 패스워드가 크리덴셜에 해당**
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
    - doFilter()는 Filter에서 어떤 처리를 하는 시작점
- Filter 인터페이스를 구현한 다수의 Filter 클래스를 구현했다면 서블릿 필터에서 **특별한 작업을 수행한 뒤, HttpServlet을 거쳐 DispatcherServlet에 요청이 전달**되며, 반대로 DispatcherServlet에서 전달한 **응답에 대해 역시 특별한 작업을 수행할  수 있다.**
- Servlet FilterChain은 요청 URI path를 기반으로 HttpServletRequest를 처리 ⇒ 따라서 클라이언트가 서버 측 애플리케이션에 요청을 전송하면 서블릿 컨테이너는 요청 URI의 경로를 기반으로 어떤 Filter와 어떤 Servlet을 매핑할지 결정한다.
- Filter Chain에서 Filter의 순서는 매우 중요하며 Spring Boot에서 여러 개의 Filter를 등록하고 순서를 지정하기 위해서는 다음과 같은 두 가지 방법을 적용가능
    - Spring Bean으로 등록되는 Filter에 `@Order` 애너테이션을 추가하거나 `Orderd` 인터페이스를 구현해서 Filter의 순서를 지정할 수 있다.
    - `FilterRegistrationBean`을 이용해 Filter의 순서를 명시적으로 지정할 수 있다.

### **Spring Security에서의 필터 역할**

<img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/76b2b680-1c43-431c-9829-02196772248e" width =500/>

- Spring Security에서 지원하는 Filter 종류는 무수히 많다.
- **Spring Security 필터 중 특별한 필터 : `DelegatingFilterProxy` , `FilterChainProxy`**
    
    `DelegatingFilterProxy`와 `FilterChainProxy` 클래스는 Filter 인터페이스를 구현하기 때문에 서블릿 필터로써의 역할을 수행한다.
    
### **1. DelegatingFilterProxy** ⭐
    
Spring Security는 Spring의 핵심인 <span style="color:OrangeRed"> ApplicationContext를 이용 </span> 한다.
    
서블릿 필터와 연결되는 Spring Security만의 필터를 ApplicationContext에 Bean으로 등록한 후에 이 Bean들을 이용해서 보안과 관련된 여러 가지 작업을 처리하게 되는데 <span style="color:OrangeRed"> `DelegatingFilterProxy` 가 Bean으로 등록된 Spring Security의 필터를 사용하는 시작점 </span> 이라고 생각하면 된다.
    
그런데 `DelegatingFilterProxy` 는 보안과 관련된 어떤 작업을 처리하는 것이 아니라 **서블릿 컨테이너 영역의 필터와 ApplicationContext에 Bean으로 등록된 <span style="color:OrangeRed">필터들을 연결해 주는 브리지 역할</span>을 한다.**
    
### 2. **FilterChainProxy** ⭐
    
- <span style="color:OrangeRed">**Spring Security의 Filter를 사용하기 위한 진입점**</span>
- **FilterChainProxy부터 Spring Security에서 제공하는 보안 필터들이 필요한 작업을 수행한다**
- Spring Security의 <span style="color:Green"> Filter Chain은 URL 별로 여러 개 등록할 수 있으며,</span> Filter Chain이 있을 때 어떤 Filter Chain을 사용할지는 <span style="color:Green">**FilterChainProxy가 결정**</span>하며, 가장 먼저 매칭된 Filter Chain을 실행
    - `/api/**` 패턴의 Filter Chain이 있고, `/api/message` URL 요청이 전송하는 경우
        - `/api/**` 패턴과 제일 먼저 매칭되므로, 디폴트 패턴인 `/**`도 일치하지만 가장 먼저 매칭되는 `/api/**` 패턴과 일치하는 Filter Chain만 실행합니다.
    - /message/** 패턴의 Filter Chain이 없는데 `/message/` URL 요청을 전송하는 경우
            - 매칭되는 Filter Chain이 없으므로 디폴트 패턴인 `/**` 패턴의 Filter Chain을 실행합니다.
- 예제
    
    <img src="https://github.com/quokkavely/quokkavely.github.io/assets/165968530/2cac7052-6ef8-4ddd-b084-2052846f6138" width=500/>
    
    - 여기서 하나하나가 다 필터가 관리하고 있다. 내부적으로 각각 하나의 필터에 맵핑된다.
    - SpringSecurity에서 지원하는 Filter의 종류가 정말 많고 내가 새롭게 구현할 일은 잘 없다. → 문제가 있을 때 어느부분이 잘못되었는지만 찾아볼 수 있으면 됨.
    
    <br/>
    <br/>

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
<br/>

<br/>

## Spring Security의 인증 컴포넌트

### UsernamePasswordAuthenticationFilter

 일반적으로 로그인 폼에서 제출되는 Username과 Password를 통한 인증을 처리하는 Filter

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f7511b05-f7b2-446f-8b14-0575185a1028/d2a6dad8-a1af-4bbb-96a0-fbbdb03e9648/Untitled.png)

- **AbstractAuthenticationProcessingFilter를 상속**
- 1 : 클라이언트의 로그인 폼을 통해 전송되는 request parameter의 디폴트 name은 `username`과 `password`
- 2 :  `AntPathRequestMatcher`는 **클라이언트의 URL에 매치되는 매처 →** 클라이언트의 URL이 "`/login`"이고, HTTP Method가 `POST`일 경우 매치될 거라는 사실을 예측가능
- AntPathRequestMatcher의 객체(`DEFAULT_ANT_PATH_REQUEST_MATCHER`)는 노란 박스에 있는 4번 에서 상위 클래스인 AbstractAuthenticationProcessingFilter 클래스에 전달되어 **Filter가 구체적인 작업을 수행할지 특별한 작업 없이 다른 Filter를 호출할지 결정하는 데 사용**
- 분홍박스에 해당하는 5번의 `attemptAuthentication()` 메서드는 메서드 이름에서도 알 수 있듯이 클라이언트에서 전달한 username과 password 정보를 이용해 인증을 시도하는 메서드
    - `attemptAuthentication()` 메서드는 상위 클래스인 AbstractAuthenticationProcessingFilter의 `doFilter()` 메서드에서 호출된다.
    - HTTP Method가 POST가 아니면 Exception을 throw한다
    - username과 password 정보를 이용해 `UsernamePasswordAuthenticationToken`을 생성 ( `UsernamePasswordAuthenticationToken`은 **인증을 하기 위해 필요한 인증 토큰**이지 인증에 성공한 인증 토큰과는 상관이 없다)
- doFilter() 는 상위클래스인**AbstractAuthenticationProcessingFilter가 포함하고 있다.**

### AbstractAuthenticationProcessingFilter

 HTTP 기반의 인증 요청을 처리하지만 실질적인 **인증 시도는 하위 클래스에 맡기고**, 인증에 성공하면 **인증된 사용자의 정보를 SecurityContext에 저장**하는 역할

```java
public abstract class AbstractAuthenticationProcessingFilter extends 
	GenericFilterBean implements ApplicationEventPublisherAware, MessageSourceAware {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        this.doFilter((HttpServletRequest)request, (HttpServletResponse)response, chain);
    }
	
	// (1)
    private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
       // (1-1)  인증 처리를 해야 하는지 아니면 다음 Filter를 호출할지 여부를 결정
        if (!this.requiresAuthentication(request, response)) {
            chain.doFilter(request, response);
        } else {
            try {
                Authentication authenticationResult = this.attemptAuthentication(request, response); 
               // (1-2) 하위 클래스 (UsernamePasswordAuthenticationFilter )에서 인증을 시도해 줄 것을 요
                if (authenticationResult == null) {
                    return;
                }

                this.sessionStrategy.onAuthentication(authenticationResult, request, response);
                
                // Authentication success
                if (this.continueChainBeforeSuccessfulAuthentication) {
                    chain.doFilter(request, response);
                }

                this.successfulAuthentication(request, response, chain, authenticationResult);
                // (1-3) 인증에 성공하면 처리할 동작을 수행하기 위해 (3) successfulAuthentication() 메서드를 호출
               
                
            } catch (InternalAuthenticationServiceException var5) {
                InternalAuthenticationServiceException failed = var5;
                this.logger.error("An internal error occurred while trying to authenticate the user.", failed);
                this.unsuccessfulAuthentication(request, response, failed); 
                // (1-4) 만약 인증에 실패한다면 (4) unsuccessfulAuthentication() 메서드를 호출해 인증 실패 시 처리할 동작을 수행
	               
               
               // Authentication failed 
            } catch (AuthenticationException var6) {
                AuthenticationException ex = var6;
                this.unsuccessfulAuthentication(request, response, ex);
            }

        }
    }

		// (2) requiresAuthenticationRequestMatcher 객체를 통해 **들어오는 요청이 인증 처리를 해야 하는지 여부를 결정**
    protected boolean requiresAuthentication(HttpServletRequest request, HttpServletResponse response) {
        if (this.requiresAuthenticationRequestMatcher.matches(request)) {
            return true;
        } else {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace(LogMessage.format("Did not match request to %s", this.requiresAuthenticationRequestMatcher));
            }

            return false;
        }
    }

    public abstract Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException;

	  // (3) 인증에 성공한 이후, **SecurityContextHolder를 통해 사용자의 인증 정보를 SecurityContext에 저장한 뒤,  SecurityContext를 HttpSession에 저장**
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authResult);
        SecurityContextHolder.setContext(context);
        this.securityContextRepository.saveContext(context, request, response);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
        }

        this.rememberMeServices.loginSuccess(request, response, authResult);
        if (this.eventPublisher != null) {
            this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
        }

        this.successHandler.onAuthenticationSuccess(request, response, authResult);
    }
    
    // (4) 인증 실패시  SeucurityContext를 초기화하고, AuthenticationFailureHandler를 호출
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        SecurityContextHolder.clearContext();
        this.logger.trace("Failed to process authentication request", failed);
        this.logger.trace("Cleared SecurityContextHolder");
        this.logger.trace("Handling authentication failure");
        this.rememberMeServices.loginFail(request, response);
        this.failureHandler.onAuthenticationFailure(request, response, failed);
    }
		 ...
		 ...
}

```

- 여기서 this는 원래  `AbstractAuthenticationProcessingFilter` 클래스의 현재 인스턴스를 참조하지만, 추상클래스 이기 때문에 직접 인스턴스화 되지 않고 이를 상속받은 하위클래스의 인스턴스가 생성된다. 따라서 상속받은 하위클래스에 해당하는 `UsernamePasswordAuthenticationFilter`의 인스턴스를 참조하게 된다.
- 즉, this는 `AbstractAuthenticationProcessingFilter` 클래스의 하위클래스 (`UsernamePasswordAuthenticationFilter`)에 해당함.

### UsernamePasswordAuthenticationToken

Spring Security에서 Username/Password로 인증을 수행하기 위해 필요한 토큰이며, 또한 인증 성공 후 인증에 성공한 사용자의 인증 정보가 UsernamePasswordAuthenticationToken에 포함되어 Authentication 객체 형태로 SecurityContext에 저장된다.

```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
	...

	private final Object principal;

	private Object credentials;

  ...
  ...

  // (1) **인증에 필요한 용도의 UsernamePasswordAuthenticationToken 객체를 생성**
	public static UsernamePasswordAuthenticationToken unauthenticated(Object principal, Object credentials) {
		return new UsernamePasswordAuthenticationToken(principal, credentials);
	}

  // (2) **인증에 성공한 이후 SecurityContext에 저장될 UsernamePasswordAuthenticationToken 객체를 생성**
	public static UsernamePasswordAuthenticationToken authenticated(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		return new UsernamePasswordAuthenticationToken(principal, credentials, authorities);
	}

  ...
  ...

}

```

- UsernamePasswordAuthenticationToken은 ****AbstractAuthenticationToken을 상속하는 확장클래스이자,  Authentication 인터페이스의 메서드 일부를 구현하는 구현 클래스이다.
****

### Authentication

Spring Security에서의 인증 자체를 표현하는 인터페이스

```java
public interface Authentication extends Principal, Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	Object getCredentials();
	Object getDetails();
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}

```

- **Principal**
    - `Principal`은 사용자를 식별하는 고유 정보입니다.
    
    일반적으로 Username/Password 기반 인증에서 `Username`이 Principal이 되며, 다른 인증 방식에서는  `UserDetails`가 Principal이 됩니다.
    
    UserDetails에 대해서는 뒤에서 다시 알아보겠습니다.
    
- **Credentials**
    - 사용자 인증에 필요한 Password를 의미하며 인증이 이루어지고 난 직후, `ProviderManager`**가 해당 Credentials를 삭제합니다.**
- **Authorities**
    - `AuthenticationProvider`에 의해 부여된 사용자의 접근 권한 목록입니다.
    일반적으로 `GrantedAuthority` 인터페이스의 구현 클래스는 `SimpleGrantedAuthority`입니다.

### AuthenticationManager

증 처리를 총괄하는 매니저 역할을 하는 인터페이스

```java
public interface AuthenticationManager {

//authenticate() 메서드 하나만 정의되어 있다.
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

- **느슨한 결합**을 유지하고 있으며, **인증을 위한 실질적인 관리는 AuthenticationManager를 구현하는 구현 클래스를 통해 이루어 진다.**

### ProviderManager

AuthenticationManager 인터페이스의 구현 클래스라고 하면 일반적으로 `ProviderManager`를 가리킨다.

AuthenticationProvider를 관리하고, AuthenticationProvider에게 인증 처리를 위임하는 역할

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
  ...
  ...

  // (1) ProviderManager 클래스가 Bean으로 등록 시, List<AuthenticationProvider> **객체를 DI 받는다**는 것을 알 수있다.
	public ProviderManager(List<AuthenticationProvider> providers, AuthenticationManager parent) {
		Assert.notNull(providers, "providers list cannot be null");
		this.providers = providers;
		this.parent = parent;
		checkState();
	}

  ...
  ...

	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		int currentPosition = 0;
		int size = this.providers.size();

    // (2) DI 받은 List<AuthenticationProvider>를 이용해 (2)와 같이 for문으로 **적절한 AuthenticationProvider를 찾는다.**
		
		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
						provider.getClass().getSimpleName(), ++currentPosition, size));
			}
			try {
				result = provider.authenticate(authentication);  
				// (3) AuthenticationProvider를 찾았다면 (3)과 같이 해당 **AuthenticationProvider에게 인증 처리를 위임**
				
				
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}

		...
    ...

		if (result != null) {
			if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {
				((CredentialsContainer) result).eraseCredentials(); 
					 // (4)인증이 정상적으로 처리되었다면 **인증에 사용된 Credentials를 제거**
			}

			if (parentResult == null) {
				this.eventPublisher.publishAuthenticationSuccess(result);
			}

			return result;
		}

    ...
    ...
	}

  ...
  ...
}

```

### AuthenticationProvider

AuthenticationManager로부터 인증 처리를 위임받아 실질적인 인증 수행을 담당하는 컴포넌트

Username/Password 기반의 인증 처리는 `DaoAuthenticationProvider`가 담당하고 있으며, `DaoAuthenticationProvider`는 **UserDetailsService**로부터 전달받은 **UserDetails**를 이용해 인증을 처리한다.

```java
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider { 
					 // (1) AbstractUserDetailsAuthenticationProvider를 상속
  ...
	private PasswordEncoder passwordEncoder;

	...
  ...

  // (2) UserDetailsService로부터 UserDetails를 조회하는 역할
  //  조회된 **UserDetails는 사용자를 인증하는 데 사용될 뿐만 아니라 인증에 성공할 경우, 인증된 Authentication 객체를 생성하는 데 사용**
	@Override
	protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		prepareTimingAttackProtection();
		try {
			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username); 
					    // (2-1) UserDetails를 조회
					    
			if (loadedUser == null) {
				throw new InternalAuthenticationServiceException(
						"UserDetailsService returned null, which is an interface contract violation");
			}
			return loadedUser;
		}
		catch (UsernameNotFoundException ex) {
			mitigateAgainstTimingAttack(authentication);
			throw ex;
		}
		catch (InternalAuthenticationServiceException ex) {
			throw ex;
		}
		catch (Exception ex) {
			throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
		}
	}

  // (3) PasswordEncoder를 이용해 사용자의 패스워드를 검증하
	@Override
	protected void additionalAuthenticationChecks(UserDetails userDetails,
			UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
		if (authentication.getCredentials() == null) {
			this.logger.debug("Failed to authenticate since no credentials provided");
			throw new BadCredentialsException(this.messages
					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
		}
		String presentedPassword = authentication.getCredentials().toString();
		if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) { 
							// (3-1)  데이터베이스에서 조회한 패스워드가 일치하는지 검증하고 있는 것을 확인
							
			this.logger.debug("Failed to authenticate since password does not match stored value");
			throw new BadCredentialsException(this.messages
					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
		}
	}

  ...
  ...
}

```

- AuthenticationProvider 인터페이스의 구현 클래스는 `AbstractUserDetailsAuthenticationProvider`이고, DaoAuthenticationProvider는 `AbstractUserDetailsAuthenticationProvider`를 상속한 **확장 클래스이다.**
- ⭐ 따라서 `AbstractUserDetailsAuthenticationProvider` 추상 클래스의 `authenticate()` 메서드에서부터 실질적인 인증 처리가 시작된다
- DaoAuthenticationProvider와 AbstractUserDetailsAuthenticationProvider의 코드를 이해하기 위해서는 메서드가 호출되는 순서가 중요
    - 두 클래스가 번갈아 가면서 호출되기 때문에 로직을 이해하기 쉽지 않음
        1. `AbstractUserDetailsAuthenticationProvider`의 authenticate() 메서드 호출
        2. `DaoAuthenticationProvider`의 retrieveUser() 메서드 호출
        3. `DaoAuthenticationProvider`의 additionalAuthenticationChecks() 메서드 호출
        4. `DaoAuthenticationProvider`의 createSuccessAuthentication() 메서드 호출
        5. `AbstractUserDetailsAuthenticationProvider`의 createSuccessAuthentication() 메서드 호출
        6. 인증된 Authentication을 `ProviderManager`에게 리턴

### UserDetails

 **데이터베이스 등의 저장소에 저장된 사용자의 Username과  사용자의 자격을 증명해 주는 `크리덴셜(Credential)`인 Password 그리고 사용자의 권한 정보를 포함하는 컴포넌트이며,**  `AuthenticationProvider`는 `UserDetails`를 이용해 자격 증명을 수행

```java
public interface UserDetails extends Serializable {

	Collection<? extends GrantedAuthority> getAuthorities(); // (1) 권한 정보
	String getPassword(); // (2) 패스워드
	String getUsername(); // (3) Username

	boolean isAccountNonExpired();  // (4) 사용자 계정의 만료 여부
	boolean isAccountNonLocked();   // (5) 사용자 계정의 lock 여부
	boolean isCredentialsNonExpired(); // (6) Credentials(Password)의 만료 여부
	boolean isEnabled();               // (7) 사용자의 활성화 여부
}

```

### UserDetailsService

UserDetails를 로드(load)하는 핵심 인터페이스

```java
public interface UserDetailsService {
						// 사용자의 정보를 로드
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}

```

- 사용자의 정보를 어디에서 로드하는지는 애플리케이션에서 사용자의 정보를 어디에서 관리하고 있는지에 따라서 달라
- 사용자의 정보를 메모리에서 로드하든 데이터베이스에서 로드하든 Spring Security가 이해할 수 있는 **UserDetails**로 리턴 해주기만 하면 된다

### SecurityContext와 SecurityContextHolder

`SecurityContext`는 인증된 Authentication 객체를 저장하는 컴포넌트이고,  `SecurityContextHolder`는 SecurityContext를 관리하는 역할을 담당

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f7511b05-f7b2-446f-8b14-0575185a1028/8e9220df-02f1-4823-813e-f8e3fb085343/Untitled.png)

- `SecurityContextHolder`를 통해 인증된 Authentication을 SecurityContext에 설정할 수 있고 또한 `SecurityContextHolder`를 통해 인증된 Authentication 객체에 접근할 수 있다는 것을 의미
- ⭐ Spring Security 입장에서는 SecurityContextHolder에 의해 **SecurityContext에 값이 채워져 있다면 인증된 사용자로 간주**

```java
public class SecurityContextHolder {
  ...
  ...

  private static SecurityContextHolderStrategy strategy;  
  // (1)  SecurityContextHolder에서 사용하는 전략을 의미하며, 
  // SecurityContextHolder 기본 전략은 ThreadLocalSecurityContextHolderStrategy
	// 이 전략은 현재 실행 스레드에 SecurityContext를 연결하기 위해 ThreadLocal을 사용하는 전략
  ...

  // (2) 현재 실행 스레드에서 SecurityContext를 얻을 수 있다
	public static SecurityContext getContext() {
		return strategy.getContext();
	}
  ...

  // (3) 현재 실행 스레드에 SecurityContext를 연결
  // setContext()는 대부분 인증된 Authentication을 포함한 SecurityContext를 현재 실행 스레드에 연결하는 데 사용
	public static void setContext(SecurityContext context) {
		strategy.setContext(context);
	}
  ...
}

```

<br/>

<br/>

<br/>

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