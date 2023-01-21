---
layout: post
title: "[Java] Stream 정리 - 데이터 수집 collect"
categories: Java
tags: [Java, 자바, 자바8, 스트림]
---

## 0. 들어가기에 앞서..
본 포스트는 책 "자바 8 인 액션"을 참고하여 작성한 것입니다.

이번 포스트에서는 데이터 수집 시에 사용하는 컬렉터를 이용하여 스트림의 최종 결과를 도출하는 다양한 방법에 대해 설명하고자 합니다.

<br/>
<hr/>

## 3. 컬렉터

이전 포스트에서는 Collectors의 toList 메서드로 스트림 요소를 항상 리스트로만 반환하였다.

하지만 collect에서도 reduce와 같이 누적 방식을 인수로 받아 스트림을 최종 결과로 도출할 수 있도록 할 수 있다. 

또한 누적 연산 이외에도 다양한 컬렉터 파라미터를 collect 메서드에 전달함으로써 연산을 구현할 수 있다.

<br/>

### 3.1 컬렉터 소개
- Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출해낼지를 지정함
- Collectors 유틸리티 클래스: 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공
  - groupingBy, maxBy 등
  
<br/>

- Collectors에서 제공하는 메서드의 기능
  1. 스트림 요소를 하나의 값으로 리듀스하고 요약
  2. 요소 그룹화
  3. 요소 분할 (한 개의 인수를 받아 불린을 반환하는 Predicate\<T>를 함수로 사용)

<br/>
<hr/>

### 3.2 리듀싱과 요약

#### 3.2.1 개수 검색

- Collectors.counting() : 요소의 수 반환
  * 반환 타입: long
- stream.count()와 동일한 기능을 함

<br/>

#### 3.2.2 최댓값과 최솟값 검색

- Collectors.maxBy() : 인수의 함수를 이용해 스트림의 최댓값 반환
- Collectors.minBy() : 인수의 함수를 이용해 스트림의 최솟값 반환

<br/>

* 반환 타입: Optional\<T>
- 예제 코드
  
  ```java
  stream().collect(maxBy(Comparator.comparingInt(User::getAge)));
  ```

<br/>

#### 3.2.3 요약 연산

- 요약 연산: 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산

<br/>

1. 단순 합계
- Collectors.summingInt(): 객체를 int로 매핑하는 함수를 인수로 받아, 총합을 계산하여 반환
  * 반환 타입: int
- Collectors.summingDouble()
  * 반환 타입: double
- Collectors.summingLong()
  * 반환 타입: long
- 초깃값은 0

<br/>

2. 평균 값 계산
- Collectors.averagingInt(): 객체를 int로 매핑하는 함수를 인수로 받아, 총 평균을 계산하여 반환
  * 반환 타입: int
- Collectors.averagingDouble()
  * 반환 타입: double
- Collectors.averagingLong()
  * 반환 타입: long

<br/>

3. 혼합 계산
- 2개 이상의 연산을 한 번에 수행해야 할 때는 summarizing__()을 사용함
- 하나의 요약 연산으로 모든 연산의 결과 반환
- Collectors.summarizingInt(): IntSummaryStatics 클래스로 총 개수, 합계, 평균, 최댓값, 최솟값 등 모든 정보 수집
  * 반환 타입: IntSummaryStatistics
- Collectors.summarizingDouble()
  * 반환 타입: DoubleSummaryStatistics
- Collectors.summarizingLong()
  * 반환 타입: LongSummaryStatistics

<br/>

- IntSummaryStatistics 결과 예시 <br/>
  `{ count=3, sum=102, min=3, max=42, average=24.3423124 }`

<br/>

#### 3.2.4 문자열 연결

- Collectors.joining(): 스트림의 각 객체에 toString 메서드를 호출해, 추출한 모든 문자열을 하나의 문자열로 연결 후 반환
  - joining 메서드는 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만듦
  - 예제 코드
  
    ```java
    stream().map(User::getName).collect(joining(", "));
    ```

<br/>

#### 3.2.5 범용 리듀싱 요약 연산

- Collectors.reducing(): 위의 모든 컬렉터는 reducing 팩토리 메서드로 정의 가능함
    - reducing(초깃값, 변환 함수, 누적 및 연산 함수)
    - reducing(누적 및 연산 함수)
    - 초깃값 없이 누적 함수만을 인수로 가질 때에는 반환 타입이 Optional\<T>
  
    <br/>

    * 첫 번째 인수: 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때의 반환 값
    * 두 번째 인수: map(함수)와 같은 기능을 수행하는 변환 함수
    * 세 번째 인수: 같은 종류의 항목을 하나의 값으로 축약하는 BinaryOperator\<T>
      - BinaryOperator\<T>: 두 인수를 받아 같은 형식을 반환하는 함수

<br/>
<hr/>

### 3.3 그룹화

- Collections.groupingBy() : 인자로 받는 함수를 기준으로 스트림 그룹화 (= 분류 함수)
  - groupingBy(분류 함수, 컬렉터)
    * 첫 번째 인수: 스트림의 항목을 분류하는 기준인 분류 함수
    * 두 번째 인수: 그룹화된 각 요소들에 연산을 가할 컬렉터
      - groupingBy()를 내부에 한 번 더 사용하여 다수준 그룹화 가능

- 그룹화 연산의 결과로 그룹화 함수가 반환하는 키 그리고 각 키에 대응하는 스트림의 모든 항목 리스트를 값으로 갖는 맵 반환
  * 반환 타입: Map<K, List\<T>>

<br/>
<b>[ 다수준 그룹화 ]</b>

