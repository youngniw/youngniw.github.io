---
layout: post
title: "[JPA] 2. JPA 시작"
categories: JPA
tags: [SprintBoot, JPA, 스프링부트]
---

# &#91;자바 ORM 표준 JPA 프로그래밍&#93; Chapter 2. JPA 시작

## 0. 들어가기에 앞서..
현재 이 포스트는 김영한님의 "자바 ORM 표준 JPA 프로그래밍 책"에 대한 내용을 바탕으로 작성하였습니다. 

<br/>
<hr/>
<br/>

이번 장에서는 JPA를 사용해서 하나의 테이블을 등록/수정/삭제/조회하는 간단한 JPA 애플리케이션을 만들어보고자 한다. 
JPA는 필요한 라이브러리도 많고 데이터베이스도 필요하다. 

H2 데이터베이스는 설치가 필요 없고 용량도 1.7M으로 가볍고 웹용 쿼리 툴도 제공하기 때문에, 이 책에서는 H2 데이터베이스를 사용하고자 한다.
데이터베이스를 연동한 후에, JPA에 필요한 라이브러리를 설정하고 실제 JPA로 애플리케이션을 만들어보자.

<br/>
<hr/>
<br/>

## 2.2 H2 데이터베이스 설치
MySQL이나 오라클 데이터베이스를 사용해도 되지만, 이와 같은 데이터베이스는 설치하는 부담이 크다. 
따라서 설치가 필요 없고 용량도 1.7M로 가벼운 H2 데이터베이스를 사용하고자 한다. 
참고로 H2 데이터베이스는 자바가 설치되어 있어야 동작한다.

H2 데이터베이스는 JVM 메모리 안에서 실행되는 임베디드 모드와 실제 데이터베이스처럼 별도의 서버를 띄워서 동작하는 서버모드가 있다. 

- 드라이버 클래스: org.h2.Driver
- JDBC URL: jdbc:h2:tcp://localhost/~/test
- 사용자명: sa
- 비밀번호: 입력하지 않음

이렇게 하면 test라는 이름의 데이터베이스에 서버 모드로 접근하고, 작업을 수행할 수 있게 된다.

Ex. 예제 테이블 생성
```
//예제 2.1 회원 테이블
CREATE TABLE MEMBER (
    ID VARCHAR(255) NOT NULL,
    NAME VARCHAR(255),
    AGE INTEGER NOT NULL,
    PRIMARY KEY (ID)
);
```

<br/>
<hr/>
<br/>

## 2.3 라이브러리와 프로젝트 구조
필요한 모든 라이브러리를 직접 내려받아서 관리하기는 어렵기 때문에 라이브러리 관리 도구를 사용하는 것이 편리하다. 
요즘은 그래들을 많이 사용하는 편이긴 하지만 이 책에서는 메이븐이라는 도구를 사용한다. 
또한 JPA 구현체로 하이버네이트를 사용한다.

JPA 구현체로 하이버네이트를 사용하기 위한 핵심 라이브러리는 다음과 같다.
- hibernate-core: 하이버네이트 라이브러리
- hibernate-entitymanager: 하이버네이트가 JPA 구현체로 동작하도록 JPA 표준을 구현한 라이브러리
<br/><br/>

### 2.3.1 메이븐과 사용 라이브러리 관리
메이븐은 라이브러리를 관리해주는 도구로, pom.xml에 사용할 라이브러리를 작성하면 라이브러리를 자동으로 내려받아서 관리해준다. 

```
cf. 최근 자바 애플리케이션은 메이븐, 그래들과 같은 도구를 사용해서 라이브러리를 관리하고 빌드한다. 메이븐은 크게 라이브러리 관리 기능과 빌드 기능을 제공한다. 

라이브러리 관리 기능: 자바 애플리케이션을 개발하려면 jar 파일로 된 여러 라이브러리가 필요하다. 메이븐은 사용할 라이브러리 이름과 버전만 명시하면 라이브러리를 자동으로 내려받고 관리해준다.

빌드 기능: 메이븐은 애플리케이션을 빌드하는 표준화된 방법을 제공한다.
```

