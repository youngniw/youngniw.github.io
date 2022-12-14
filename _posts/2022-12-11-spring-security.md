---
layout: post
toc: true
title: "[Spring Security] Spring Security 간단 정리"
categories: Spring Security
tags: SpringSecurity, 스프링시큐리티, SpringBoot, 스프링부트, Spring, 스프링
---

## 0. 들어가기에 앞서..

이번 포스트에서는 Spring Security에 대해 간략하게 설명하고자 합니다.

<br/>
<hr/>

## 1. Spring Security 소개

- Spring 기반의 애플리케이션을 위한 인증, 권한 부여 및 기타 보안 기능을 제공하는 프레임워크
- Spring Security는 Filter 기반으로 동작하기에 Spring MVC와 분리되어 동작
- 기본적으로 `세션 - 쿠키` 방식으로 인증

<br/>

### 1.1 관련 용어 소개

- <mark>인증 (authentication)</mark>: 식별 가능한 사용자의 정보를 이용해 사용자의 신원 확인
- <mark>인가 (authorization)</mark>: 사용자가 특정 서비스나 페이지에 접근할 수 있는 권한 확인 및 허락
  - 인증만으로 서비스를 제공하면 모든 사용자가 똑같은 서비스를 누릴 수 있게 되기에 문제 발생
  - 따라서 스프링 시큐리티는 기본적으로 인증 후에 인가 절차 진행
- <mark>접근 주체 (principal)</mark>: 보호된 리소스에 접근하는 사용자 (ex. 아이디, 이메일 등)
- <mark>신용 증명 정보 (credential)</mark>: 리소스에 접근하는 사용자의 비밀번호
- <mark>권한 (authority)</mark>: 인증된 사용자가 특정 서비스나 페이지에 접근할 수 있도록 부여하는 것
- <mark>역할 (role)</mark>: 시스템에서 사용하는 사용자의 역할 (권한 or 권한을 담는 컨테이너로 사용)

<br/>
<hr/>

### 1.2 동작 방식

