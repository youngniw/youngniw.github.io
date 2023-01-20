---
layout: post
toc: true
title: "[Spring Security] JWT 토큰을 통한 로그인"
categories: SpringSecurity
tags: SpringSecurity, 스프링시큐리티, SpringBoot, 스프링부트, Spring, 스프링, JWT
---

## 0. 들어가기에 앞서..

이번 포스트에서는 Spring Security를 통한 JWT 토큰 이용 로그인 방법에 대해 설명하고자 합니다.

<br/>

Spring Security를 적용한 관련 데모 프로젝트 코드는 아래의 링크에서 확인할 수 있습니다.

[Spring Security Demo Project](https://github.com/youngniw/demo-spring-security)

<br/>
<hr/>

## 1. 로그인 방식 소개

본 포스트는 이전 포스트에서 설명한 4가지 로그인 방식 중, 토큰인 JWT를 사용한 인증 방식에 대해 설명한다.

- <mark>Access Token</mark>
  - 사용자의 정보가 담겨있는 토큰으로 요청을 할 시에 해당 토큰을 header에 담아 전달
  - 서버에서는 전달받은 Access Token이 유효한지 검증 후에 유효하다면 요청에 대한 응답 반환
  - 토큰 유출에 대한 위험성이 있어 만료 기한을 짧게 설정해야 함
  - 일반적으로 유효기간은 30분

- <mark>Refresh Token</mark>
  - 아무런 정보를 포함하고 있지 않은 토큰으로 Access Token의 만료 기한을 넘겼을 때, Access Token 재발급을 위해 사용됨
  - 재발급을 위해 클라이언트는 Access Token과 Refresh Token을 전달해, 서버에서는 Access Token에 담겨 있는 사용자 정보 확인 후 Refresh Token의 유효성을 검증 후 Access Token 재발급
  - Access Token 보다 만료 기한이 긺
  - 일반적으로 유효기간은 2주
  
> Access Token만 사용할 시에는 만료 기한이 짧을 경우에는 빈번한 로그인에 대한 불편함이, 만료 기한이 긴 경우에는 탈취에 대한 위험이 있기에 Refresh Token을 함께 사용하는 것이 좋다.

<br/>
<hr/>

## 2. Spring Security 관련 설정

### 2.1 라이브러리 설정

Spring Security를 활용하여 JWT 토큰을 이용한 로그인을 구현하기 위해 다음과 같이 build.gradle에 아래의 코드를 추가해야 한다.

```gradle
dependencies {
  // 스프링 시큐리티
  implementation 'org.springframework.boot:spring-boot-starter-security'

  // JWT 관련 의존성
  implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
  implementation 'io.jsonwebtoken:jjwt-impl:0.11.5'
  implementation 'io.jsonwebtoken:jjwt-jackson:0.11.5'
}
```

<br/>

<b>[ application.properties 설정 ]</b>

로깅 레벨을 위해 
`logging.level.org.springframework.security=debug 혹은 info` 
로 설정함으로써 원하는 레벨 이상의 로그를 확인하도록 한다.

또한 jwt 관련 속성 값으로 header, 비밀키, 토큰 만료 시간을 작성할 수 있다.

```
jwt.header = Authorization
jwt.secret = 알고리즘에_맞는_길이의_비밀키로_사용할_문자열
jwt.token-validity-in-seconds = 토큰만료시간(sec)
```

> 참고로 HS512 알고리즘 사용 시, Secret Key는 64Byte 이상이 되어야 함
> -> Secret Key 값은 보통 어떤 긴 문자열을 Base64와 같은 것으로 인코딩한 값을 사용

<br/>
<hr/>

### 2.2 Configuration 설정

Spring Security 사용을 위해서는 몇 가지 설정을 해야 한다. 
설정의 예시는 다음과 같다.

```java
@EnableWebSecurity
public class SecurityConfig {
  @Bean
  public WebSecurityCustomizer webSecurityCustomizer() {
    // configure Web Security
  }

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    // configure Http Security
  }
}
```

<b>[ `@EnableWebSecurity` ]</b>

- 웹 보안을 활성화
- 기본 웹 응용 프로그램 보안 구성을 완전히 끄려면 `@EnableWebSecurity`를 사용하여 빈 추가 가능

<br/>

<b>[ `WebSecurityCustomizer 사용` ]</b>

WebSecurityCustomizer를 반환하도록 하는 빈을 통해 웹에 대한 보안 설정 또한 가능하다.

- `ignoring()`: 무시할 경로 지정 가능
  - `antMatchers("경로")`

<br/>

<b>[ `SecurityFilterChain 사용` ]</b>

- 기존에는 WebSecurityConfigurerAdapter를 상속받았지만 해당 방법은 Deprecated 됨
- 대신, SecurityFilterChain을 사용하여 빈 등록 권장
- 위와 같이 빈을 등록하여 설정하기에 컴포넌트 기반의 보안 설정을 할 수 있게 됨

+ filterChain에서 HttpSecurity 파라미터를 받아 SecurityFilterChain 반환하는 빈 등록

cf. http 객체 메서드
- `authorizeRequests()`: HttpServletRequest를 사용하는 요청들에 대한 접근 제한 설정하도록 함 (세부 설정)
  
- `antMatchers(“경로 패턴”)`: 특정 경로에 대한 설정
  - `hasRole()` / `hasAnyRole()`: 시스템 상에서 특정 역할을 가진 사람만이 접근 가능
  - `hasAuthority()` / `hasAnyAuthority()`: 시스템 상에서 특정 권한을 가진 사람만이 접근 가능
  - `anonymous()`: 인증되지 않은 사용자만 가능
  - `permitAll()`: 경로에 해당 요청은 인증 없이 접근을 허용함
	- `denyAll()`: 모든 사용자의 접근 거부와 비슷한 뜻으로 access 조건을 false로 만들어 AccessDeniedException 발생
	- `authenticated()`: 경로에 해당하는 요청은 인증을 받아야 함
  
- `anyRequest()`: 나머지 경로에 대한 설정 (위의 antMatchers 와 가능 값이 동일함)

- `formLogin()`: form 태그 기반의 로그인을 지원함 
  - 별도의 로그인 페이지 제작 없이 페이지 확인 가능
  - `loginPage("로그인 페이지 URI")`: 커스텀 한 로그인 페이지 적용

<br/>

> cf. `AuthenticationManagerBuilder`: 인증에 대한 다양한 설정 생성<br/>
> ex) 메모리상에 있는 정보만을 이용 or JDBC나 LDAP 등의 정보를 이용해서 인증 처리
>
> cf. `HttpSession`: 웹과 관련된 로그인 정보에서 이용됨<br/>
> -> 세션 쿠키를 이용하기 때문에 기존 로그인 정보를 삭제하기 위해서는 브라우저를 완전히 종료하거나 세션 쿠키를 삭제해야 함
>
> cf. `@Secured("제한경로")` 를 통해 특정 경로에 접근을 제한할 수 있음
>   - 사용 시에는 config 클래스에 @EnableGlobalMethodSecurity(securedEnabled = true)를 추가해야 함
>
> cf. `exceptionHandling()`에서 예외처리 가능<br/>
> -> 그 중 `accessDeniedHandler()`에서 핸들러 등록도 가능하지만, `accessDeniedPage("페이지 경로")`를 통해 페이지 또한 설정 가능
> 
> cf. `http.logout().invalidateHttpSession(true);`를 통해 세션 무효화함으로써 자신이 로그인했던 모든 정보를 삭제할 수 있음<br/>
> -> 이때 `logout().logoutUrl("로그아웃 경로")`를 통해 로그아웃을 위한 URI 지정이 가능함

