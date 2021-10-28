---
layout: post
toc: true
title: "[JPA] 3. 영속성 관리"
categories: JPA
tags: 스프링부트, JPA
---

# &#91;자바 ORM 표준 JPA 프로그래밍&#93; Chapter 3. 영속성 관리

## 0. 들어가기에 앞서..
현재 이 포스트는 김영한님의 "자바 ORM 표준 JPA 프로그래밍 책"에 대한 내용을 바탕으로 작성하였습니다. 

<br/>
<hr/>
<br/>

JPA가 제공하는 기능은 크게 엔티티와 테이블을 매핑하는 설계 부분과 매핑한 엔티티를 실제 사용하는 부분으로 나눌 수 있다. 
이번 장에서는 매핑한 엔티티를 엔티티 매니저<sub>EntityManager</sub>를 통해 어떻게 사용하는지를 알아보고자 한다.

엔티티 매니저는 엔티티를 저장/수정/삭제/조회하는 등 엔티티와 관련된 모든 일을 처리한다. 
지금부터 이 엔티티 매니저를 자세히 알아보자:)

<br/>
<hr/>
<br/>

## 3.1 엔티티 매니저 팩토리와 엔티티 매니저
데이터베이스를 하나만 사용하는 애플리케이션은 일반적으로 EntityManagerFactory를 하나만 생성한다. 
엔티티 매니저 팩토리를 생성하는 코드는 다음과 같다.

`EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");`

위와 같이 호출하면 META-INF/persistence.xml에 있는 정보를 바탕으로 EntityManagerFactory를 생성한다.(비용이 많이 듦)

이후에는 필요할 때마다 엔티티 매니저 팩토리에서 엔티티 매니저를 생성하면 된다.(비용이 거의 안듦) 이에 대한 코드는 다음과 같다.

`EntityManager em = emf.createEntityManager();`

엔티티 매니저 팩토리는 생성하는 비용이 매우 크기 때문에 한 개만 만들어서 애플리케이션 전체에서 공유하도록 설계되어 있다. 
반면 엔티티 매니저는 생성하는 비용이 거의 들지 않는다.
또한 <b>엔티티 매니저 팩토리는 여러 스레드가 동시에 접근해도 안전하기 때문에 서로 다른 스레드 간에 공유해도 되지만, 엔티티 매니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로 절대로 스레드 간에 공유하면 안된다.</b>

엔티티 매니저는 데이터베이스 연결이 꼭 필요한 시점까지 커넥션을 얻지 않는다. 보통 트랜잭션을 시작할 때 커넥션을 획득한다.

하이버네이트를 포함한 JPA 구현체들은 EntityManagerFactory를 생성할 때 커넥션풀도 만드는데 이것은 J2SE 환경에서 사용하는 방법이다.
JPA를 J2EE 환경에서 사용하면 해당 컨테이너가 제공하는 데이터소스를 사용한다.

<br/>
<hr/>
<br/>

## 3.2 영속성 컨텍스트란?
영속성 컨텍스트<sub>persistence context</sub>는 <b>엔티티를 영구 저장하는 환경</b>이다. 
엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다.

`em.persist(member);`

이전 장에서는 단순히 위의 코드를 회원 엔티티를 저장한다고 말했었다. 
하지만 정확히 말하면 persist() 메소드는 <b>엔티티 매니저를 사용해서 회원 엔티티를 영속성 컨텍스트에 저장한다.</b>

영속성 컨텍스트는 논리적인 개념에 가깝고 눈에 보이지 않는데, 이것은 엔티티 매니저를 생성할 때 하나 만들어진다.
그리고 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있고, 영속성 컨텍스트를 관리할 수 있다.

`cf. 여러 엔티티 매니저가 같은 영속성 컨텍스트에 접근할 수 있다.`

<br/>
<hr/>
<br/>

## 3.3 엔티티의 생명주기
엔티티에는 4가지 상태가 존재한다.
- <b>비영속</b>(new/transient): 영속성 컨텍스트와 전혀 관계가 없는 상태
- <b>영속</b>(managed): 영속성 컨텍스트에 저장된 상태
- <b>준영속</b>(detached): 영속성 컨텍스트에 저장되었다가 분리된 상태
- <b>삭제</b>(removed): 삭제된 상태

<br/>
<b>▽ 비영속</b>
<br/>
```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```
엔티티 객체를 생성했다면 그것은 순수한 객체 상태이며 아직 저장되지 않은 것이다.
따라서 영속성 컨텍스트나 데이터베이스와는 전혀 관련이 없는데, 이것을 비영속 상태라고 한다.

