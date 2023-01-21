---
layout: post
title: "[JPA] 4. 엔티티 매핑"
categories: JPA
tags: [SprintBoot, JPA, 스프링부트]
---

# &#91;자바 ORM 표준 JPA 프로그래밍&#93; Chapter 4. 엔티티 매핑

## 0. 들어가기에 앞서..
현재 이 포스트는 김영한님의 "자바 ORM 표준 JPA 프로그래밍 책"에 대한 내용을 바탕으로 작성하였습니다. 

<br/>
<hr/>
<br/>

JPA를 사용하는 데 가장 중요한 일은 엔티티와 테이블을 정확히 매핑하는 것이다.
JPA는 다양한 매핑 애노테이션을 지원하는데 크게 4가지로 분류할 수 있다.
- 객체와 테이블 매핑: @Entity, @Table
- 기본 키 매핑: @Id
- 필드와 컬럼 매핑: @Column
- 연관관계 매핑: @ManyToOne, @JoinColumn

이번 장에서는 객체와 테이블 매핑, 기본 키 매핑, 필드와 컬럼 매핑에 대해 알아보고자 한다. 

참고로 매핑 정보는 XML이나 애노테이션 중에 선택해서 기술하면 되는데 이 포스트에서는 애노테이션만 사용한다.

<br/>
<hr/>
<br/>

## 4.1 @Entity
JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 애노테이션을 필수로 붙여야 한다.
@Entity가 붙은 클래스는 JPA가 관리하는 것으로, 엔티티라고 부른다. 이 엔티티의 속성은 다음과 같다.

|속성|기능|기본값|
|---|---|---|
|name|JPA에서 사용할 <b>엔티티 이름</b>을 지정한다. 보통 기본값인 클래스 이름을 사용하는데, 만약 다른 패키지에 이름이 같은 엔티티 클래스가 있다면 이름을 지정해서 충돌하지 않도록 해야 한다.|설정하지 않으면 클래스 이름을 그대로 사용한다.|

@Entity 적용 시 주의사항은 다음과 같다.
- 기본 생성자는 필수이다.(파라미터가 없는 public 또는 protected 생성자)
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 저장할 필드에 final을 사용하면 안된다.

첫 번째 주의사항을 보면, 자바는 생성자가 하나도 없으면 기본 생성자를 자동으로 만들기 때문에 문제가 될 것은 없다. 
하지만 생성자를 하나 이상 만들면 자바는 기본 생성자를 자동으로 만들지 않기 때문에, 이때는 기본 생성자를 직접 만들어야 한다.

<br/>
<hr/>
<br/>

## 4.2 @Table
@Table은 엔티티와 매핑할 테이블을 지정한다. 만약 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다. 
@Table에 대한 속성은 다음과 같다.

|속성|기능|기본값|
|---|---|---|
|name|매핑할 테이블 이름|앤티티 이름을 사용한다.|
|catalog|catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다.||
|schema|schema 기능이 있는 데이터베이스에서 schema 매핑한다.||
|uniqueConstraints|DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다.||

<br/>
<hr/>
<br/>

## 4.3 다양한 매핑 사용
다음의 요구사항을 만족하도록 기존에 회원 엔티티를 수정하고자 한다.
1. 회원은 일반 회원과 관리자로 구분해야 한다.
2. 회원 가입일과 수정일이 있어야 한다.
3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

```java
//예제 4.1 회원 엔티티
@Entity
@Table(name="MEMBER")
public class Member {
    @Id
    @Column(name="ID")
    private String id;
    
    @Column(name="NAME")
    private String username;
    
    private Integer age;
    
    //--추가--
    @Enumerated(EnumType.STRING)
    private RoleType roleType;
    
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;
    
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
    
    @Lob
    private String description;
    
    //Getter, Setter
    ···
}

public enum RoleType {
    ADMIN, USER
}
```

요구사항 첫 번째를 만족하기 위해 자바의 enum을 사용해서 회원의 타입을 구분했다. 이처럼 자바의 enum을 사용하기 위해서는 @Enumerated 애노테이션으로 매핑해야 한다.

요구사항 두 번째를 만족하기 위해서 날짜 필드를 추가했는데, 자바의 날짜 타입은 @Temporal을 사용해서 매핑한다.