<br/>

<b>[ 예제 코드 ]</b>

```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {
  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }

  @Bean
  public WebSecurityCustomizer configure() {
    return web -> web.ignoring().antMatchers(
            "/resources/**"
    );
  }
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        // 토큰 방식을 사용하기 때문에 CSRF 설정 disable
        .csrf()
        .disable()
        .cors()

        .and()
        .formLogin().disable()
        .httpBasic().disable()
        .exceptionHandling()
        .authenticationEntryPoint(jwtAuthenticationEntryPoint)      // 인증되지 않은 사용자일 때 UnAuthorized(401) 반환
        .accessDeniedHandler(jwtAccessDeniedHandler)                // 필요 권한이 없는 사용자일 때 Forbidden(403) 반환

        // 시큐리티는 기본적으로 세션 사용, but 여기서는 세션을 사용하지 않기 때문에 세션을 Stateless로 설정
        .and()
        .sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

        // 요청 보안 설정
        .and()
        .authorizeRequests()
        .antMatchers("/login").permitAll()
        .antMatchers("/login/**").permitAll()
        .antMatchers("/login/oauth/google").permitAll()
        .antMatchers("/token/reissue").permitAll()
        .antMatchers("/signup").permitAll()
        .antMatchers("/member/**").access("hasRole('ROLE_ADMIN')")
        .anyRequest().authenticated()  // 나머지 API는 모두 인증 필요

        // JwtFilter를 addFilterBefore로 등록했던 JwtSecurityConfig 클래스 적용
        .and()
        .apply(new JwtSecurityConfig(tokenProvider));

    return http.build();
  }
}
```