<br/>
<b>▽ 영속</b>
<br/>
```java
//객체를 저장한 상태(영속)
em.persist(member);
```
엔티티 매니저를 통해 엔티티를 영속성 컨텍스트에 저장한다.
이렇게 <b>영속성 컨텍스트가 관리하는 엔티티를 영속 상태</b>라고 한다. 
생성 이후에 영속성 컨텍스트에 저장까지 되면 엔티티는 비영속 상태에서 영속 상태가 된다. 
<b>결국 영속 상태라는 것은 영속성 컨텍스트에 의해 관리된다는 뜻이다.</b>

또한 em.find() 나 JPQL을 사용해서 조회한 엔티티도 영속성 컨텍스트가 관리하는 영속 상태이다.

<br/>
<b>▽ 준영속</b>
<br/>
```java
//회원 엔티티를 영속성 컨텍스트에서 분리한 상태(준영속)
em.detach(member);
```
영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다. 
특정 엔티티를 준영속 상태로 만들려면 em.detach()를 호출하면 된다. 
또한 em.close()를 호출해서 영속성 컨텍스트를 닫거나 em.clear()를 호출해서 영속성 컨텍스트를 초기화해도 영속성 컨텍스트가 관리하던 영속 상태의 엔티티는 준영속 상태가 된다.

<br/>
<b>▽ 삭제</b>
<br/>
```java
//객체를 삭제한 상태(삭제)
em.remove(member);
```
엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.

<br/>
<hr/>
<br/>

## 3.4 영속성 컨텍스트의 특징
영속성 컨텍스트는 엔티티를 식별자 값(@Id로 테이블의 기본 키와 매핑한 값)으로 구분한다.
따라서 <b>영속 상태는 식별자 값이 반드시 있어야 한다.</b> 만약 식별자 값이 없으면 예외가 발생한다.

JPA는 보통 트랜잭션을 커밋하는 순간에 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영한다.
바로 이것을 <b>플러시</b><sub>flush</sub>라고 한다.

영속성 컨텍스트가 엔티티를 관리하면 다음과 같은 장점이 있다.
- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연
- 변경 감지
- 지연 로딩

지금부터 영속성 컨텍스트가 왜 필요하고 어떤 장점이 있는지 엔티티를 등록/조회/수정/삭제(CRUD)하면서 알아보고자 한다.
<br/><br/>

### 3.4.1 엔티티 조회
<b>▽ 1차 캐시 or 데이터베이스에서 조회</b>
<br/>
영속성 컨텍스트는 내부에 캐시를 가지고 있는데, 이것을 <b>1차 캐시</b>라고 한다. 영속 상태의 엔티티는 모두 이 1차 캐시에 저장된다.
쉽게 말하자면 영속성 컨텍스트 내부에 Map이 하나 존재하는데, 키는 @Id로 매핑한 식별자이고 값은 엔티티 인스턴스라고 할 수 있다.

엔티티를 생성한 후에(비영속) 엔티티를 persist()를 통해 영속시키면 1차 캐시에 엔티티가 저장이 된다. 
하지만 이 엔티티는 아직 데이터베이스에 저장이 되지 않은 상태이다.

1차 캐시의 키는 식별자 값인데, 이 식별자 값은 데이터베이스 기본 키와 매핑되어 있다. 
따라서 여속성 컨텍스트에 데이터를 저장하고 조회하는 모든 기준은 데이터베이스 기본 키 값이다.

저장 말고도 find() 메소드를 통해 조회했을 때를 생각해보자.
```java
//EntityManager.find() 메소드 정의
public <T> T find(Class<T> entityClass, Object primaryKey);
```
find() 메소드를 보면 첫 번째 파라미터는 엔티티 클래스의 타입이고, 두 번째 파라미터는 조회할 엔티티의 식별자 값이다.

em.find() 메소드를 호출하게 되면 먼저 1차 캐시에서 엔티티를 찾는다. 
만약 찾는 엔티티가 있다면 데이터베이스를 조회하지 않고 메모리에 있는 1차 캐시에서 엔티티를 조회한다.
반면에 찾는 엔티티가 1차 캐시에 없다면 엔티티 매니저는 데이터베이스를 조회해서 엔티티를 생성하고 1차 캐시에 저장한 후에 영속 상태의 엔티티를 반환한다.

