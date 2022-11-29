---
layout: post
toc: true
title: "[Java] Stream 정리 - 소개 및 스트림 생성"
categories: 자바
tags: 자바, 자바8, 스트림
---

## 0. 들어가기에 앞서..
본 포스트는 책 "자바 8 인 액션"을 참고하여 작성한 것입니다.

<br/>
<hr/>

## 1. 스트림 Stream 소개

스트림은 람다를 활용할 수 있는 기술로, 자바 8에서 새로 추가된 기능이다.

자바 8 이전에는 배열 또는 컬렉션을 다루기 위해서는 for 문을 통해 요소 하나씩 순회하여 다루는 방법을 사용했다.
하지만 복잡한 데이터를 처리할 경우에는 코드의 양이 방대해질 뿐만 아니라, 복잡하고 어려워진다.

이러한 문제점을 해결하며 편리하게 데이터 컬렉션 반복을 처리할 수 있는 스트림에 대해 소개하고자 한다.

<br/>

### 1.1 스트림 소개

- 스트림: 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

> 스트림: 데이터 소스 + 중간 연산 + 최종 연산

- 스트림은 자바 API에 새로 추가된 기능으로, 스트림을 이용하면 선언형으로 컬렉션 처리 가능
  - 선언형: 데이터를 처리하는 임시 구현 코드 대신 질의로 표현
- 스트림을 이용하면 멀티 스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리 가능
  - stream() 대신에 parallelStream() 이용

<br/>
<b>[ 특징 ]</b>

1. 선언형: 더 간결하고 가독성이 좋음
2. 조립 가능: 유연성이 좋음
3. 병렬화: 성능이 좋음

<br/>
<b>[ 스트림 연산 특징 ]</b>

1. 파이프 라이닝
   - 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환
   - 게으름(laziness), 쇼트서킷(short-circuiting) 같은 최적화도 얻을 수 있음
   - collect를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환
2. 내부 반복
   - 반복을 알아서 처리하고 결과 스트림 값을 어딘가에 저장

- 컬렉션의 주제는 데이터, 스트림의 주제는 계산
  - 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 함
  - 스트림은 이론적으로 요청할 때만 요소를 계산 & 딱 한 번만 탐색할 수 있음
- 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서 유지
- 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산 지원
  - ex) filter, map, reduce, find, match, sort 등

<br/>
<hr/>

### 1.2 스트림 연산

- 중간 연산: 연결할 수 있는 스트림 연산 (스트림 반환)
  - 게으름 -> 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않음
  - 쇼트서킷: 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 되는 상황(즉시 결과 반환 가능)
  - 루프 퓨전: 서로 다른 연산이지만 한 과정으로 병합

- 최종 연산: 파이프라인을 실행해 스트림을 닫는 연산
  - 보통 스트림 이외의 결과 반환

<br/>
<hr/>

### 1.3 기본형 특화 스트림

- 자바 8에서는 3가지 기본형 특화 스트림을 제공
- 스트림 API는 박싱 비용을 피할 수 있도록 'int 요소에 특화된 `IntStream`', 'double 요소에 특화된 `DoubleStream`', 'long 요소에 특화된 `LongStream`'을 제공
- 각각의 인터페이스는 아래와 같이 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공
  - sum() : 숫자 스트림의 합계 계산
  - max() : 최댓값 요소 검색
  - min() : 최솟값 요소 검색

\+ 필요할 때 다시 객체 스트림으로 복원하는 기능 제공

-> 이러한 특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지는 않음

<br/>
<b>[ 숫자 스트림으로 매핑 ]</b>

- 스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong 3가지 메서드를 가장 많이 사용
- 해당 메서드는 map()과 동일한 기능을 수행하지만, Stream\<T> 대신 특화된 스트림을 반환
  * 반환 결과: IntStream, DoubleStream, LongStream
- 따라서 이후에는 숫자 관련 리듀싱 연산 수행 메서드 활용 가능
  
cf. IntStream 지원 메서드: max, min, average 등

<br/>
<b>[ 객체 스트림으로 복원 ]</b>

- 숫자 스트림을 만든 이후 복원하기 위해서는 스트림 인터페이스에 정의된 일반적인 연산(boxed 메서드) 사용
  - 예제 코드
  
    ```java
    // 스트림을 숫자 스트림으로 변환
    IntStream intStream = menu.stream().mapToInt(Dish::getCalories);

    // 숫자 스트림을 스트림으로 변환
    Stream<Integer> = intStream.boxed();
    ```
- IntStream에서는 mapToObj() 메서드를 통해 스트림으로 복원이 가능함

<br/>
<b>[ 기본 값으로의 OptionalInt ]</b>

- max 메서드 사용 시, 기본값이 0으로 설정되어 있기 때문에 스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 구별할 필요가 있음
- 따라서 max 메서드의 반환 결과로 OptionalInt를 반환
- 스트림에 요소가 없을 때의 최댯값을 명시적으로 설정하고자 한다면, 아래와 같이 작성

  `stream().___.max().orElse(값)`

<br/>
<hr/>

### 1.4 숫자 범위

- 특정 범위 숫자를 이용 시, 자바 8의 IntStream과 LongStream에서는 range와 rangeClosed 정적 메서드 제공
- range 메서드: 시작 값과 종료 값이 결과에 포함되지 않음
- rangeClosed 메서드: 시작 값과 종료 값이 결과에 포함
  