<br/>
<hr/>

## 3. JWT를 활용하는 로그인

JWT를 이용한 로그인 기능을 구현하기 위해 추가적으로 구현해야 될 내용은 다음과 같다.

- JWT 관련
  - TokenProvider: 사용자 정보(Authentication)로 JWT 토큰을 만들거나 토큰을 바탕으로 사용자 정보(Authentication)를 가져옴 (토큰의 생성, 유효성 검증 등)
  - JwtFilter: Spring Request 앞단에 적용되어 토큰을 추출하고 토큰을 검증하도록 하는 Filter
  - JwtSecurityConfig: 작성한 JwtFilter를 등록
  
- Spring Security 관련
  - JwtAuthenticationEntryPoint: 인증 정보 없을 때 401 오류 발생
  - JwtAccessDeniedHandler: 접근 권한 없을 때 403 발생
  - SecurityConfig: 스프링 시큐리티에 필요한 설정
  - SecurityUtil: SecurityContext에서 전역으로 유저 정보를 제공하는 유틸 클래스
  - CustomUserDetailsService: 사용자 정보 조회

위의 모든 파일들은 @Component 어노테이션이 필요하다.

<br/>

### 3.1 TokenProvider - 토큰 생성 및 유효성 검증

- 인터페이스를 `InitializingBean` implements 함으로써 생성자 호출 시에 JWT에 필요한 값을 할당하고, 추가로 오버라이드 한 `afterPropertiesSet()` 메서드 정의
  - 빈이 생성되고 주입을 받은 뒤, afterPropertiesSet() 메서드를 통해 secret 값을 Base64로 Decode 해서 Key 변수에 할당

<b>[ 토큰 생성 ]</b>

- 토큰 생성 메서드에서는 Authentication 타입의 인수를 받아 권한을 조회함
  
  ```java
  String authorities = authentication.getAuthorities().stream()
    .map(GrantAuthority::getAuthority)
    .collect(Collectors.joining(","));
  ```
	
- 권한 조회 후에 Access Token과 Refresh Token을 생성함
  - Access Token: 사용자 정보 담음
  
  ```java
  Jwts.builder()
		.setSubject(토큰제목)
		.claim(권한담을데이터키, 데이터내용)
		.setExpiration(만료시간)
		.signWith(key, SignatureAlgorithm.알고리즘명)
		.compact();
  ```
	- Refresh Token: 사용자 정보 담음
  
  ```java
  Jwts.builder()
    .setExpiration(만료시간)
    .signWith(key, SignatureAlgorithm.알고리즘명)
    .compact();
  ```

- 토큰에 담긴 정보를 이용해 Authentication 객체 반환
  - 토큰에 담긴 정보 조회
    
    ```java
    Claims claims = Jwts.parseBuilder()
      .setSigningKey(키)
      .build()
      .parseClaimsJws(토큰 객체)
      .getBody();
    ```

	- 권한(authorities) 조회
    
    ```java
    Collection<? Extends GrantedAuthority> authorities = Arrays.stream(claims.get(데이터키).toString().split(","))
      .map(SimpleGrantedAuthority::new)
      .collect(Collectors.toList());
    ```

  - 유저 객체 생성

    ```java
    UserDetails principal = new User(유저 이름, 비밀번호, authorities);
    ```

	- 최종적으로 Authentication 객체 반환
    
    ```java
    return new UsernamePasswordAuthenticationToken(principal, token, authorities);
    ```