요구사항 세 번째를 만족하기 위해서 설명 필드를 추가했는데, 이 필드는 길이 제한이 없기 때문에 데이터베이스의 VARCHAR 타입 대신에 CLOB 타입으로 저장해야 한다. 이것은 @Lob을 사용하면 CLOB, BLOB 타입을 매핑할 수 있다.

이전에는 테이블을 먼저 생성하고 그 후에 엔티티를 만들었지만, 이번에는 데이터베이스 스키마 자동 생성을 사용해서 엔티티만 만들고 테이블은 자동 생성되도록 하자.

<br/>
<hr/>
<br/>

## 4.4 데이터베이스 스키마 자동 생성
JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 지원한다. 
보통 클래스의 매핑 정보를 보면 어떤 테이블에 어떤 컬럼을 사용할 수 있는지 알 수 있는데, JPA는 이 매핑정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 생성한다.

스키마 자동 생성 기능을 사용해보고자 한다. 이 포스트에서는 메이븐을 사용하기 때문에 persistence.xml에 다음 속성을 추가한다.

`<property name="hibernate.hbm2ddl.auto" value="create"/>`

위의 속성을 추가하면 <b>애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성</b>한다. 참고로 hibernate.show_sql 속성을 true로 설정하면 콘솔에 실행되는 테이블 생성 DDL<sub>Data Definition Language</sub>을 출력할 수 있다.

`<property name="hibernate.show_sql" value="true"/>`

애플리케이션을 실행하면 다음과 같이 콘솔에 DDL이 출력되면서 실제 테이블이 생성된다.

```
Hibernate:
    drop table MEMBER if exists
Hibernate:
    create table MEMBER (
        ID varchar(255) not null,
        NAME varchar(255),
        age integer,
        roleType varchar(255),
        createDate timestamp,
        lastModifiedDate timestamp,
        description clob,
        primary key (ID)
    )
```
참고로 자동 생성되는 DDL은 지정한 데이터베이스 방언에 따라 달라진다. 예를 들어 오라클은 varchar이 varchar2로, integer 대신에 number 타입이 생성된다.

스키마 자동 생성 기능을 사용하면 애플리케이션 실행 시점에 데이터베이스 테이블이 자동으로 생성되므로 개발자가 테이블을 직접 생성하는 수고를 덜 수 있다.
하지만 이렇게 만들어진 DDL은 운영 환경에서 사용할 만큼 완벽하지는 않으므로 개발 환경에서 사용하거나 매핑을 어떻게 해야 하는지 참고하는 정도로 사용하는 것이 좋다.

다음은 hibernate.hbm2ddl.auto 속성과 관련된 내용이다.

|옵션|설명|
|---|---|
|create|기존 테이블을 삭제하고 새로 생성한다. DROP+CREATE|
|create-drop|create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다. DROP+CREATE+DROP|
|update|데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다.|
|validate|데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. 이 설정은 DDL을 수정하지 않는다.|
|none|자동 생성 기능을 사용하지 않으려면 hibernate.hbm2ddl.auto 속성 자체를 삭제하거나 유효하지 않은 옵션 값(none)을 주면 된다.|

참고로 운영 서버에서 create, create-drop, update처럼 DDL을 수정하는 옵션은 절대 사용하면 안된다. 오직 개발 서버나 개발 단계에서만 사용해야 한다.
그 이유는 이 옵션들은 운영 중인 데이터베이스의 테이블이나 컬럼을 삭제할 수 있기 때문이다!

<br/>
<hr/>
<br/>

## 4.5 DDL 생성 기능
만약 회원 이름은 필수로 입력되어야 하고, 10자를 초과하면 안된다는 제약조건이 추가된다면 다음과 같이 코드를 수정해야 한다. 

```java
//예제 4.4 추가 코드
@Entity
@Table(name="MEMBER")
public class Member {
    @Id
    @Column(name="ID")
    private String id;
    
    @Column(name="NAME", nullable = false, length = 10)     //추가
    private String username;
    
    ...
}
```
@Column 매핑정보의 nullable 속성 값을 false로 지정하면 자동 생성되는 DDL에 not null 제약조건을 추가할 수 있다. 
또한 length 속성 값을 사용하면 자동 생성되는 DDL에 문자의 크기를 지정할 수 있다.