따라서 1차 캐시에 엔티티 인스턴스가 있는 엔티티를 조회하면 메모리에 있는 1차 캐시에서 바로 불러올 수 있게 되므로 성능상 이점을 누릴 수 있다.

<br/>
<b>▽ 영속 엔티티의 동일성 보장</b>
<br/>
다음 코드를 통해 식별자가 같은 엔티티 인스턴스를 조회해서 비교해보자.
```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a == b); //동일성 비교
```
em.find(Member.class, "member1")를 반복해서 호출해도 영속성 컨텍스트는 1차 캐시에 잇는 같은 엔티티 인스턴스를 반환한다.
따라서 a와 b는 같은 인스턴스이기 때문에 결과는 참이다. 

<b><u>결론적으로 영속성 컨텍스트는 성능상 이점과 엔티티의 동일성을 보장한다.</u></b>
<br/><br/>

### 3.4.2 엔티티 등록
엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 모아둔다. 
그리고 트랜잭션을 커밋할 때 모아둔 쿼리를 데이터베이스에 보내는데 이것을 <b>트랜잭션을 지원하는 쓰기 지연</b><sub>transactional write-behind</sub>이라고 한다.

엔티티 매니저를 사용해서 엔티티를 영속성 컨텍스트에 등록하는 코드를 살펴보자.
```java
//예제 3.2 엔티티 등록 코드
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야 한다.
transaction.begin();    //트랜잭션 시작

em.persist(memberA);
em.persist(memberB);
//아직 INSERT SQL을 데이터베이스에 보내지 않음

//커밋하는 순간 데이터베이스에 INSERT SQL을 보냄
transaction.commit();   //트랜잭션 커밋
```
먼저 회원 A를 영속화했다. 영속성 컨텍스트는 1차 캐시에 회원 엔티티를 저장하면서 동시에 회원 엔티티 정보로 등록 쿼리를 만든다.
그리고 만들어진 등록 쿼리를 쓰기 지연 SQL 저장소에 보관한다.

마찬가지로 회원 B를 영속화했다. 이에 대한 수행 내용도 회원 A와 같다.
따라서 현재 쓰기 지연 SQL 저장소에는 등록 쿼리가 2건이 저장되어있게 된다.

마지막으로 트랜잭션을 커밋한다. 이때 트랜잭션을 커밋하면 엔티티 매니저는 우선 영속성 컨텍스트를 플러시한다.
플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업으로, 이때 등록/수정/삭제한 엔티티를 데이터베이스에 반영한다.
자세히 말하면 쓰기 지연 SQL 저장소에 모인 쿼리를 데이터베이스에 보낸다.
이렇게 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화한 후에 실제 데이터베이스 트랜잭션을 커밋한다.

<br/>
<b>▽ 트랜잭션을 지원하는 쓰기 지연이 가능한 이유</b>
<br/>
등록 쿼리를 그때 그때 데이터베이스에 전달하더라도 트랜잭션을 커밋하지 않으면 아무 소용이 없다. 
어떻게든 커밋 직전에만 데이터베이스에 SQL을 전달하면 되는데, 이것이 트랜잭션을 지원하는 쓰기 지연이 가능한 이유다.

이 기능을 잘 활용하면 모아둔 등록 쿼리를 데이터베이스에 한번에 전달해서 성능을 최적화할 수 있다.
<br/><br/>

### 3.4.3 엔티티 수정
<b>▽ SQL 수정 쿼리의 문제점</b>
<br/>
SQL을 사용하면 수정 쿼리를 직접 작성해야 한다. 이때 프로젝트가 점점 커지고 요구사항이 늘어나면서 수정 쿼리도 점점 추가된다.
수정쿼리를 상황에 따라 계속해서 추가하면서 개발하는 방식은 수정 쿼리가 많아지는 것은 물론이고 비즈니스 로직을 분석하기 위해 SQL을 계속 확인해야 한다.
<b>결국 직접적이든 간접적이든 비즈니스 로직이 SQL에 의존하게 된다.</b>

<br/>
<b>▽ 변경 감지</b>
<br/>
JPA로 엔티티를 수정할 때는 단순히 엔티티를 조회해서 데이터만 변경하면 된다. 
이것이 가능한 이유는 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능인 <b>변경 감지</b><sub>dirty checking</sub>가 존재하기 때문이다.

JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장해두는데 이것을 <b>스냅샷</b>이라고 한다.
그리고 플러시 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾는다.