![spring-security-architecture](https://user-images.githubusercontent.com/78736070/207508172-bd3ff4c5-2941-4d96-bfe4-e86e18daec89.png)

> 포괄적 순서: AuthenticationFilter <-<small>`UsernamePasswordAuthenticationToken`</small>-> AuthenticationManager (ProviderManager) <-> AuthenticationProvider
> 
> AuthenticaionProvider 내의 처리 과정: UserDetailsService <-> UserDetails(User)
> 
> 마지막 순서: AuthenticationFilter -<small>`Authentication`</small>-> SecurityContext

<br/>

1. 클라이언트가 사용자의 정보(아이디, 비밀번호)를 담아 HTTP Request를 통해 인증 요청
2. 필터 체인 중 `AuthenticationFilter`가 사용자의 정보를 토대로 인증용 객체인 `UsernamePasswordAuthenticationToken` (Authentication) 생성
3. `AuthenticationManager`에게 `UsernamePasswordAuthenticationToken` 객체를 전달하여 인증을 진행하도록 위임
4. `AuthenticationManager`가 인증을 처리할 `AuthenticationProvider`에게 `UsernamePasswordAuthenticationToken`를 인수로 전달해 `authenticate` 메서드를 통해 인증을 진행
5. 인증 진행 시, `UserDetailsService`의 `loadUserByUsername` 메서드가 실행되어 DB에 있는 사용자의 정보와 클라이언트로부터 전달받은 사용자 정보 비교 후 사용자 정보인 `UserDetails` 객체 반환
6. `AuthenticationProvider`는 반환된 `UserDetails` 객체를 넘겨받고 클라이언트로부터 넘겨받은 사용자 정보와 비교
7. 인증이 완료되면 사용자 정보(principal)와 권한을 담은 `Authentication` 객체를 `AuthenticationManager`로 반환
8. `AuthenticationManager`에서 `AuthenticationFilter`로 `Authentication` 객체 전달
9. Authentication을 `SecurityContextHolder` 내의 `SecurityContext`에 저장 후 AuthenticationSuccessHandler 실행 (실패: AuthenticationFailureHandler 실행)

<br/>

cf. `AuthenticationFilter`: 설정된 로그인 URL로 오는 요청을 감시하고 사용자의 인증을 처리 (AuthenticationManager를 통한 인증 실행)

cf. `UsernamePasswordAuthenticationToken`: `Authentication` 인터페이스의 구현체

cf. `ProviderManager`: `AuthenticationManager`의 구현체로, AuthenticationProvider에게 인증을 위임하고 인증을 처리할 수 있는 AuthenticationProvider 객체가 인증을 시도하여 성공 시에 인증이 되었다고 알려줌

cf. `authenticate` 메서드
  - 유효한 접근 주체인 경우: `Authentication` 반환 (authenticated = true)
  - 유효하지 않은 접근 주체인 경우: `AuthenticationException` 예외 발생
  - 결정할 수 없는 경우: null

cf. `User`: `UserDetails`의 구현체

<br/>

<b>[ Authentication ]</b>

`Authentication`는 접근하는 주체의 정보와 권한을 담는 인터페이스이다.

모든 접근 주체는 `Authentication`을 생성하여 `SecurityContext`에 저장된다.

`Authentication`는 SecurityContextHolder 내의 SecurityContext에 접근해 조회할 수 있다.

```java
public interface Authentication extends Principal, Serializable {
  Collection<? extends GrantedAuthority> getAuthorities();  // 접근 주체의 권한
  Object getCredentials();                                  // 주로 비밀번호
  Object getDetails();
  Object getPrincipal();                                    // 주로 아이디 혹은 비밀번호

  boolean isAuthenticated();                                // 인증 여부
  
  void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

<br/>

<b>[ UserDetails ]</b>

`UserDetails`는 사용자의 정보를 담는 인터페이스이다.

인터페이스 구현 시에 다른 정보를 추가할 수 있어서 별도의 회원 클래스를 만들지 않고 사용할 수 있다.

추가된 필드들은 getter와 setter를 만들어 사용하면 된다.

```java
public interface UserDetails extends Serializable {
  Collection<? extends GrantedAuthority> getAuthorities();
  String getPassword();
  String getUsername();

  boolean isAccountNonExpired();
  boolean isAccountNonLocked();
  boolean isCredentialsNonExpired();
  boolean isEnabled();
}
```

<br/>
<hr/>

### 참고. 로그인 방식

본격적으로 구현 방법에 대해 설명하기에 앞서, 
로그인은 다음의 4가지 경우로 이루어진다.

- 세션 방식
  - HttpSession 이용
  - 세션 정보를 클라이언트가 아닌 서버에서 보관하고, 클라이언트는 세션 키(ID)만을 이용하기에 안전하고 접근이 용이함
  - 서비스 요청 시 서버에서는 세션을 조회한 후 작업을 처리함
  - but, 브라우저 종료 시에는 사용할 수 없기에 불편함이 존재

- 쿠키 이용 방식
  - 서버는 일정한 정보를 담은 쿠키를 전송하고, 클라이언트에서는 서버에 접근할 때 해당 쿠키를 담아 전송
  - 이때 서버에서는 쿠키에 유효기간을 지정 가능하기에 브라우저가 종료되어도 이후 접근을 할 때에 정보 유지 가능

- 토큰 인증 방식
  - 로그인을 통해 서버에서 계정 정보 검증 후 사용자에게 토큰 발급
  - 클라이언트는 발급받은 토큰을 요청 시에 헤더(header)에 담아 서버에 전달 후, 서버에서는 해당 토큰을 검증 후 응답 반환
  - ex) JWT를 이용한 로그인 방식

- OAuth 이용 방식
  - 다른 웹 사이트 상의 회원 정보로 서비스의 접근 권한 부여
  - 외부 계정을 기반으로 인증(로그인 및 회원가입)을 통해 외부 서비스로부터 정보를 안전하게 넘겨받음
  - 별도의 추가적인 정보 입력 없이 간단하고 빠르게 로그인 가능
  - but, 사용자의 필요 정보를 얻기 위해 추가적으로 요청이 필요하기에 불편함이 존재

<br/>
<hr/>

<b>[ 참고 자료 ]</b>

- [https://spring.io/guides/topicals/spring-security-architecture](https://spring.io/guides/topicals/spring-security-architecture)

<br/>

지금까지 Spring Security를 간략하게 소개 및 동작 방식에 대해 설명하였습니다.<br/>
다음 포스트에서는 SpringBoot 환경에서 Spring Security를 적용한 JWT을 통한 로그인에 대해 설명하겠습니다.

감사합니다:)
