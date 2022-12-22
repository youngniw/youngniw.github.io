---
layout: post
toc: true
title: "[SpringBoot] Redis 사용하기"
categories: SpringBoot
tags: SpringBoot, 스프링부트, Redis
---

## 0. 들어가기에 앞서..

이번 포스트에서는 Spring Boot에서 Redis를 사용하는 방법에 대해 설명하겠습니다.

<br/>

SpringBoot에서 Redis를 사용한 관련 데모 프로젝트 코드는 아래의 링크에서 확인할 수 있습니다.

[Redis In Spring Boot Demo Project](https://github.com/youngniw/demo-redis)

<br/>
<hr/>

## 1. Spring Boot에서 Redis 사용하기

### 1.1 라이브러리 설정

Redis를 사용하기 위해 다음과 같이 build.gradle에 아래의 코드를 추가해야 한다.

```gradle
dependencies {
  implementation("org.springframework.boot:spring-boot-starter-redis")
  implementation("org.springframework.session:spring-session-data-redis")
}
```

- `spring-boot-starter-data-redis`는 `Lettuce`와 `Jedis`라는 클라이언트 라이브러리 제공
  - 기본적으로 Redis 클라이언트는 `Lettuce` 이용
  - `Lettuce` 모듈을 별도로 추가하지 않아도 `spring-boot-starter-data-redis`을 통해 `Lettuce` 관련 모듈 사용 가능
 
  - `Jedis`는 파이프라인을 제외하고는 모두 동기식 (스레드에 안전 X)
  - 하지만 `Lettuce`는 동기와 비동기 모두 지원하기에 Lettuce를 보통 사용
    - netty 라이브러리 위에 구축되었고, connection 인스턴스(StatefulRedisConnection)를 여러 스레드에서 공유가 가능하기 때문에 스레드에 안전함
  - 속도 및 성능: `Lettuce` > `Jedis`

- Session 데이터를 Redis에 저장하기 위해서는 `spring-session-data-redis` 의존성 추가해야 함
  - Config 파일 혹은 main 메서드가 담긴 Application 클래스에 `@EnableRedisHttpSession` 어노테이션 추가
	
	[ `@EnableRedisHttpSession` ]
	- 서블릿의 Filter 인터페이스를 구현(implements)한 `springSessionRepositoryFilter`을 스프링 빈으로 자동으로 등록
	- Spring Session 모듈에 있는 `AbstractHttpSessionApplicationInitializer`가 Filter를 서블릿 컨텍스트에 추가함으로써 모든 서블릿 요청에 Filter가 작동

<br/>

<b>[ application.properties 설정 ]</b>

다음과 같이 Redis 서버 주소와 포트 번호를 설정할 수 있다. 

```
spring.redis.host=레디스_서버_주소 (default: localhost)
spring.redis.port=서버_포트번호 (default: 6379)
spring.redis.database=사용할_데이터베이스_번호 (default: 0)

# 세션 저장 방식
spring.session.store-type=redis
```

<br/>
<hr/>

### 1.2 Redis Config 파일 설정

- Redis Repository를 사용할 경우에는 `@EnableRedisRepositories` 선언
  - 이때, `enableKeyspaceEvents` 값을 `RedisKeyValueAdapter.EnableKeyspaceEvents.ON_STARTUP`로 설정함으로써 RedisRepository 사용할 때 만료에 대한 인덱스 정리를 위한 키스페이스 이벤트를 수신하여 만료 감지

- RedisConnectionFactory 빈 등록
  - Redis 서버와의 통신을 위한 설정 및 연결
  - Jedis와 Lettuce 등의 클라이언트 라이브러리를 설정
  - 예제 코드
  
    ```java
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
      LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisHost, redisPort);
      lettuceConnectionFactory.setDatabase(redisDatabase);

      return lettuceConnectionFactory;
    }
    ```

- `RedisTemplate` 혹은 `StringRedisTemplate` 빈 등록
  - `RedisTemplate`
    - defaultSerializer: `JdkSerializationRedisSerializer`
    - `RedisTemplate` 빈은 객체 Object를 Redis에 저장하는 경우에 사용
  
  - `StringRedisTemplate`
    - defaultSerializer: `StringRedisSerializer`
    - `StringRedisTemplate` 빈은 일반적인 문자열 String 값을 key, value로 사용하는 경우에 사용

  > 자바에서 기본적으로 Redis는 `RedisTemplate`, `StringRedisTemplate` (reactive의 경우 `ReactiveRedisTemplate`, `ReactiveStringRedisTemplate`)을 호출하여 사용

  > 참고로, `Jackson2RedisSerializer` 클래스는 Jackson을 활용해 객체 등의 데이터를 JSON의 형태로 바꿈으로써 Redis 내의 데이터를 읽고 쓸 수 있게 함
  
  - 예제 코드
    
    ```java
    // RedisTemplate 이용 시
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
      RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
      redisTemplate.setKeySerializer(new StringRedisSerializer());
      redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer(new ObjectMapper()));
      redisTemplate.setValueSerializer(new StringRedisSerializer());
      redisTemplate.setHashKeySerializer(new StringRedisSerializer());
      redisTemplate.setHashValueSerializer(new StringRedisSerializer());
      redisTemplate.setConnectionFactory(redisConnectionFactory);

      return redisTemplate;
    }
    ```

<br/>
<hr/>

### 1.3 RedisTemplate의 자료구조 활용

- 다양한 자료구조: `String`, `List`, `Hash`, `Set`, `Sorted Set` 등

- 다양한 명령을 지원하기 위한 `opsFor___` 메서드 제공
- 해당 메서드 호출 시, 각 Redis 명령에 대응되는 객체 반환

|RedisTemplate 메서드|Operations|
|--|--|
|`opsForValue()`|ValueOperations|
|`opsForHash()`|HashOperations|
|`opsForList()`|ListOperations|
|`opsForSet()`|SetOperations|
|`opsForZSet()`|ZSetOperations|
|`opsForStream()`|StreamOperations|
|`opsForGeo()`|GeoOperations|
|`opsForHyperLogLog()`|HyperLogLogOperations|
|`opsForCluster()`|ClusterOperations|

cf. 데이터 중복 저장 시에 대한 저장 방법
  - `String`: 덮어쓰기
  - `List`: 끝에 데이터 추가
  - `Hash`: 덮어쓰기
  - `Set`: 중복된 값이 있으면 저장하지 않음, 중복되지 않으면 순서 상관없이 저장
  - `Sorted Set`: 중복된 값이 있으면 score만 업데이트하여 오름차순 정렬, 중복되지 않으면 score 오름차순으로 저장

<br/>

<b>[ Sorted Set ]</b>

- 자료구조: Struct.SortedSet
- 중복을 허용하지 않고 정렬
- 정렬의 기준: score 값

<br/>
<hr/>

### 1.4 Redis Repositories 사용

Redis의 `Hash`는 Object에 대응하는 형태로 구성하기에 적절하기 때문에 Redis Repositories를 사용한 도메인 객체 저장 형식은 `Hash`를 기본적으로 사용한다.

따라서 도메인 객체를 변환 및 저장하는 경우에 커스텀 한 매핑 전략을 사용하고 `@Indexed`를 이용함으로써 별도의 추가 인덱스(Secondary Index)를 설정할 수 있다.

만약 Object 객체와 Redis Hash 변환 및 처리를 위해 `Redis Repositories`를 사용하지 않고 `RedisTemplate`만 활용하고자 한다면, application.properties에 다음과 같이 비활성화 처리를 하면 된다.

```
spring.data.redis.repositories.enabled=false
```

- 만약 `Redis Repositories` 사용 시,
  - Config 파일에서 `@EnableRedisRepositories` 선언
  - `@RedisHash`를 이용한 도메인 객체 생성
  - 관련 레포지토리(Repository) 인터페이스 생성(`extends CrudRepository<도메인객체명, String>`)
  - 서비스에서 레포지토리의 메서드를 호출함으로써 조회, 저장, 삭제 등의 기능(`save()`, `findOne()`, `count()`, `delete()` 메서드) 수행 가능

즉, Redis Repositories는 Spring Data JPA를 사용해 객체를 릴레이션으로 매핑하는 것과 유사한 방법이다.

<br/>

<b>[ 도메인 객체 ]</b>

- `@RedisHash("Key 이름")`: 도메인(엔티티와 동일한 기능) 생성 (`@Id`는 SQL의 Id와 동일한 기능)
- `@Id`: ID 필드 지정
  - Redis에 들어가는 key 값: "`@RedisHash`의 value 값 + `@Id`가 선언된 멤버 변수명"
  - `@RedisHash`의 value 값이 없는 경우, `getClass().getName()`의 반환값 사용
- `@Indexed`: 조회를 위한 Secondary Index 설정
- `@TimeToLive`: 만료 시간 필드로 설정할 수 있으며, 옵션으로 시간 단위인 unit을 설정할 수 있음

<b>[ 예제 코드 ]</b>

아래의 코드는 Redis 내에 저장할 도메인 객체를 정의한 것이다.

```java
@RedisHash(value = "refreshToken")
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class RefreshToken {
  @Id
  private Long refreshTokenId;  // 토큰 일련번호 (키 값)

  @Indexed
  private Long userId;    // 사용자 일련번호

  private String value;   // 토큰 값

  private LocalDateTime createdAt;

  @TimeToLive(unit = TimeUnit.SECONDS)
  private long expired;   // 만료 시간 (단위: sec, 설정: 7일)
}
```

아래의 코드는 Repository를 구현한 것이다.

```java
public interface RedisRefreshTokenRepository extends CrudRepository<RefreshToken, Long> {
  Optional<RefreshToken> findByUserId(Long userId);
}
```

<br/>
<hr/>

지금까지 Spring Boot에서 Redis를 사용하는 방법에 대해 설명하였습니다.

감사합니다:)
