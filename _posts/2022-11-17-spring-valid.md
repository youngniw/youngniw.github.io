---
layout: post
toc: true
title: "[Spring] @Valid 정리"
categories: 스프링
tags: 스프링, 스프링부트
---

## 0. 들어가기에 앞서..
해당 포스트는 현재 인턴으로 참여하고 있는 프로젝트 진행 중 추가로 학습하게 됨으로써 기억하기 위해 작성한 것입니다.

<br/>
<hr/>

## 1. @Valid
<br/>
SpringBoot에서는 어노테이션을 통해 객체의 필드 값 검증이 가능하다.

`@Valid` 는 빈 검증기(Bean Validator)를 이용해 객체의 제약 조건을 검증하도록 지시하는 어노테이션이다.

특히 RestController에서 @RequestBody 객체를 클라이언트로부터 받을 때, 해당 객체의 값을 검증할 수 있는 방법에 대해 소개한다.

<br/>

### 1.1 @Valid 사용 방법

<b>[ 라이브러리 ]</b>

Valid 어노테이션을 사용하기 위해 다음과 같이 build.gradle에 아래의 코드를 추가해야 한다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

<b>[ 예시 코드 ]</b>

다음은 @Valid를 사용한 예시 코드로, 로그인 시 클라이언트에서 전달하는 정보를 검증하는 코드이다.

- 컨트롤러

```java
import javax.validation.Valid;

@Slf4j
@RestController
public class AuthController {
	@PostMapping(value = "/login")
	public ResponseEntity<String> login(@Valid @RequestBody LoginDto loginDto) {
		log.info("loginDto: {}", loginDto.toString());
		return ResponseEntity.ok().body("로그인 정보 검증 성공");
	}
}
```
<br/>

- DTO

```java
@ToString
@Getter
public class LoginDto {
	@NotBlank
	private String loginId;

	@NotNull
	private String loginPassword;
}
```
이와 같이 객체를 정의할 때, 각 필드에 맞는 검증 어노테이션을 사용하면 된다.

|어노테이션|설명|
|--|--|
|`@NotBlank`|인자로 들어온 필드 값에 null 값을 허용하지 않을 뿐만 아니라, 공백이 아닌 길이가 0보다 큰 문자열|
|`@NotNull`|인자로 들어온 필드 값에 null 값을 허용하지 않음|

객체 정의 시에 이와 같이 검증 어노테이션을 쓰면, 객체 단계에서 오류 찾을 수 있다.

<br/>
<hr/>

### 1.2 예외 처리 핸들러
`@Valid`로 객체의 검증 상황에서 발생 가능한 Bad Request에 대해 커스텀 하게 예외 처리를 진행할 수 있다.

- 잘못된 객체의 값이 주어졌을 때에는 MethodArgumentNotValidException 발생
- 해당 예외를 @ControllerAdvice를 이용해 Controller 단의 예외 처리 진행 가능
- MethodArgumentNotValidException에 대한 @ExceptionHandler 어노테이션을 지정해 커스텀 가능

<b>[ 예시 코드 ]</b>

```java
@RestControllerAdvice(annotations = RestController.class)
public class ResponseExceptionHandler {
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
	protected ResponseEntity<Map<String, String>> badRequestException(MethodArgumentNotValidException exception) {
   		Map<String, String> errors = new HashMap<>();

    		ex.getBindingResult().getAllErrors().forEach((error) -> {
        		String fieldName = ((FieldError) error).getField();
       			String msg = error.getDefaultMessage();
        		errors.put(fieldName, msg);
    		});

    		return ResponseEntity.badRequest().body(errors);
	}
}
```
유효성 검사 후 유효하지 않은 필드의 이름과 오류 메시지를 반환할 수 있다.

<br/>
<hr/>

### 1.3 유효성 검사 어노테이션
<b>[ 변수 사용 시 주의사항 ]</b>

1. 문자열 관련 유효성 검증
- `@NotNull` : null이 아닌 값 (empty는 가능)
- `@NotEmpty` : null도 아니면서 크기 및 길이가 0보다 커야 함 (빈 문자열이 아니어야 함)
  - type: CharSequence, Collection, Map, 혹은 Array
- `@NotBlank` : 문자열에 한하며, null이 아니면서 공백이 아닌 문자를 하나 이상 포함
- `@Null` : null 값

    cf. 가능 속성: message(담을 오류 메시지), groups, payload

<br/>

2. 불린 값에 대한 검증
- `@AssertTrue` : 값이 항상 true 여야 함
- `@AssertFalse` : 값이 항상 false 여야 함

    cf. null 값은 유효하다고 판단됨

<br/>

3. 최대/최소에 대한 검증
- `@DecimalMax` : 지정된 최댓값(String)보다 작거나 같아야 함
- `@DecimalMin` : 지정된 최솟값(String)보다 크거나 같아야 함

    cf. 가능 속성: value(지정할 값), inclusive(등호 포함 여부), message(담을 오류 메시지), groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

- `@Max` : 지정된 최댓값(Integer)보다 작거나 같아야 함
- `@Min` : 지정된 최솟값(Integer)보다 크거나 같아야 함

    cf. 가능 속성: value(지정할 값), message(담을 오류 메시지), groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

<br/>

4. 자릿수 범위 검증
- `@Digits` : 허용되는 범위 내의 숫자

    cf. 가능 속성: integer(최대 정수 자릿수), fraction(최대 소수 자릿수), message, groups, payload

<br/>

5. 크기 검증
- `@Size`: 크기가 지정된 값 사이에 있어야 함
  - type: type: CharSequence, Collection, Map, 혹은 Array

    cf. 가능 속성: min(최소 크기), max(최대 크기), message, groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

<br/>

6. 값의 범위에 대한 검증
- `@Positive` : 양수의 값이어야 함
- `@PositiveOrZero` : 양수 혹은 0이어야 함
- `@Negative` : 음수의 값이어야 함
- `@NegativeOrZero` : 음수 혹은 0이어야 함

    cf. 가능 속성: message(담을 오류 메시지), groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

<br/>

7. 시간 값에 대한 검증
- `@Future` : 현재의 시간보다 미래의 날짜 및 시간이어야 함
- `@FutureOrPresent` : 현재의 시간이거나 미래의 날짜 및 시간이어야 함
- `@Past` : 현재의 시간보다 과거의 날짜 및 시간이어야 함
- `@PastOrPresent` : 현재의 시간이거나 과거의 날짜 및 시간이어야 함

    cf. 가능 속성: message(담을 오류 메시지), groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

<br/>

8. 이메일 검증
- `@Email` : 올바른 이메일 주소여야 함(@ 포함됨)

    cf. 가능 속성: regexp(정규 표현식), flags, message, groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

<br/>

9. 정규식 검증
- `@Pattern` : 정규식과 대응되는 문자열이어야 함

    cf. 가능 속성: regexp(정규 표현식), flags, message, groups, payload <br/>
    cf. null 값은 유효하다고 판단됨

<br/>
<hr/>

[ 참고 사이트 ]
- https://www.baeldung.com/spring-boot-bean-validation
- https://www.baeldung.com/java-bean-validation-not-null-empty-blank
- https://jyami.tistory.com/55

<br/>
지금까지 자바의 Valid 어노테이션을 설명하는 포스트였습니다. 감사합니다:)