다음은 메이븐 설정 파일 예제이다.

```xml
//예제 2.3 메이븐 설정 파일 pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    <modelVersion>4.0.0</modelVersion>
	<groupId>jpabook</groupId>
	<artifactId>jpa-start</artifactId>
	<version>1.0-SNAPSHOT</version>

	<dependencies>
		<!-- JPA, 하이버네이트 -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		<!-- H2 데이터베이스 -->
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>${h2db.version}</version>
		</dependency>
	</dependencies>
    
</project>
```

&lt;dependencies&gt;에 사용할 라이브러리를 지정한다. 
이때 groupId + artifactId + version만 적어주면 라이브러리(jar 파일)를 메이븐 공식 저장소에서 내려받아 라이브러리에 추가해준다.

JPA에 하이버네이트 구현체를 사용하려면 많은 라이브러리가 필요하지만 핵심 라이브러리는 다음과 같다.
- JPA, 하이버네이트(hibernate-entitymanager)
- H2 데이터베이스

<br/>
<hr/>
<br/>

## 2.4 객체 매핑 시작
회원 테이블을 이전에 만들었다면 애플리케이션에서 사용할 회원 클래스를 만들고자 한다.

```java
//예제 2.5 회원 클래스
public class Member {
    private String id;
    private String username;
    private Integer age;
    
    //Getter, Setter
    ···
}
```
JPA를 사용하려면 가장 먼저 회원 클래스와 회원 테이블을 매핑해야 한다. 
그러기 위해서 회원 클래스에 JPA가 제공하는 매핑 애노테이션을 추가하자.

```java
//예제 2.6 매핑 정보가 포함된 회원 클래스

@Entity
@Table(name="MEMBER")
public class Member {
    @Id
    @Column(name = "ID")
    private String id;
                     
    @Column(name = "NAME")
    private String username;
                     
    private Integer age;
    
    //Getter, Setter
    ···
}
```
회원 클래스에 매핑 정보를 표시하는 애노테이션을 추가하였는데 그것은 바로 @Entity, @Table, @Column이다. 
JPA는 매핑 애노테이션을 분석해서 어떤 객체가 어떤 테이블과 관계가 있는지 알아낸다.

<br/>
<b>▽ @Entity</b>
: 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다. 이렇게 @Entity가 사용된 클래스를 엔티티 클래스라고 한다.

<br/>
<b>▽ @Table</b>
: 엔티티 클래스에 매핑할 테이블 정보를 알려준다. 이때 name 속성을 사용해서 엔티티를 작성한 속성값을 이름으로 하는 테이블에 매핑할 수 있다. 이 애노테이션을 생략하면 엔티티 이름을 테이블 이름으로 매핑한다.

<br/>
<b>▽ @Id</b>
: 엔티티 클래스의 필드를 테이블의 기본 키<sub>Primary key</sub>에 매핑한다. 이렇게 @Id가 사용된 필드를 식별자 필드라고 한다.

<br/>
<b>▽ @Column</b>
: 필드를 컬럼에 매핑한다. 이때 name 속성을 사용해서 엔티티의 필드를 테이블의 속성값을 이름으로 하는 컬럼에 매핑하도록 할 수 있다.

<br/>
<b>▽ 매핑 정보가 없는 필드</b>
: 매핑 애노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑한다.

이렇게 매핑 정보로 인해 JPA는 어떤 엔티티를 어떤 테이블에 저장해야 하는지 알 수 있다.

`cf. 자바 애노테이션의 패키지는 javax.persitence이다.`

<br/>
<hr/>
<br/>