엔티티 수정에 대한 순서는 다음과 같다.
1. 트랜잭션을 커밋하면 엔티티 매니저 내부에서 먼저 플러시가 호출된다.
2. 엔티티와 스냅샷을 비교해서 변경된 엔티티를 찾는다.
3. 변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보낸다.
4. 쓰기 지연 저장소의 SQL을 데이터베이스에 보낸다.
5. 데이터베이스 트랜잭션을 커밋한다.

<b>변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다.</b>
영속성 컨텍스트의 관리를 받지 못하는 엔티티는 값을 변경해도 데이터베이스에 반영되지 않는다.

JPA의 기본 전략은 엔티티의 모든 필드를 업데이트하는 것이다. 이에 대한 장점과 단점은 다음과 같다.

<b>모든 필드를 업데이트 시 장점</b>
- 수정 쿼리가 항상 같아 애플리케이션 로딩 시점에 수정쿼리를 미리 생성해두고 재사용할 수 있다.
- 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전에 한 번 파싱된 쿼리를 재사용할 수 있다.

<b>모든 필드를 업데이트 시 단점</b> 
- 데이터베이스에 보내는 데이터 전송량이 증가한다.

만약 필드가 많거나 저장되는 내용이 너무 크면 수정된 데이터만 사용해서 동적으로 UPDATE SQL을 생성하는 전략을 선택하면 된다.
예를 들어 @org.hibernate.annotations.DynamicUpdate를 사용해 적용이 가능하다. 

cf. 데이터를 저장할 때 데이터가 존재하는(null이 아닌) 필드만으로 INSERT SQL을 동적으로 생성하는 @DynamicInsert도 있다.
<br/><br/>

### 3.4.4 엔티티 삭제
em.remove()에 삭제 대상 엔티티를 넘겨주면 엔티티를 삭제한다. 
물론 엔티티를 즉시 삭제하는 것이 아니라 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록한다.
이후 트랜택션을 커밋해서 플러시를 호출하면 데이터베이스에 삭제 쿼리를 전달한다.

참고로 em.remove()를 호출하는 순간 넘겨준 엔티티는 영속성 컨텍스트에서 제거된다.

<br/>
<hr/>
<br/>

## 3.5 플러시
<b>플러시<sub>flush</sub>는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.</b>
1. 변경 감지가 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다. 그리고 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.

영속성 컨텍스트를 플러시하는 방법은 3가지가 있다.
1. em.flush()를 직접 호출한다.<br/>
-> 테스트나 다른 프레임워크와 JPA를 함께 사용할 때를 제외하고는 거의 사용하지 않는다.
2. 트랜잭션 커밋 시 플러시가 자동 호출된다.<br/>
-> 데이터베이스에 변경 내용을 SQL로 전달하지 않고 트랜잭션만 커밋하면 어떤 데이터도 데이터베이스에 반영되지 않는데, 이런 문제를 예방하기 위해 트랜잭션을 커밋할 때 플러시를 자동으로 호출한다.
3. JPQL 쿼리 실행 시 플러시가 자동 호출된다.<br/>
-> JPQL을 중간에 실행할 때, JPQL은 SQL로 변환되어 데이터베이스에서 엔티티를 등록/조회/수정/삭제를 한다. 그렇기 때문에 이전 내용들을 반영한 후에 JPQL의 SQL이 실행되어야 한다. 따라서 이런 문제를 예방하기 위해 JPQL을 실행할 때도 플러시를 자동 호출한다.

cf. 식별자를 기준으로 조회하는 find() 메소드를 호출할 때는 플러시가 실행되지 않는다.
<br/><br/>

### 3.5.1 플러시 모드 옵션
엔티티 매니저에 플러시 모드를 직접 지정하려면 javax.persistence.FlushModeType을 사용하면 된다.
- FlushModeType.AUTO: 커밋이나 쿼리를 실행할 때 플러시(기본값)
- FlushModeType.COMMIT: 커밋할 때만 플러시

플러시 모드를 별도로 설정하지 않으면 AUTO로 동작한다. COMMIT 모드는 성능 최적화를 위해 사용할 수 있다.

`em.setFlushMode(FlushModeType.COMMIT);     //플러시 모드 직접 설정`

<br/>
<hr/>
<br/>

## 3.6 준영속
영속석 컨텍스트가 관리하는 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된<sub>detached</sub> 것을 준영속 상태라고 한다. 
따라서 <b>준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.</b>

영속 상태의 엔티티를 준영속 상태로 만드는 방법은 다음과 같다.
1. em.detach(entity): 특정 엔티티만 준영속 상태로 전환한다.
2. em.clear(): 영속성 컨텍스트를 완전히 초기화한다.
3. em.close(): 영속성 컨텍스트를 종료한다.
<br/><br/>