스키마 자동 생성하기를 통해 만들어지는 DDL을 한번 살펴보자.
```
//예제 4.5 생성된 DDL
create table MEMBER (
    ID varchar(255) not null,
    NAME varchar(10) not null,
    ...
    primary key (ID)
)
```

이번에는 유니크 제약조건을 만들어 주는 @Table의 uniqueConstraints 속성을 알아보자.
```java
//예제 4.6 유니크 제약조간
@Entity(name="Member")
@Table(name="MEMBER", uniqueConstraints = {@UniqueConstraint(
    name="NAME_AGE_UNIQUE",
    columnNames={"NAME", "AGE"} )})     //추가
public class Member {
    @Id
    @Column(name="ID")
    private String id;
    
    @Column(name="NAME")
    private String username;
    
    private Integer age;
    ...
}
```
```
//예제 4.7 생성된 DDL
ALTER TABLE MEMBER
    ADD CONSTRAINT NAME_AGE_UNIQUE UNIQUE (NAME, AGE)
```
예제 4.7의 생성된 DDL을 보면 유니크 제약조건이 추가되었다. 
앞에서 본 @Column의 length와 nullable 속성을 포함해서 <b>이런 기능들은 단지 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.</b>
따라서 스키마 자동 생성 기능을 사용하지 않고 직접 DDL을 만든다면 사용할 이유가 없다.
그래도 이 기능을 사용하면 애플리케이션 개발자가 엔티티만 보고도 손쉽게 다양한 제약조건을 파악할 수 있는 장점이 있다.

<br/>
<hr/>
<br/>

## 4.6 기본 키 매핑
지금까지 @Id 애노테이션만 사용해서 기본 키를 애플리케이션에서 직접 할당했다. 

만약 기본키를 애플리케이션에서 직접 할당하는 대신에 데이터베이스가 생성해주는 값을 사용하려면 어떻게 매핑해야 하는 지를 생각해보자.

하지만 데이터베이스마다 기본 키를 생성하는 방식이 서로 다르기 때문에 이 문제를 해결하기는 쉽지 않다. JPA는 이런 문제들을 어떻게 해결하는지 이번 장에서 설명할 것이다.

JPA가 제공하는 데이터베이스 기본 키 생성 전략은 다음과 같다.
- <b>직접 할당</b>: 기본 키를 <u>애플리케이션에서 직접 할당</u>한다.
- <b>자동 생성</b>: 대리 키 사용 방식
    - IDENTITY: 기본 키 생성을 데이터베이스에 위임한다.
    - SEQUENCE: 데이터베이스 시퀀스를 사용해서 기본 키를 할당한다.
    - TABLE: 키 생성 테이블을 사용한다.

자동 생성 전략이 다양한 이유는 데이터베이스 벤더마다 지원하는 방식이 다르기 때문이다. 
예를 들어 오라클 데이터베이스는 시퀀스를 제공하지만, MySQL은 시퀀스를 제공하지 않고 기본 키 값을 자동으로 채워주는 AUTO_INCREMENT 기능을 제공한다.
따라서 SEQUENCE나 IDENTITY 전략은 사용하는 데이터베이스에 의존한다. 
TABLE 전략은 키 생성용 테이블을 하나 만들어주고 마치 시퀀스처럼 사용하는 방법으로, 모든 데이터베이스에서 사용할 수 있다.

결론적으로 기본 키를 직접 할당하려면 @Id만 사용하면 된다. 반면에 자동 생성 전략을 사용하려면 @Id에 @GeneratedValue를 추가하고 원하는 키 생성 전략을 선택하면 된다.

이제부터 각각의 전략을 어떻게 사용하는지 살펴보도록 하자.
<br/><br/>

### 4.6.1 기본 키 직접 할당 전략
```java
@Id
@Column(name = "id")
private String id;
```
기본 키를 직접 할당하려면 위와 같이 @Id로 매핑하면 된다.

@Id 적용 가능한 자바 타입은 다음과 같다.
- 자바 기본형
- 자바 래퍼<sub>Wrapper</sub>형
- String
- java.util.Date
- java.sql.Date
- java.math.BigDecimal
- java.math.BigInteger