## 2.5 persistence.xml 설정
JPA는 persistence.xml을 사용해서 필요한 설정 정보를 관리한다. 
이 설정 파일이 META-INF/persistence.xml 클래스 패스 경로에 있으면 별도의 설정 없이 JPA가 인식할 수 있다.

```xml
//예제 2.7 JPA 환경설정 파일 persistence.xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
    <persistence-unit name="jpabook">
        <properties>
            <!--필수 속성-->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!--옵션-->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.id.new_generator_mappings" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```
설정 파일은 persistence로 시작한다. 이곳에 XML 네임스페이스와 사용할 버전을 지정한다. 

JPA 설정은 영속성 유닛<sub>persistence-unit</sub>이라는 것부터 시작하는데, 일반적으로 연결할 데이터베이스당 하나의 영속성 유닛을 등록한다.
그리고 영속성 유닛에는 고유한 이름을 부여해야 한다.

&lt;properties&gt;를 통해 속성을 설정할 수 있다.

- <b>JPA 표준 속성</b>
    - javax.persistence.jdbc.driver: JDBC 드라이버
    - javax.persistence.jdbc.user: 데이터베이스 접속 아이디
    - javax.persistence.jdbc.password: 데이터베이스 접속 비밀번호
    - javax.persistence.jdbc.url: 데이터베이스 접속 URL
- <b>하이버네이트 속성</b>
    - hibernate.dialect: 데이터베이스 방언<sub>Dialect</sub> 설정

이름이 javax.persistence로 시작하는 속성은 JPA 표준 속성으로 특정 구현체에 종속되지 않는다. 
반면에 hibernate로 시작하는 속성은 하이버네이트 전용 속성이므로 하이버네이트에서만 사용할 수 있다.
<br/><br/>

### 2.5.1 데이터베이스 방언
JPA는 특정 데이터베이스에 종속적이지 않은 기술이다. 따라서 다른 데이터베이스로 쉽게 교차할 수 있다.
하지만 각 데이터베이스가 제공하는 SQL 문법과 함수가 일부 다르다는 문제점이 있다.

데이터베이스마다 다음과 같은 차이점이 있다.
- <b>데이터 타입</b>: 가변 문자 타입으로 MySQL은 VARCHAR를 사용하지만, 오라클은 VARCHAR2를 사용한다.
- <b>다른 함수명</b>: 문자열을 자르는 함수로 SQL 표준은 SUBSTRING()를 사용하지만 오라클은 SUBSTR()을 사용한다.
- <b>페이징 처리</b>: MySQL은 LIMIT를 사용하지만 오라클은 ROWNUM을 사용한다.

방언<sub>Dialect</sub>: JPA에서 SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능

애플리케이션 개발자가 특정 데이터베이스에 종속되는 기능을 많이 사용하면 이후에 데이터베이스를 교체하기가 어렵다.
대부분의 JPA 구현체들은 이런 문제를 해결하기 위해 다양한 데이터베이스 방언 클래스를 제공한다.

따라서 개발자는 JPA가 제공하는 표준 문법에 맞추어 JPA를 사용하고, 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 처리하게 함으로써 데이터베이스가 변경되어도 애플리케이션 코드를 변경할 필요 없이 데이터베이스 방언만 교체하면 된다.

하이버네이트는 다양한 데이터베이스 방언을 제공하는데 그에 대한 예시로는 다음과 같다.
- <b>H2</b>: org.hibernate.dialect.H2Dialect
- <b>MySQL</b>: org.hibernate.dialect.MySQL5InnoDBDialect

```
cf. 하이버네이트 전용 속성
- hibernate.show_sql: 하이버네이트가 실행한 SQL을 출력한다.
- hibernate.format_sql: 하이버네이트가 실행한 SQL을 출력할 때 보기 쉽게 정렬한다.
- hibernate.use_sql_comments: 쿼리를 출력할 때 주석도 함께 출력한다.
- hibernate.id.new_generator_mappings: JPA 표준에 맞춘 새로운 키 생성 전략을 사용한다.
```