- 토큰 유효성 검증 메소드 (boolean 값 반환)
  
  ```java
  try { 
    Jwts.parseBuilder().setSigningKey(key).build().parseClaimJws(token);
    return true; 
  } catch (SecurityException | MalformedJwtException e) {
    log.info("잘못된 jwt 서명입니다.");
  } catch (ExpiredJwtException e) {
    log.info("만료된 jwt 토큰입니다.");
  } catch (UnsupportedJwtException e) {
    log.info("지원되지 않는 jwt 토큰입니다.");
  } catch (IllegalArgumentException e) {
    log.info("jwt 토큰이 잘못되었습니다.");
  }
  return false;
  ```

<br/>
<hr/>

### 3.2 JwtFilter - 요청 시 토큰 확인 필터

- `GenericFilterBean`을 상속하거나 `OncePerRequestFilter`를 상속하여 Filter를 정의
- 실제 필터링 로직을 정의함 (토큰의 인증 정보를 SecurityContext에 저장하는 역할 수행)

1. `GenericFilterBean` 상속
   - `doFilter(ServletRequest, ServletResponse, FilterChain)` 메서드 오버라이드
   + Request Header에서 토큰을 꺼냄 (토큰 유효성 검증)
   + 토큰으로 TokenProvider의 getAuthentication을 호출해 인증(Authentication) 반환
   + 정상 토큰일 때 SecurityContext에 Authentication 객체 저장

2. `OncePerRequestFilter` 상속
   - `doFilterInternal(HttpServletRequest, HttpServletResponse, FilterChain)` 메서드 오버라이드
   + 위와 동일한 내용

- 모든 request 요청은 토큰 정보가 없거나 유효하지 않으면 정상적으로 수행되지 않게 하기 위해 JwtFilter 필터를 거침
	- 요청이 정상적으로 Controller에 도달했으면 SecurityContext에 회원 ID가 존재함
	- but, DB에서 직접 조회한 것이 아니라 Access Token 내의 정보를 꺼낸 것이기 때문에 예외 상황은 Service에서 작성해야 함

- 추가적으로 `shouldNotFilter` 메서드를 오버라이드함으로써 경로를 사전 확인하여 doFilterInternal 내용 즉 토큰 검증 수행 여부를 설정할 수 있음

  -예제 코드
  
  ```java
  private static final List<String> EXCLUDE_URL_LIST = List.of(
    "/auth/token/reissue",
    "/user/login-id",
    "/user/password"
  );

  // jwt 토큰 검사 안하는 경로 사전 확인
  @Override
  protected boolean shouldNotFilter(@NonNull HttpServletRequest request) throws ServletException {
    return EXCLUDE_URL_LIST.stream().anyMatch(excludeUrl -> excludeUrl.equalsIgnoreCase(request.getServletPath()));
  }
  ```

<br/>
<hr/>

### 3.3 JwtSecurityConfig - JWT 관련 설정 파일

- 직접 만든 TokenProvider와 JwtFilter를 SecurityConfig에 적용할 때 사용함
- `SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity>` 상속(extends)
	- `configure(HttpSecurity http)`를 오버라이드함으로써 필터 등록
  	- JwtFilter 객체 생성
    - http.`addFilterBefore(필터 객체, 뒤에 올 필터)`: 지정된 필터 앞에 커스텀 필터를 추가

cf. Security를 정의하는 설정 파일에서도 등록할 수도 있음

<br/>
<hr/>

### 3.4 JwtAuthenticationEntryPoint - 인증되지 않은 유저 접근 처리

- 유효한 자격 증명을 제공하지 않고 접근하려 할 때, 상태 코드로 401 Unauthorized 반환 
  - ex. 유저정보 없이 접근, 토큰이 유효하지 않음
- `AuthenticationEntryPoint` 인터페이스 구현(implements)
	- `commence(HttpServletRequest, HttpServletResponse, AuthenticationException)`를 오버라이드 함으로써 인증 과정 중 발생한 예외에 대한 응답 처리
	- 응답 HTTP 상태가 401이 발생할 만한 에러가 발생할 경우 해당 로직을 타게 되어 commence 메서드 실행

<br/>

<b>[ 예제 코드 ]</b>