- Collectors.groupingBy() 메서드의 2번째 인수로 groupingBy를 전달해 두 수준으로 스트림의 항목을 그룹화 가능
- n 수준 그룹화의 결과는 n 수준 트리 구조로 표현되는 n 수준 맵

- 예제 코드
  
  ```java
  stream().collect(Collectors.groupingBy(User::getAge
		, groupingBy(user -> {
			if (user.getGender().equals("Male"))
				return "M";
			else
				return "F";
		})));
  ```

<br/>
<b>[ 서브그룹으로 데이터 수집 ]</b>

- groupingBy 메서드의 두 번째 인수로 계산하는 컬렉터를 전달해 그룹별 계산된 결과 반환
- 예제 코드

  ```java
  Map<String, Long> ageCount = userList.stream()
    .collect(Collectors.groupingBy(User::getAge, Collectors.counting());
  ```

> groupingBy(분류 함수) == groupingBy(분류 함수, Collectors.toList())

<br/>

- Collectors.collectingAndThen(): 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터 반환
  - collectingAndThen(컬렉터, 변환 함수)
  - 첫 번째 인수인 컬렉터의 값을 변환 함수로 변환하여 그룹화됨
  - 예제 코드
  
    ```java
    stream().collect(groupingBy(User::getAge, collectingAndThen(maxBy(comparingInt(User::getHeight)), Optional::get)));
    ```

<br/>

- Collectors.mapping() : 입력 요소를 누적하기 전에 매핑 함수를 적용해서 다양한 형식의 객체를 주어진 형식의 컬렉터에 맞게 변환
  - 스트림의 인수를 변환하는 함수와 변환 함수의 결과 객체를 누적하는 컬렉터를 인수로 가짐
  - 예제 코드
  
    ```java
    stream().collect(
      groupingBy(User::getAge, mapping(User::getGender, toSet())));
    ```
  - 예제 코드 결과 <br/>
  `{ 20=[Male, Female], 21=[Female], 23=[Male] }`

<br/>

- 형식을 지정하고 싶다면 stream().collect(컬렉터, toCollection(형식::new))
  - 예제 코드
  
    ```java
    stream().collect(groupingBy(User::getAge), toCollection(HashSet::new));
    ```

<br/>
<hr/>

### 3.4 분할

- Collectors.partitoiningBy() : Predicate\<T>를 분류 함수로 사용하는 특수한 그룹화 기능으로, 그룹화 맵은 최대 참, 거짓의 최대 2개의 그룹으로 분류됨
  - partitioningBy(분할 함수, 컬렉터)

<br/>

- 예제 코드
  
  ```java
  stream().collect(Collectors.partitioningBy(User::isAdult));
  ```
  
- 예제 코드 결과<br/>
  `{ true=[ann, young, eun], false=[kim] }`

> groupingBy는 함수로 그룹화한다면, partitioningBy는 프레디케이트로 2개의 그룹으로 나눔

<br/>

- 예제 코드 (소수-비소수 분류)

  ```java
  // 소수인지 확인하는 불린 함수
  public boolean isPrime(int number) {
  	return IntStream.rangeClosed(2, (int)Math.sqrt((double) number))
      		.noneMatch(i -> number%i == 0);
  }

  // 소수와 비소수 분류 함수
  public Map<Boolean, List<Integer>> partitionPrimes(int n) {
  	return IntStream.rangeClosed(2, n).boxed()
      		.collect(partitioningBy(num -> isPrime(num)));
  }
  ```

<br/>
<hr/>

### 3.5 정리

|팩토리 메서드|반환 형식|설명|
|--|--|--|
|toList|List\<T>|스트림의 모든 항목을 리스트로 수집|
|toSet|Set\<T>|스트림의 모든 항목을 중복이 없는 집합으로 수집|
|toCollection|Collection\<T>|스트림의 모든 항목을 공급자가 제공하는 컬렉션으로 수집|
|counting|Long|스트림의 항목 수 계산|
|summingInt|Integer|스트림의 항목에서 정수 프로퍼티의 합계 계산|
|averagingInt|Double|스트림의 항목에서 정수 프로퍼티의 평균값 계산|
|summarizingInt|IntSummaryStatistics|스트림 내의 항목의 최대, 최소, 합계, 평균 등의 정수 정보 통계 수집|
|joining|String|스트림의 각 항목에 toString 메서드를 호출한 결과 문자열 연결|
|minBy|Optional\<T>|주어진 비교자를 이용해 스트림의 최솟값 요소를 Optional로 감싼 값 반환<br/>(요소가 없을 시 Optional.empty() 반환)|
|maxBy|Optional\<T>|주어진 비교자를 이용해 스트림의 최댓값 요소를 Optional로 감싼 값 반환<br/>(요소가 없을 시 Optional.empty() 반환)|
|reducing|리듀싱 연산 반환 형식|누적자를 초깃값으로 설정한 다음, BinaryOperator로 스트림의 각 요소를 반복적으로 누적자와 합쳐 스트림을 하나의 값으로 리듀싱|
|collectingAndThen|변환 함수 반환 형식|다른 컬렉터를 감싸고, 그 결과에 변환 함수 적용|
|groupingBy|Map<K, List\<T>>|하나의 프로퍼티 값을 기준으로 스트림의 항목을 그룹화하며 기준 프로퍼티 값을 결과 맵의 키로 사용|
|partitioningBy|Map<Boolean, List\<T>>|Predicate를 스트림의 각 항목에 적용한 결과로 항목 분할|

<br/>
<hr/>

지금까지 자바 8의 컬렉터 관련 내용을 설명하는 마지막 자바 스트림 관련 포스트였습니다.

감사합니다:)