```
JPA 구현체들은 보통 엔티티 클래스를 자동으로 인식하지만, 환경에 따라 인식하지 못할 때도 있다. 그때는 persistence.xml에 <class>를 사용해서 JPA에서 사용할 엔티티 클래스를 지정하면 된다.
```

<br/>
<hr/>
<br/>


## 2.6 애플리케이션 개발
객체 매핑을 완료하고 persistence.xml로 JPA 설정도 완료했으니 이제 JPA 애플리케이션을 개발해보자.

```java
//예제 2.8 시작 코드
public class JpaMain {
    public static void main(String[] args) {
        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
        //엔티티 매니저 생성
        EntityManager em = emf.createEntityManager();
        //트랜잭션 획득
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            logic(em);
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
    
    //비즈니스 로직
    private static void logic(EntityManager em) { ... }
}
```
코드는 크게 3부분으로 나누어져 있다.
- 엔티티 매니저 설정
- 트랜잭션 관리
- 비즈니스 로직
<br/><br/>

### 2.6.1 엔티티 매니저 설정
<b>▽ 엔티티 매니저 팩토리 생성</b>
<br/>
JPA를 시작하려면 우선 persistence.xml의 설정 정보를 사용해서 엔티티 매니저 팩토리를 생성해야 한다. 
이때 Persistence 클래스를 통해 엔티티 매니저 팩토리를 생성해서 JPA를 사용할 수 있게 준비한다.

`EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");`

위의 코드를 통해 META-INF/persistence.xml에서 이름이 "jpabook"인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성한다. 
이때 persistence.xml의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성하ㅡㅁ로 엔지지 매니저 팩토리를 생성하는 비용은 아주 크다.
따라서 <b>엔티티 매니터 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.</b>

<br/>
<b>▽ 엔티티 매니저 생성</b>
<br/>
엔티티 매니저 팩토리에서 엔지지 매니저를 생성한다. 특히 JPA의 기능 대부분은 이 엔티티 매니저가 제공한다. 
대표적으로 <b>엔티티 매니저를 사용해서 엔티티를 데이터베이스에 등록/수정/삭제/조회할 수 있다.</b> 엔티티 매니저는 내부에 데이터소스를 유지하면서 데이터베이스와 통신한다. 
참고로 <b>엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용하면 안된다.</b>

<br/>
<b>▽ 종료</b>
<br/>
마지막으로 사용이 끝난 엔티티 매니저는 종료해야 한다. 또한 애플리케이션을 종료할 때 엔티티 매니저 팩토리도 종료해야 한다. 
<br/><br/>

### 2.6.2 트랜잭션 관리
JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다. 만약 트랜잭션 없이 데이터를 변경할 시에는 예외가 발생한다.
이 트랜잭션을 시작하려면 엔티티 매니저<sub>EntityManage</sub>에서 트랜잭션 API를 받아와야 한다.

트랜잭션 API를 사용해서 비즈니스 로직이 정상 동작하면 트랜잭션을 커밋<sub>commit</sub>하고 예외가 발생하면 트랜잭션을 롤백<sub>rollback</sub>한다.
<br/><br/>

### 2.6.3 비즈니스 로직
회원 엔티티를 하나 생성한 다음 엔티티 매니저를 통해 데이터베이스에 등록/수정/삭제/조회한다.

```java
//예제 2.10 비즈니스 로직 코드
public static void logic(EntityManager em) {
    String id = "id1";
    Member member = new Member();
    member.setId(id);
    member.setUsername("김땡땡");
    member.setAge(2);
    
    //등록
    em.persist(member);
    
    //수정
    member.setAge(20);
    
    //한건 조회
    Member findMember = em.find(Member.class, id);
    System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());
    
    //목록 조회
    List<Member> members = 
        em.createQuery("select m from Member m", Member.class)
        .getResultList();
    System.out.println("members.size=" + members.size());
    
    //삭제
    em.remove(member);
}
```
출력 결과는 다음과 같다.<br/>
<b>findMembers=김땡땡, age=20<br/>
members.size=1</b>