```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
  @Override
  public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
    // response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "UnAuthorized");

    // 요청에 커스텀한 오류 번호를 전달해 발생하는 예외 이유마다의 응답 메시지를 다르게 함
    Integer exceptionCode = (Integer) request.getAttribute("exception");

    if (exceptionCode == ErrorCode.EXPIRED_JWT_TOKEN.getErrorId())    // 유효기간 만료 토큰
      setResponse(response, ErrorCode.EXPIRED_JWT_TOKEN);
    else
      setResponse(response, ErrorCode.INVALID_JWT_TOKEN);
  }

  // 예외에 대한 커스텀한 응답 메시지 형식을 반환 (401 오류 반환)
  private void setResponse(HttpServletResponse response, ErrorCode errorCode) throws IOException {
    ObjectMapper mapper = new ObjectMapper();

    response.setContentType("application/json;charset=UTF-8");
    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    response.getWriter().write(mapper.writeValueAsString(ErrorResponse.fail(errorCode)));
  }
}
```

<br/>
<hr/>

### 3.5 JwtAccessDeniedHandler - 권한이 없는 유저 접근 처리

- 인증된 유저지만 필요한 권한이 존재하지 않은 경우에는 상태 코드로 403 Forbidden 반환
- `AccessDeniedHandler` 인터페이스 구현(implements)
	- `handle(HttpServletRequest, HttpServletResponse, AccessDeniedException)`를 오버라이드 함으로써 403을 반환한 예외에 대한 응답 처리

<b>[ 예제 코드 ]</b>

```java
@Component
public class JwtAccessDeniedHandler implements AccessDeniedHandler {
  @Override
  public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
    // response.sendError(HttpServletResponse.SC_FORBIDDEN, "Forbidden");

    setResponse(response, ErrorCode.NO_AUTHORITIES_TOKEN);
  }

  // 예외에 대한 권한이 없는 토큰이라는 메시지를 담은 커스텀한 응답을 반환 (403 오류 반환)
  private void setResponse(HttpServletResponse response, ErrorCode errorCode) throws IOException {
    ObjectMapper mapper = new ObjectMapper();

    response.setContentType("application/json;charset=UTF-8");
    response.setStatus(HttpServletResponse.SC_FORBIDDEN);
    response.getWriter().write(mapper.writeValueAsString(ErrorResponse.fail(errorCode)));
  }
}
```

<br/>
<hr/>

### 3.6 SecurityUtil - 유틸리티 메서드 정의

- 간단한 유틸리티 메서드를 정의할 수 있음
- SecurityContext의 Authentication 객체를 이용해 username을 리턴하는 메서드를 정의할 수 있음

cf. **Security Context에 Authentication 객체가 저장되는 시점**: JwtFilter의 `doFilterInternal` 메서드에서 request가 들어올 때

<br/>
<hr/>

### 3.7 CustomUserDetailsService - 사용자(인증 주체) 정보 조회

- `UserDetailsService` 인터페이스 구현(implements)
  - `UserDetailsService` 인터페이스: Spring Security에서 사용자(인증 주체)의 정보를 가져오는 인터페이스
  - `UserDetails` 인터페이스: Spring Security에서 사용자의 정보를 담음

- `loadUsersByUsername(String username)` 메서드 오버라이딩 
  - 반환 타입: `UserDetails`
  - 계정 아이디 혹은 일련번호를 의미하는 매개변수 username의 값으로 사용자 정보 조회
  - 보통 회원 Repository에서 username 값에 해당하는 사용자 정보를 조회해 `User`타입으로 변환
  - 이때 username에 해당하는 사용자가 없을 경우에는 `UsernameNotFoundException` 예외 반환

<br/>

<b>[ Authentication Manager ]</b>

- 인증 매니저(Authentication Manager)
  - 인증에 대한 실제적인 처리하는 방법을 정의
  - 결과적으로 인증과 관련된 모든 정보를 UserDetails 타입으로 반환
  
- UserDetailService: 사용자(인증 주체)의 정보를 가져옴
  - 인증되는 방식을 다르게 하고 싶다면 UserDetailsService 인터페이스 구현 후 인증 매니저에 연결하면 됨
  - `loadUsersByUsername()` 메서드를 구현함으로써 `UserDetails`를 반환 (인증의 주체에 대한 정보를 가져옴)

<br/>

<b>[ UserDetails ]</b>