기본 키 직접 할당 전략은 em.persist()로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법이다.

```java
Member member = new Member();
member.setId("id1");    //기본 키 직접 할당
em.persist(member);
```

참고로, 기본 키 직접 할당 전략에서 식별자 값 없이 저장하면 예외가 발생한다.
<br/><br/>

### 4.6.2 IDENTITY 전략
IDENTITY는 기본 키 생성을 데이터베이스에 위임하는 전략이다.
주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.

예를 들어 MySQL의 AUTO_INCREMENT 기능은 데이터베이스가 기본 키를 자동으로 생성해준다.
이 기능을 사용한다면 데이터베이스에 값을 저장할 때 해당 컬럼을 비워두면 데이터베이스가 순서대로 값을 채워준다. 

IDENTITY 전략은 데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다.
이 IDENTITY 전략을 사용하기 위해서는 @GeneratedValue의 strategy 속성 값을 GenerationType.IDENTITY로 지정하면 된다. 
이에 대한 예제 코드는 다음과 같다.

```java
//예제 4.9 IDENTITY 매핑 코드
@Entity
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    ...
}
```

```java
//예제 4.10 IDENTITY 사용 코드
private static void logic(EntityManager em) {
    Board board = new Board();
    em.persist(board);
    System.out.println("board.id = " + board.getId());
}

//출력: board.id = 1
```
위의 예제 4.10 코드를 보면 em.persist()를 호출해서 엔티티를 저장한 직후에 할당된 식별자 값을 출력했다.
이 출력된 값 1은 저장 시점에 데이터베이스가 생성한 값을 JPA가 조회한 것이다.

참고로, 엔티티가 영속 상태가 되려면 식별자가 반드시 필요하다. 
그런데 IDENTITY 식별자 생성 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 em.persist()를 호출하는 즉시 INSERT SQL이 데이터베이스에 전달된다.
<u>따라서 이 전략은 트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다.</u>
<br/><br/>

### 4.6.3 SEQUENCE 전략
데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다. SEQUENCE 전략은 이 시퀀스를 사용해서 기본 키를 생성한다.
이 전략은 SEQUENCE를 지원하는 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용할 수 있다.

시퀀스는 다음과 같이 생성해야 한다.(DDL)

`CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;`

```java
//예제 4.12 시퀀스 매핑 코드
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "BOARD_SEQ",     //매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, 
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
    ...
}
```
우선 사용할 데이터베이스 시퀀스를 매핑해야 한다. 
예제 4.12에서는 @SequenceGenerator를 사용해서 시퀀스 생성기를 등록했다. 
그리고 sequenceName 속성을 통해 지정한 이름으로 JPA는 시퀀스 생성기를 실제 데이터베이스의 시퀀스와 매핑한다.

다음으로 키 생성 전략을 GenerationType.SEQUENCE로 설정하고, generator값으로 등록한 시퀀스 생성기를 선택함으로써 식별자 값은 해당 시퀀스 생성기가 할당하게 된다.

시퀀스 사용 코드는 IDENTITY 전략과 같지만 내부 동작 방식은 다르다. 
SEQUENCE 전략은 em.persist()를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다. 
그리고 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다. 
이후에 트랜잭션을 커밋해서 플러시가 일어나게 되면 엔티티를 데이터베이스에 저장한다.(트랜잭션을 지원하는 쓰기 지연 동작 o)<br/>
cf. IDENTITY 전략은 먼저 엔티티를 데이터베이스에 저장한 후에 식별자를 조회해서 엔티티의 식별자에 할당한다.

@SequenceGenerator의 속성은 다음과 같다.

|속성|기능|기본값|
|---|---|---|
|name|식별자 생성기 이름|필수|
|sequenceName|데이터베이스에 등록되어 있는 시퀀스 이름|hibernage_sequence|
|initialValue|DDL 생성 시에만 사용된다. 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다.|1|
|allocationSize|시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)|50|
|catalog, schema|데이터베이스 catalog, schema 이름||

매핑할 DDL은 다음과 같다.
`CREATE SEQUENCE [sequenceName] START WITH [initialValue] INCREMENT BY [allocationSize];`
JPA 표준 명세에서는 sequenceName의 기본값을 JPA 구현체가 정의하도록 했다. 위에서 설명한 기본값은 하이버네이트 기준이다.