비즈니스 로직을 보면 등록, 수정, 삭제, 조회 작업이 엔티티 매니저<sub>em</sub>를 통해서 수행되는 것을 알 수 있다.
이러한 엔티티 매니저는 객체를 저장하는 가상의 데이터베이스처럼 보인다. 등록, 수정, 삭제 코드에 대해서 자세히 알아보도록 하자.

<br/>
<b>▽ 등록</b>
<br/>
엔티티를 저장하려면 엔티티 매니저의 persist() 메소드에 저장할 엔티티를 넘겨주면 된다. 
그렇게 하면 JPA는 엔티티의 매핑 정보(애노테이션)를 분석해서 INSERT SQL을 만들어 데이터베이스에 전달한다.

<br/>
<b>▽ 수정</b>
<br/>
엔티티를 수정한 후에 수정 내용을 반영하기 위해서는 em.update() 같은 메소드를 호출해야할 것 같지만 단순히 엔티티의 값만 변경함으로써 수정이 가능하다.
그 이유는 JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있기 때문에 엔티티의 값만 변경하면 UPDATE SQL을 생성해서 데이터베이스에 값을 변경하게 된다.

<br/>
<b>▽ 삭제</b>
<br/>
엔티티를 삭제하려면 엔티티 매니저의 remove() 메소드에 삭제하려는 엔티티를 넘겨준다. 
그러면 JPA는 DELETE SQL을 생성해서 실행함으로써 삭제가 된다.

<br/>
<b>▽ 한 건 조회</b>
<br/>
find() 메소드는 조회할 엔티티 타입과 @Id로 데이터베이스 테이블의 기본 키와 매핑한 식별자 값으로 엔티티 하나를 조회하는 가장 단순한 조회 메소드이가. 
이 메소드를 호출하면 SELECT SQL을 생성해서 데이터베이스에 결과를 조회하고, 조회한 결과 값으로 엔티티를 생성해서 반환한다.
<br/><br/>

### 2.6.4 JPQL
JPA를 사용하면 애플리케이션 개발자는 엔티티 객체를 중심으로 개발하고 데이터베이스에 관한 처리는 JPA에 맡겨야 한다. 
앞에서 살펴본 등록/수정/삭제/한 건 조회에서는 SQL을 전혀 사용하지 않았지만 검색 쿼리는 다르다.
JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다.

만약 테이블이 아닌 엔티티 객체를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 불러와 엔티티 객체로 변경한 다음 검색해야 하지만 사실상 불가능하다.
따라서 애플리케이션이 필요한 데이터만 데이터베이스에서 불러오려면 결국 검색 조건이 포함된 SQL을 사용해야 한다. 
JPA는 JPQL<sub>Java Persistence Query Language</sub>이라는 쿼리 언어로 이 문제를 해결한다.

JPA는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공한다. JPQL은 SQL과 문법이 거의 유사한데, 둘의 차이점은 다음과 같다.
- JPQL은 <b>엔티티 객체</b>를 대상으로 쿼리한다. 즉, 클래스와 필드를 대상으로 쿼리한다.
- SQL은 <b>데이터베이스 테이블</b>을 대상으로 쿼리한다.

JPQL을 사용하려면 먼저 em.createQuery(JPQL, 반환타입) 메소드를 실행해서 쿼리 객체를 생성한 후 쿼리 객체의 getResultList() 메소드를 호출하면 된다.
JPA는 JPQL을 분석해서 적절한 SQL을 만들어 데이터베이스에서 데이터를 조회하게 된다.

<br/>
<hr/>
<br/><br/>
다음 포스트는 '자바 ORM 표준 JPA 프로그래밍' 책의 Chapter 3. 영속성 관리에 대한 글입니다. 감사합니다:)