### 3.6.1 엔티티를 준영속 상태로 전환: detach()
em.detach() 메소드는 특정 엔티티를 준영속 상태로 만든다. 
즉, 영속성 컨텍스트가 더는 해당 엔티티를 관리하지 않게 된다.
이 메소드를 호출하는 순간 1차 캐시부터 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 모든 정보가 제거된다.
<br/><br/>

### 3.6.2 영속성 컨텍스트 초기화: clear()
em.detach가 특정 엔티티 하나를 준영속 상태로 만들었다면 em.clear()는 영속성 컨텍스트를 초기화해서 해당 영속성 컨텍스트의 모든 엔티티를 준영속 상태로 만든다.
즉, 이것은 영속성 컨텍스트를 제거하고 새로 만든 것과 같다.
그리고 준영속 상태가 되면 영속성 컨텍스트가 지원하는 변경 감지는 동작하지 않아 변경이 일어나도 데이터베이스에 반영이 되지 않는다.
<br/><br/>

### 3.6.3 영속성 컨텍스트 종료: close()
영속성 컨텍스트를 종료하면 해당 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모두 준영속 상태가 된다. 
또한 1차 캐시도 사라지고 쓰기 지연 SQL 저장소도 사라진다.

cf. 영속 상태의 엔티티는 주로 영속성 컨텍스트가 종료되면서 준영속 상태가 된다. 개발자가 직접 준영속 상태로 만드는 일은 드물다.
<br/><br/>

### 3.6.4 준영속 상태의 특징
준영속 상태의 특징은 다음과 같다.
- 거의 비영속 상태와 가깝다. <br/>
: 영속성 컨텍스트가 관리하지 않기 때문에 영속성 컨텍스트가 제공하는 어떠한 기능도 동작하지 않는다.


- 식별자 값을 가지고 있다. <br/>
: 비영속 상태는 식별자 값이 없을 수 있지만, 준영속 상태는 이미 한 번 영속 상태였기 때문에 반드시 식별자 값을 가지고 있다.


- 지연 로딩을 할 수 없다. <br/>
: 지연 로딩<sub>lazy loading</sub>은 실제 객체 대신 프록시 객체를 로딩해두고 해당 객체를 실제 사용할 때 영속성 컨텍스트를 통해 데이터를 불러오는 방법이다.
하지만 준영속 상태는 영속성 컨텍스트가 더이상 관리하지 않기 때문에 지연 로딩 시에 문제가 발생한다.
<br/><br/>

### 3.6.5 merge()
준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다.
merge() 메소드는 준영속 상태의 엔티티를 받아서 그 정보로 <b><u>새로운</u> 영속 상태의 엔티티를 반환한다.</b>

```java
//예제 3.11 merge() 메소드 정의
public <T> T merge(T entity);
```

<br/>
<b>▽ 준영속 병합</b>
<br/>
주의할 점은 엔티티가 준영속 상태에서 영속 상태로 변경되는 것이 아니고 새로운 영속 상태의 엔티티가 반환되는 것이다.

merge()의 동작 방식은 다음과 같다.
1. merge()를 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고 1차 캐시에 저장한다.
3. 조회한 영속 엔티티에 넘겨받은 엔티티의 값을 채워 넣는다.
4. 새로운 영속 상태의 엔티티를 반환한다.

따라서 merge()는 파라미터로 넘어온 준영속 엔티티를 사용해서 새롭게 병합된 영속 상태의 엔티티를 반환한다. 
반면에 파라미터로 넘어온 엔티티는 병합 후에도 준영속 상태로 남아 있다.
그래서 준영속 엔티티를 참조하던 변수를 영속 엔티티를 참조하도록 변경하는 것이 안전한다.

<br/>
<b>▽ 비영속 병합</b>
<br/>
병합은 비영속 엔티티도 영속 상태로 만들 수 있다. 

<br/>
결론적으로 병합은 준영속, 비영속을 신경쓰지 않는다. 단지 식별자 값으로 엔티티를 조회할 수 있으면 불러서 병합하고 조회할 수 없으면 새로 생성해서 병합한다.
따라서 병합은 save or update 기능을 수행한다.

<br/>
<hr/>
<br/><br/>
다음 포스트는 '자바 ORM 표준 JPA 프로그래밍' 책의 Chapter 4. 엔티티 매핑에 대한 글입니다. 감사합니다:)