cf. 위의 2가지 메서드 사용 시, filter 메서드를 연결하면 숫자 범위 중 조건에 맞게 사용 가능

<br/>
<hr/>

### 1.5 스트림 생성

<b>[ 값으로 스트림 만들기 ]</b>

- Stream.of(임의의 값들) : 임의의 값을 인수로 받는 정적 메서드를 이용해 스트림 생성

  ```java
  Stream<String> stream = Stream.of("안", "녕", "하", "세", "요");
  ```
- Stream.empty() : 빈 스트림 생성

<br/>
<b>[ 배열로 스트림 만들기 ]</b>

- Arrays.stream(배열 변수명) : 배열을 인수로 받는 정적 메서드 Arrays.stream을 이용해 스트림 생성 가능
- 배열의 타입에 따라 생성되는 스트림 형태 또한 달라짐

|메서드 시그니처|반환 결과|
|--|--|
|Arrays.stream(T[] array)|Stream\<T>|
|Arrays.stream(int[] array)|IntStream| 	
|Arrays.stream(long[] array)|LongStream|
|Arrays.stream(double[] array)|DoubleStream|
|Arrays.stream(T[] array, int startInclusive, int endInclusive)|Stream\<T>|
	
- startInclusive와 endInclusive를 인자로 주어 시작 인덱스와 종료 인덱스(포함 X)를 설정할 수 있음 (위의 4가지 스트림 모두 가능)

<br/>
<b>[ 파일로 스트림 만들기 ]</b>

- 파일을 처리하는 등의 I/O 연산에 사용하는 자바의 NIO API도 스트림 API를 활용할 수 있도록 업데이트됨
- java.nio.file.Files의 많은 정적 메서드가 스트림 반환

- Files.lines(Path, Charset) : 주어진 파일의 행 스트림을 문자열 <u>스트림</u>으로 반환
- Files.readAllLines(Path, Charset) : 주어진 파일의 행을 문자열 <u>리스트</u>로 반환

<br/>
<b>[ 함수로 무한 스트림 만들기 ]</b>

- 스트림 API는 함수에서 스트림을 만들 수 있는 2개의 정적 메서드 Stream.iterate와 Stream.generate 제공
- 무한 스트림: 크기가 고정되지 않은 스트림
  
  -> 2개의 메서드로 만든 스트림은 요청할 때마다 주어진 함수를 이용해 값 생성

- 언바운드 스트림: 요청할 때마다 값을 생산할 수 있으며 끝이 없는 무한 스트림 생성하는 스트림
- 따라서 무한 스트림이 되지 않게 하기 위해 limit 메서드를 활용해 제한을 줘야 함

<br/>

- iterate 메서드
  * 초깃값과 람다를 인수로 받아 새로운 값을 끊임없이 생산할 수 있음
  * 기존 결과에 의존해서 순차적으로 연산을 수행
  * 일반적으로 연속된 일련의 값을 만들 때 사용

    <br/>
  * 피보나치 수열 예제 코드
  
    ```java
    Stream.iterate(new int[] {0,1}, i-> new int[]{i[1], i[0]+i[1]})
		.limit(10)
		.map(i -> i[0])
		.forEach(System.out::println);
    ```

<br/>

- generate 메서드
  * Supplier\<T>를 인수로 받아서 새로운 값을 생산
  * iterate와 달리 생상된 각 값을 연속적으로 계산하지 않음
  
  <br/>

  * 예제 코드 1
  
    ```java
	Stream.generate(Math::random)
		.limit(10)
		.forEach(System.out::println);
    ```
  
  <br/>

  * 예제 코드 2 (피보나치 수열)
  
    ```java
    IntStream.generate(new IntSupplier() {
      private int previous = 0;
      private int current = 1;

      public int getAsInt() {	// 오버라이드 해야 함
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;	
        this.current = nextValue;
        return oldPrevious;
      }
    })
    .limit(10)
    .forEach(System.out::println);
    ```
  * 공급자가 상태를 갖기 때문에 부작용과 병렬 스트림에서 문제 발생 가능

<br/>
<b>[ 병렬 스트림 ]</b>

- 병렬 스트림으로 만들 시에는 Stream에서 parallel 메서드 적용
- 병렬 스트림에서 순환 스트림으로 만들 시에는 sequential 메서드 적용

- 병렬 스트림이 올바르게 동작하려면 공유된 가변 상태를 피해야 함
- 병렬 스트림에서 박싱과 언박싱은 성능을 크게 저하시킬 수 있기에 기본형 특화 스트림(IntStream, LongStream, DoubleStream)을 사용하는 것이 좋음
- limit나 findFirst와 같이 요소의 순서에 의존하는 연산은 순차 스트림이 더 좋음
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않음
- iterate보다 range 팩토리 메서드로 만든 기본형 스트림이 더 분해하기 쉬움

- 자바 8은 컬렉션 프레임워크에 포함된 모든 자료 구조에 사용할 수 있는 디폴트 Spliterator 인터페이스 구현을 제공

<br/>
<hr/>

지금까지 자바의 스트림 소개 및 생성 방법에 대해 설명하는 포스트였습니다.

다음 포스트는 다양한 데이터 처리 질의를 표현할 수 있는 스트림에서 제공하는 주요 연산을 설명하겠습니다.

감사합니다:)