참고로 SEQUENCE 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다. 따라서 데이터베이스와 2번 통신한다.
1. 식별자를 구하려고 데이터베이스 시퀀스를 조회한다.
2. 조회한 시퀀스를 기본 키 값으로 사용해 데이터베이스에 저장한다.
<br/><br/>

### 4.6.4 TABLE 전략
TABLE 전략은 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다.
이 전략은 테이블을 사용하기 때문에 모든 데이터베이스에 적용할 수 있다.

TABLE 전략을 사용하려면 키 생성 용도로 사용할 테이블을 만들어야 한다.
```
//예제 4.14 TABLE 전략 키 생성 DDL
CREATE TABLE MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key (sequence_name)
);
```
위의 코드를 보면 sequence_name을 시퀀스 이름으로 사용하고, next_val 컬럼을 시퀀스 값으로 사용한다. 
참고로 컬럼의 이름은 변경할 수 있는데 위의 컬럼명이 기본 값이다.

```java
//예제 4.15 TABLE 매핑 코드
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BAORD_SEQ", allocationSize = 1)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, 
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
    ...
}
```
위의 코드를 보면 @TableGenerator를 사용해서 테이블 키 생성기를 등록한다. 
그리고 table 속성을 통해 위에서 생성한 MY_SEQUENCES 테이블을 키 생성용 테이블로 매핑했다.
그리고 TABLE 전략을 사용하기 위해 GenerationType.TABLE을 선택했다. 그리고 @GeneratedValue의 generator로 테이블 키 생성기를 지정했다.
따라서 이후에는 식별자 값은 지정한 테이블 키 생성기가 할당하게 된다.

TABLE 전략은 시퀀스 대신에 테이블을 사용한다는 것만 제외하면 SEQUENCE 전략과 내부 동작방식이 같다.

참고로 키 생성 용도로 사용하는 테이블에 값이 없으면 JPA가 값을 INSERT하면서 초기화하므로 값을 미리 넣어둘 필요가 없다.

@SequenceGenerator의 속성은 다음과 같다.

|속성|기능|기본값|
|---|---|---|
|name|식별자 생성기 이름|필수|
|table|키생성 테이블명|hibernage_sequences|
|pkColumnName|시퀀스 컬럼명|sequence_name|
|valueColumnName|시퀀스 값 컬럼명|next_val|
|pkColumnValue|키로 사용할 값 이름|엔티티 이름|
|initialValue|초기 값, 마지막으로 생성된 값이 기준이다.|0|
|allocationSize|시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)|50|
|catalog, schema|데이터베이스 catalog, schema 이름||
|uniqueConstraints(DDL)|유니크 제약 조건을 지정할 수 있다.||

JPA 표준 명세에서는 table, pkColumnName, valueColumnName의 기본값을 JPA 구현체가 정의하도록 했다. 위에서 설명한 기본값은 하이버네이트 기준이다.

참고로 TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고 다음 값으로 증가시키기 위해 UPDATE 쿼리를 사용한다. 
이 전략은 SEQUENCE 전략과 비교해서 데이터베이스와 한 번 더 통신하는 단점이 있다.
<br/><br/>

### 4.6.5 AUTO 전략
데이터베이스는 종류도 많고 기본 키를 만드는 방법도 다양한다.
GenerationType.AUTO는 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.
예를 들어 오라클을 선택하면 SEQUENCE를, MySQL을 선택하면 IDENTITY를 사용한다.

AUTO 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 것이다. 

참고로, AUTO를 사용할 때 SEQUENCE나 TABLE 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어 두어야 한다.
그런데 만약 스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본 값을 사용해서 적절한 시퀀스나 키 생성용 테이블을 만들어 줄 것이다.
<br/><br/>

### 4.6.6 기본 키 매핑 정리
영속성 컨텍스트는 엔티티를 식별자 갑승로 구분하므로 엔티티를 영속 상태로 만들려면 식별자 값이 반드시 있어야 한다.
em.persist()를 호출한 직후에 발생하는 일을 정리하면 다음과 같다.
- 직접 할당: em.persist()를 호출하기 전에 애플리케이션에서 직접 식별자 값을 할당해야 한다. 만약 식별자 값이 없으면 예외가 발생한다.
- IDENTITY: 데이터베이스에 엔티티를 저장해서 식별자 값을 획득한 후에 영속성 컨텍스트에 저장한다.(테이블에 데이터를 저장해야 식별자 값을 획득할 수 있음)
- SEQUENCE: 데이터베이스 시퀀스에서 식별자 값을 획득한 후에 영속성 컨텍스트에 저장한다.
- TABLE: 데이터베이스 시퀀스 생성용 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.