- 기본적인 사용자 계정과 같은 정보와 더불어 사용자가 어떤 권한(Authority)들을 가지고 있는지 Collection 타입으로 가지고 있음
- "사용자의 계정 정보 + 사용자가 가진 권한 정보"의 묶음
- 설계 시에는 id와 같은 식별 데이터를 이용하지만, 스프링 시큐리티에서는 username라는 용어 사용

<br/>

<b>[ 구조 ]</b>

* AuthenticationManager(인증 매니저)
  * AuthenticationManagerBuilder
  * Authentication
* UserDetailsService 인터페이스
  * UserDetailsManager 인터페이스
* UserDetails 인터페이스
  * User 클래스

<br/>

<b>[ 예제 코드 ]</b>

```java
@RequiredArgsConstructor
@Service
public class CustomUserDetailsService implements UserDetailsService {
  private final MemberRepository memberRepository;

  @Override
  @Transactional
  public UserDetails loadUserByUsername(String memberId) throws UsernameNotFoundException {
    // 인증 처리 방식 정의
    // username: memberId
    return memberRepository.findById(Long.parseLong(memberId))
        .map(member -> createUser(memberId, member))
        .orElseThrow(() -> new UsernameNotFoundException("등록되지 않은 사용자입니다."));
}

  private User createUser(String username, Member member) {
    Set<GrantedAuthority> grantedAuthorities = new HashSet<>();
    grantedAuthorities.add(new SimpleGrantedAuthority(member.getMemberRole().getValue()));

    return new User(username, member.getLoginPassword(), grantedAuthorities);
  }
}
```

<br/>
<hr/>

### 3.8 그 이외의 로그인 MVC

<b>[ 로그인/회원 Controller ]</b>

- Configuration 파일 외에도 요청에 대한 권한 확인 가능
- @PreAuthorize("hasAnyRole('권한명','권한명')") 선언: 권한명으로 정의한 권한들을 갖고 있지 않은 토큰을 입력 시에는 403 Forbidden 발생

<br/>

<b>[ 로그인/회원 Service ]</b>

TokenProvider와 AuthenticationManager, 그리고 PasswordEncoder를 주입받아 로그인 및 회원가입 기능을 수행하는 로직을 처리한다.

1. 회원가입
   - Repository에 회원 정보를 저장
   - 이때 비밀번호는 `PasswordEncoder`를 통해 전달받은 비밀번호 값을 encode 하여 저장함으로써 암호화되어 DB에 저장될 수 있게 함
  
2. 로그인
   1. Repository에서 전달받은 로그인 아이디와 비밀번호에 맞는 회원 엔티티 객체 조회 
   2. Authentication Manager의 `authenticate()` 메서드를 통해 사용자 정보 인증
      - `Authentication authentication = authenticationManager.authenticate(인증할 토큰)`
      - 이때 인수로 username으로 사용하는 아이디 혹은 일련번호와 credential 값인 비밀번호로 UsernamePasswordAuthenticationToken 객체(인증 토큰) 생성
      - `authenticate` 메서드가 실행될 때, `CustomUserDetailsService`의 `loadUsersByUsername` 메서드가 실행됨 (사용자 User 정보 조회)
   3. 인증이 완료되면 사용자 정보를 가진 Authentication 객체를 SecurityContextHolder에 저장
   4. 인증 시에 비밀번호가 옳지 않은 등의 AuthenticationException 발생 시, 이에 대한 처리 (오류 상태 코드 반환)
   5. Access Token과 Refresh Token 생성 (Refresh Token을 저장소에 따로 저장)
   6. 클라이언트에게 응답으로 토큰 정보 반환
   
<br/>

<b>[ PasswordEncoder ]</b>

- 사용자의 패스워드를 암호화 처리
- PasswordEncoder 인터페이스를 구현한 AbstractPasswordEncoder, BCryptPasswordEncoder 등을 이용함으로써 암호화 처리
  - BCryptPasswordEncoder: 단방향 해시 알고리즘으로 구현 (추천!)
  - StandardPasswordEncoder: SHA 해시 알고리즘

- 적용 방식
  1. 로그인 관련 인터페이스 구현 클래스 작성
  2. 시큐리티 설정 추가 (별도의 @Bean으로 생성될 수 있도록 처리)
  3. 관련 컨트롤러나 서비스와 연동

<br/>
<hr/>

지금까지 Spring Security를 사용한 JWT 로그인 인증에 대해 설명했습니다.

감사합니다:)
