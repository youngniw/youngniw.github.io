---
layout: post
title: "[Spring] Mapstruct 라이브러리"
categories: Spring
tags: [Spring, SpringBoot, 스프링, 스프링부트, 라이브러리]
---

## 0. 들어가기에 앞서..

이번 포스트에서는 엔티티와 DTO 같이 서로 다른 개체 모델 간에 매핑을 자동화하여 단순화하는 Mapstruct 라이브러리에 대해 설명하고자 합니다.

<br/>

Mapstruct을 활용한 관련 데모 프로젝트 코드는 아래의 링크에서 확인할 수 있습니다.

[Mapstruct Demo Project](https://github.com/youngniw/demo-mapstruct)

<br/>
<hr/>

## 1. Mapstruct 란?

MapStruct: 자바 빈 타입 간의 매핑 구현을 매우 단순화하는 코드 생성기

### 1.1 특징

- 생성된 매핑 코드는 일반적인 메서드 호출을 사용하므로 빠르고 유형 안전하며 이해하기 쉬움
- 매핑 코드를 최대한 자동화하여 단순화함
- Java 컴파일러에 연결된 주석 처리기
  - 다른 매핑 프레임워크와 달리 컴파일 시 Bean 매핑을 생성하여 철저한 오류 검사 가능

즉, MapStruct 기술을 사용하여 DTO와 Entity 간의 매핑을 쉽고 편리하게 할 수 있으며 일일이 객체를 생성하는 중복 코드를 줄일 수 있다.

<br/>
<hr/>

### 1.2 사용 방법

1. Mapstruct 라이브러리 설치에 
2. 서로 다른 객체 모델을 매핑하기 위해서는 매퍼 인터페이스 정의

<br/>

<b>[ 라이브러리 ]</b>

Mapstruct 라이브러리를 사용하기 위해 다음과 같이 build.gradle에 아래의 코드를 추가해야 한다.

```gradle
dependencies {
  ...
  implementation 'org.mapstruct:mapstruct:1.5.3.Final'
  annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'
}
```

<br/>

<b>[ 매퍼 인터페이스 정의 ]</b>

매퍼를 생성하려면 매핑을 위한 매퍼 인터페이스를 정의해야 한다.

아래의 코드는 매퍼 인터페이스에 대한 기본적인 예제 코드이다.

```java
@Mapper
public interface PostMapper {
  PostMapper INSTANCE = Mappers.getMapper( PostMapper.class );   // 외부 클래스에서 참조할 매퍼 인스턴스

  @Mapping(source = "title", target = "head")
  @Mapping(source = "content", target = "body")
  @Mapping(target = "createdDate", expression = "java(post.getCreatedDate().toLocalDate())")
  PostDto postToPostDto(Post post);
}
```

- `@Mapper`: 인터페이스를 매핑 인터페이스로 표시하며, 컴파일 중에 MapStruct 프로세서가 작동하도록 함
- 매핑 메소드: source 객체를 매개 변수로 예상하고 Mapping 어노테이션을 참고해 target 객체를 반환
- `@Mapping`: 소스 및 타겟 객체에서 이름이 다른 속성이 존재하는 경우에는 이름을 구성할 수 있으며, 이외의 다양한 작업을 통해 속성 값 설정 가능
- Mappers 클래스의 getMapper 메서드: 인터페이스 구현체의 인스턴스 검색

하나의 인터페이스에 여러 매핑 방법 정의 가능하며, 모든 구현은 MapStruct에 의해 생성된다. 
따라서 매퍼 인터페이스를 기반으로 클라이언트는 매우 쉽고 형식이 안전한 방식으로 객체 매핑을 수행 가능하다.

<br/>

<b>[ 예시 코드 ]</b>

다음은 위의 매퍼 인터페이스를 제외한 나머지 예제 코드이다. 

- Entity

```java
public class Post {
  private Long postId;
  private String title;
  private String content;
  private LocalDateTime createdDate;

  //constructor, getters, setters etc.
}
```
<br/>

- DTO

```java
public class Post {
  private Long postId;
  private String head;
  private String body;
  private LocalDate createdDate;

  //constructor, getters, setters etc.
}
```

다음 코드와 같이 매퍼 인터페이스를 기반으로 객체 매핑을 수행할 수 있다.

```java
@Test
public void mappingPostToDto() {
  // given
  LocalDateTime today = LocalDateTime.now();
  Post post = new Post(1L, "게시글 제목", "게시글 내용", today);

  // when
  PostDto postDto = PostMapper.INSTANCE.postToPostDto(post);

  // then
  assertThat(postDto).isNotNull();
  assertThat(postDto.getPostIdx()).isEqualTo(1L);
  assertThat(postDto.getHead()).isEqualTo("게시글 제목");
  assertThat(postDto.getBody()).isEqualTo("게시글 내용");
  assertThat(postDto.getCreatedDate()).isEqualTo(today.toLocalDate());
}
```
이와 같이 매퍼 인터페이스의 구현체 인스턴스의 메서드를 호출해 매핑을 진행할 수 있다.

<br/>
<hr/>

## 2. Mapstruct 관련 어노테이션

### 2.1 @Mapper

- Mapper 사용 시, 스프링과 함께 사용할 경우 `@Mapper`의 옵션으로 `componentModel = "spring"`을 설정해 주어야 오류가 발생하지 않는다.
- `ReportingPolicy.IGNORE`를 설정해서 일치하지 않는 필드 무시
  - cf. @Mapping의 속성 ignore 값을 true 함으로써 무시할 필드를 명시할 수 있음

<br/>

**[ 옵션 ]**

다음은 자주 사용하는 옵션에 대한 설명이다.

|옵션명|설명|
|--|--|
|componentModel|생성된 매퍼가 부착되어야 하는 구성요소 모델을 지정|
|uses|현 매퍼에서 사용하는 다른 매퍼 유형 명시|
|nullValueCheckStrategy|빈 매핑의 원본 속성 값에 대한 null 검사를 포함할 시기 결정|
|nullValueMappingStrategy|매퍼의 메서드에 null이 source의 값으로 전달될 때 적용할 전략 지정|

<br/>

<b>[ 예시 코드 ]</b>
  
  예외로, 다음의 코드와 같이 하나의 매퍼의 형식(틀)을 하나로 통일시켜 상속받아 객체 매퍼를 정의할 수 있다.

- EntityMapper.java

  ```java
  public interface EntityMapper<Dto, Entity> {
    Entity toEntity(Dto dto);

    Dto toDto(Entity entity);

    List<Entity> toEntity(List<Dto> dtoList);

    List<Dto> toDto(List<Entity> entityList);

    @Named("partialUpdate")
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void partialUpdate(@MappingTarget Entity entity, Dto dto);
  }
  ```

- UserMapper.java (이후의 실제 객체 매퍼 인터페이스 정의)

  ```java
  @Mapper(componentModel = "spring")
  public interface UserMapper extends EntityMapper<UserDto, User> {
    @Mapping(target = "createdDate", ignore = true)
    @Mapping(target = "updatedDate", ignore = true)
    @Mapping(target = "loginId", ignore = true)
    @Mapping(target = "loginPw", ignore = true)
    User toEntity(UserDto userDto);

    @Mapping(target = "isCreatedToday", expression = "java(user.getCreatedDate().toLocalDate().isEqual(java.time.LocalDate.now()))")
    UserDto toDto(User user);

    @Named("partialUpdate")
    @Mapping(target = "uid", ignore = true)
    @Mapping(target = "createdDate", ignore = true)
    @Mapping(target = "updatedDate", ignore = true)
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void partialUpdate(@MappingTarget User entity, UserDto dto);

    default User fromIdx(Long idx) {
      if (idx == null) {
        return null;
      }
      return User.builder()
        .uid(idx)
        .build();
    }
  }
  ```
  - User 엔티티 클래스와 UserDto 클래스 간의 변환을 위한 설정
  - 위의 코드에서 사용하는 어노테이션 @Mapping, @Named에 대한 설명은 아래에서 확인할 수 있다.

<br/>

- [ 실제 서비스에서의 사용 예시 ]

  ```java
  UserDto userDto = userMapper.toDto(user);
  List<UserDto> userDto = userMapper.toDto(userList);
  ```

  - UserMapper 인터페이스를 통해 자동으로 mapstruct에 의해 만들어진 구현체를 활용하여 실제 구현된 메서드를 호출
  - 2줄의 코드 모두 User 타입의 객체를 찾아 해당 객체를 UserDto로 변환한 값 반환

<br/>
<hr/>

### 2.2 @Mapping

- 변환하려는 객체와 필드 명이 일치하지 않는 경우에는 @Mapping 어노테이션을 이용해 수동으로 매핑

  [ 예시 코드 ]
  
  ```java
  @Mapping(source = "userId", target = "writerId")
  ```

- 만약 하나의 객체인 필드를 나눠서 저장하려고 할 때에는 `변수명.필드명`을 통해 변환 가능

  [ 예시 코드 ]

  ```java
  @Mapping(source = "user.name", target = "writerName")
  ```

<br/>

**[ 옵션 ]**

|옵션명|설명|
|--|--|
|target|**필수**적인 옵션으로, 변환 대상의 속성<br/>(target="."일 경우: 대상 객체로 모든 속성을 매핑함)|
|source|매핑에 사용할 소스|
|dateFormat|속성이 문자열에서 날짜로 매핑되거나 그 반대인 경우 SimpleDateFormat으로 처리할 수 있는 형식 문자열|
|numberFormat|속성이 문자열에서 날짜로 매핑되거나 그 반대인 경우 DecimalFormat으로 처리할 수 있는 형식 문자열|
|constant|지정한 target 속성의 값으로 설정할 상수 문자열|
|expression|지정한 target 속성의 값으로 설정할 표현식(문자열)|
|defaultExpression|지정된 source가 null인 경우에만 지정된 target 속성의 값으로 설정할 기본값 표현식(문자열)|
|ignore|target 속성의 값을 생성된 매핑 메서드에서 무시할지 여부|
|qualifiedBy|적합한 매퍼의 선택 프로세스를 돕기 위해 한정자 지정 (커스텀한 어노테이션 명시)|
|qualifiedByName|문자열 기반 한정자 형식으로, @Named 어노테이션을 선언한 메소드의 @Named 어노테이션 값을 명시|
|conditionQualifiedBy|qualifiedBy 옵션과 비슷하지만, 본 옵션은 Condition 메서드에만 적용 가능|
|conditionQualifiedByName|qualifiedByName 옵션과 비슷하지만, 본 옵션은 Condition 메서드에만 적용 가능|
|conditionExpression|지정한 속성의 존재 여부를 확인하는 기준이 되는 조건 표현식(문자열)|
|resultType|여러 매핑 메서드가 적합한 경우 사용할 매핑 메서드의 결과 유형 지정|
|defaultValue|source 속성 값이 null인 경우, 제공된 기본 String 값 설정|
|nullValueCheckStrategy|Bean 매핑의 source 속성 값에 대한 null 검사를 포함할 시기 결정|
|nullValuePropertyMappingStrategy|source 속성 값이 null이거나 존재하지 않을 때 적용할 전략<br/>(기본값: `NullValuePropertyMappingStrategy.SET_TO_NULL`)|

<br/>

<b>[ 예시 코드 ]</b>
  
  ```java
  @Mapping(target = "updatedDate", ignore = true)
  @Mapping(source = "writer", target = "writer")
  Post toEntity(PostDto postDto);
  ```

  변환 대상의 updatedDate 속성 값은 매핑하지 않고 제외하며, 소스의 writer 속성 값은 변환 대상인 target의 writer 속성 값으로 매핑 되도록 한다.

<br/>
<hr/>

### 2.3 @BeanMapping

- 두 빈 유형 간의 매핑 구성
- resultType 메서드, qualifiedBy 메서드 또는 nullValueMappingStrategy 메서드를 지정해야 함

<br/>

**[ 옵션 ]**

|옵션명|설명|
|--|--|
|nullValueMappingStrategy|빈 매핑에 source 빈 인수 값으로 null이 전달될 때 적용할 전략|
|nullValuePropertyMappingStrategy|원본 빈 속성이 null이거나 없을 때 적용할 전략|
|resultType|여러 팩토리 메서드가 적합한 경우에 사용할 팩터리 메서드의 결과 유형 지정|
|qualifiedBy|적합한 팩토리 메서드의 선택 프로세스 또는 적용 가능한 @BeforeMapping 및 @AfterMapping 메서드를 필터링하는 데 도움이 되는 한정자를 지정|

<br/>
<hr/>

### 2.4 @MappingTarget

- 매핑으로 기존의 객체를 업데이트하는 경우에 사용
- target 유형의 새 인스턴스를 생성하지 않는 매핑을 하는즉, 유사한 유형의 기존 인스턴스를 업데이트
- 이와 같은 경우의 매핑은 target 객체에 대한 매개변수를 추가하고 target  객체 매개변수를 @MappingTarget로 표시함
- 즉, 값의 변경을 원하는 객체에 @MappingTarget 어노테이션을 표기함

<br/>

<b>[ 예시 코드 ]</b>
  
  ```java
  @Named("partialUpdate")
  @Mapping(target = "uid", ignore = true)
  @Mapping(target = "createdDate", ignore = true)
  @Mapping(target = "updatedDate", ignore = true)
  @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
  void partialUpdate(@MappingTarget User entity, UserDto dto);
  ```

  위의 메소드 호출 시, userDto의 값으로 entity의 값이 업데이트가 된다.

<br/>
<hr/>

지금까지 서로 다른 객체 간의 변환을 쉽게 할 수 있도록 도와주는 mapstruct 라이브러리를 설명하였습니다.

감사합니다:)