<br/>
참고로 테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.
- 자연 키(natural key): 비즈니스에 의미가 있는 키
- 대리 키(surrogate key): 비즈니스와 관련 없는 임의로 만들어진 키. 대체 키라고도 불린다.

보통은 자연 키보다 대리 키를 권장한다. 그 이유는 현실과 비즈니스 규칙은 생각보다 쉽게 변하기 때문이다.

<br/>
<hr/>
<br/>

## 4.7 필드와 컬럼 매핑: 레퍼런스
JPA가 제공하는 필드와 컬럼 매핑용 애노테이션들은 다음과 같다.

|분류|매핑 애노테이션|설명|
|---|---|---|
|필드와 컬럼 매핑|@Column|컬럼을 매핑한다.|
||@Enumerated|자바의 enum 타입을 매핑한다.|
||@Temporal|날짜 타입을 매핑한다.|
||@Lob|BLOB, CLOB 타입을 매핑한다.|
||@Transient|특정 필드를 데이터베이스에 매핑하지 않는다.|
|기타|@Access|JPA가 엔티티에 접근하는 방식을 지정한다.|
<br/>

### 4.7.1 @Column
가장 많이 사용되고 기능이 많은 @Column은 객체 필드를 테이블 컬럼에 매핑한다. 
속성 중에는 name과 nullable이 주로 사용되며 나머지는 잘 사용되지 않는다.
또한 속성 중에 insertable과 updatable 속성은 데이터베이스에 저장되어 있는 정보를 읽기만 하고 실수로 변경하는 것을 방지하고 싶을 때 사용한다.

|속성|기능|기본값|
|---|---|---|
|name|필드와 매핑할 테이블의 컬럼 이름|객체의 필드 이름|
|insertable|엔티티 저장 시에 이 필드도 저장한다. false로 설정하면 이 필드는 데이터베이스에 저장하지 않는다.(읽기 전용일 때 사용)|true|
|updatable|엔티티 수정 시에 이 필드도 저장한다. false로 설정하면 데이터베이스에 수정하지 않는다.(읽기 전용일 때 사용)|true|
|table|하나의 엔티티를 두 개 이상의 테이블에 매핑할 때 사용한다. 지정한 필드를 다른 테이블에 매핑할 수 있다.|현재 클래스가 매핑된 테이블|
|nullable(DDL)|null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.|true|
|unique(DDL)|@Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.||
|columnDefinition(DDL)|데이터베이스 컬럼 정보를 직접 줄 수 있다.|필드의 자바 타입과 방언 정보를 사용해서 적절한 컬럼 타입을 생성한다.|
|length(DDL)|문자 길이 제약 조건이며, String 타입에만 사용한다.|255|
|precision, scale(DDL)|BigDecimal 혹은 BigInteger타입에서 사용한다. precision은 소수점을 포함한 전체 자릿수를, scale은 소수의 자릿수다. 참고로 double과 float 타입에는 적용되지 않는다.|precision=19, scale=2|

@Column 속성을 생략하면 대부분 @Column 속성의 기본값이 적용되는데, 자바 기본 타입일 때는 nullable 속성에 예외가 있다.
int 같은 자바 기본 타입에는 null 값을 입력할 수 없다. 반면에 Integer처럼 객체 타입일 때만 null 값이 허용된다.
따라서 자바 기본 타입인 int를 DDL로 생성할 때는 not null 제약 조건을 추가하는 것이 안전하다.
<br/><br/>

### 4.7.2 @Enumerated
자바의 enum 타입을 매핑할 때 사용한다. 이 애노테이션의 속성은 다음과 같다.

|속성|기능|기본값|
|---|---|---|
|value|- EnumType.ORDINAL: enum 순서를 데이터베이스에 저장</br>- EnumType.STRING: enum 이름을 데이터베이스에 저장|EnumType.ORDINAL|

- EnumType.ORDINAL은 정의된 순서대로 값이 데이터베이스에 저장된다.
    - 장점: 데이터베이스에 저장되는 데이터 크기가 작다.
    - 단점: 이미 저장된 enum의 순서를 변경할 수 없다.
- EnumType.STRING은 enum 이름 그대로 문자로 데이터베이스에 저장된다.
    - 장점: 저장된 enum의 순서가 바뀌거나 enum이 추가되어도 안전하다.
    - 단점: 데이터베이스에 저장되는 데이터 크기가 ORDINAL에 비해서 크다.

기본값인 ORDINAL은 주의해서 사용해야 한다. 만약 중간에 enum이 추가된다면 데이터베이스의 값과 객체 값이 다를 수 있으므로 EnumType.STRING을 사용하는 것을 권장한다.
<br/><br/>

### 4.7.3 @Temporal
날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용한다. 이 애노테이션의 속성은 다음과 같다.

|속성|기능|기본값|
|---|---|---|
|value|- TemporalType.DATE: 날짜, 데이터베이스 date 타입과 매핑 (ex. 2021-10-28)</br>- TemporalType.TIME: 시간, 데이터베이스 time 타입과 매핑 (ex. 11:11:11)</br>- TemporalType.TIMESTAMP: 날짜와 시간, 데이터베이스 timestamp 타입과 매핑 (ex. 2021-10-28 11:11:11)|TemporalType은 필수로 지정해야 한다.|

자바의 Date 타입에는 년월일 시분초가 있지만 데이터베이스는 date(날짜), time(시간), timestamp(날짜와 시간)로 세 가지 타입이 별도로 존재한다.

이때 @Temporal을 생략하면 자바의 Date와 가장 유사한 timestamp로 정의된다. 
timestamp 대신에 datetime을 예약어로 쓰는 데이터베이스도 있는데 데이터베이스 방언으로 인해 애플리케이션 코드는 변경하지 않아도 된다.
- datetime: MySQL
- timestamp: H2, 오라클, PostgreSQL
<br/><br/>

### 4.7.4 @Lob
데이터베이스 BLOB, CLOB 타입과 매핑한다. 

@Lob에는 지정할 수 있는 속성이 없다. 
대신에 매핑하는 필드 타입이 문자면 CLOB로 매핑하고, 나머지는 BLOB로 매핑한다.

- CLOB: String, char[], java.sql.CLOB
- BLOB: byte[], java.sql.BLOB
<br/><br/>

### 4.7.5 @Transient
이 필드는 매핑하지 않는다. 따라서 데이터베이스에 저장하지 않고 조회하지도 않는다. 
보통은 객체에 임시로 어떤 값을 보관하고 싶을 때 사용한다.
<br/><br/>

### 4.7.6 @Access
JPA가 엔티티 데이터에 접근하는 방식을 지정한다.

- <b>필드 접근</b>: AccessType.FIELD로 지정하며, 필드에 직접 접근한다. 이 방식은 필드 접근 권한이 private이어도 접근할 수 있다.
- <b>프로퍼티 접근</b>: AccessType.PROPERTY로 지정하며, 접근자<sub>Getter</sub>를 사용한다.

만약 @Access를 설정하지 않으면 @Id의 위치를 기준으로 접근 방식이 설정된다.

```java
//예제 4.20 필드, 프로퍼티 접근 함께 사용
@Entity
public class Member {
    @Id
    private String id;
    
    @Transient
    private String firstName;
    
    @Transient
    private String lastName;
    
    @Access(AccessType.PROPERTY)
    public String getFullName() {
        return firstName + lastName;
    }
    ...
}
```
위의 코드는 @Id가 필드에 있으므로 기본은 필드 접근 방식을 사용하고, getFullName()만 프로퍼티 접근 방식을 사용한다. 
따라서 회원 엔티티를 저장하면 회원 테이블의 FULLNAME 컬럼에 firstName + lastName의 결과가 저장된다.

<br/>
<hr/>
<br/><br/>
다음 포스트는 '자바 ORM 표준 JPA 프로그래밍' 책의 Chapter 5. 연관관계 매핑 기초에 대한 글입니다. 감사합니다:)
