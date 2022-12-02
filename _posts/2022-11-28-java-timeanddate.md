---
layout: post
toc: true
title: "[Java] 날짜와 시간 정리 - LocalDate, LocalTime, Instant, Duration, Period"
categories: 자바
tags: 자바, 자바8
---

## 0. 들어가기에 앞서..
본 포스트는 책 "자바 8 인 액션"을 참고하여 작성한 것입니다.

이번 포스트에서는 자바 8에서 제공하는 새로운 날짜와 시간 API에 대해 설명하고자 합니다.

<br/>

> 핵심 부분 바로가기!

- [2.1.1 LocalDate](#211-localdate)
- [2.1.2 LocalTime](#212-localtime)
- [2.1.3 LocalDateTime](#213-localdatetime)
- [2.2 Instant](#22-instant)
- [2.3.1 Duration](#231-duration)
- [2.3.2 Period](#232-period)

<br/>
<hr/>

## 1. java.time 패키지

자바 1.0에서는 java.util.Date 클래스를 통해 날짜와 시간 관련 기능을 제공했다.

날짜의 생성 방법은 다음과 같다.

> Date date = new Date([1900년을 기준으로 하는 오프셋 연도], [0에서 시작하는 달/월 인덱스], [일]);

위와 같이 날짜로 연도의 기준을 1900년으로 했으며 월의 시작 인덱스가 1이 아닌 0이라는 점과 같이 직관적이지 못하다는 문제점이 있다.
뿐만 아니라 Date 클래스의 toString으로 반환되는 문자열을 활용하기가 어렵다는 등의 여러 가지의 문제가 있었다.

따라서 이후의 자바 1.1에서는 java.util.Calendar 클래스를 제공했다.
하지만 해당 클래스 역시 오류가 있으며, 기능에 대한 호환이 되지 않았다.

이러한 문제점을 해결하기 위해, 자바 8에서는 **java.time 패키지**에서 새로운 날짜와 시간 API를 통해 새로운 기능을 제공한다.

|인터페이스|구현체|
|--|--|
|Temporal|LocalDate, LocalTime, LocalDateTime, Instant|
|TemporalField|ChronoField|

<br/>
<hr/>

## 2. LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period

날짜와 시간, 그리고 시간 간격을 정의하기 위해 java.time 패키지에서는 LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period 등의 클래스를 제공한다.

<br/>

### 2.1 LocalDate와 LocalTime, LocalDateTime

- `LocalDate` 인스턴스: 시간을 제외한 날짜를 표현하는 불변 객체
- `LocalTime` 인스턴스: 시간을 표현하는 불변 객체
- `LocalDateTime` 인스턴스: 날짜와 시간 모두를 표현하는 불변 객체

#### 2.1.1 LocalDate

**[ 인스턴스 생성 방법 ]**
- 정적 팩토리 메서드 of
  * 첫 번째 인수: 연도
  * 두 번째 인수: 월
  * 세 번째 인수: 일

  - 예제 코드

    ```java
    LocalDate date = LocalDate.of(2022, 11, 28);  // 2022-11-28
    ```

- 팩토리 메서드 now
  - 시스템 시계의 정보를 이용한 현재 날짜 정보 반환
  
  * 가능한 인수 1) ZoneId: 타임존으로, 보통 of 메서드를 이용해 타임존을 지정
    - `ZoneId.of("Asia/Seoul");`
  
  * 가능한 인수 2) Clock

  - 예제 코드

    ```java
    LocalDate date = LocalDate.now();             // 2022-11-28
    LocalDate date1 = LocalDate.now(ZoneId.of("Asia/Seoul"));
    LocalDate date2 = LocalDate.now(Clock.systemDefaultZone());
    ```

- parse 메서드
  - 날짜 문자열로 LocalDate 인스턴스 생성
  - 파싱 할 수 없을 때: DateTimeParseException 예외 발생
  
  * 첫 번째 인수: 날짜 문자열
  * 두 번째 인수: DateTimeFormatter (생략 가능)

  - 예제 코드

    ```java
    LocalDate date = LocalDate.parse("2022-11-28", DateTimeFormatter.ofPattern("yyyy-MM-dd"));
    ```

<br/>

**[ 날짜 정보 조회 메서드 ]**

|메서드|설명|
|--|--|
|int get(TemporalField field)<br/>long getLong(TemporalField field)|해당 날짜 객체의 명시된 필드의 값을 int형이나 long형으로 반환|
|int getYear()|해당 날짜 객체의 연도 값을 반환|
|Month getMonth()|해당 날짜 객체의 월 값을 Month 열거체로 반환<br/>ex) JANUARY, FEBRUARY 등|
|int getMonthValue()|해당 날짜 객체의 월 값을 반환 (1~12)|
|int getDayOfMonth()|해당 날짜 객체의 일 값을 반환(1~31)|
|int getDayOfYear()|해당 날짜 객체의 해당 연도에서의 일 값을 반환(1~366)|
|DayOfWeek getDayOfWeek()|해당 날짜 객체의 요일의 값을 DayOfWeek 열거체로 반환<br/>ex) MONDAY, TUESDAY 등|
|int lengthOfYear|해당 연도의 총 일 수|
|int lengthOfMonth|해당 월의 총 일 수|
|boolean isLeapYear|해당 날짜가 윤년인지 여부|

- TemporalField: 시간 관련 객체에서 어떤 필드 값에 접근할지 정의하는 인터페이스
  - 열거자 ChronoField: TemporalField 인터페이스를 정의함
  - ex) `ChronoField.DAY_OF_YEAR`, `ChronoField.DAY_OF_MONTH`

- 예제 코드
  
  ```java
  LocalDate date = LocalDate.now();     // 2022-11-28

  int yearTF = date.get(ChronoField.YEAR);          // 2022
  int monthTF = date.get(ChronoField.MONTH_OF_YEAR);// 11
  int dayTF = date.get(ChronoField.DAY_OF_MONTH);   // 28

  int year = date.getYear();            // 2022
  Month month = date.getMonth();        // NOVEMBER
  int monthValue = date.getMonthValue();// 11
  int day = date.getDayOfMonth();       // 28
  int dayYear = date.getDayOfYear();    // 332 (해당 연도의 몇 번째 일)
  DayOfWeek week = date.getDayOfWeek(); // MONDAY
  int lenYear = date.lengthOfYear();    // 365 (해당 연도의 일수)
  int lenMonth = date.lengthOfMonth();  // 30 (해당 월의 일수)
  boolean leap = date.isLeapYear();     // false (윤년이 아님)
  ```

<br/>

#### 2.1.2 LocalTime

**[ 인스턴스 생성 방법 ]**
- 정적 팩토리 메서드 of
  * 첫 번째 인수: 시
  * 두 번째 인수: 분
  * 세 번째 인수: 초 (생략 가능)
  * 네 번째 인수: 나노초 (생략 가능)

  - 예제 코드

    ```java
    LocalTime time = LocalTime.of(15, 3, 21); // 15:03:21
    ```

- 팩토리 메서드 now
  - 시스템 시계의 정보를 이용해서 현재 시간 정보를 얻음

  - 예제 코드
  
    ```java
    LocalTime time = LocalTime.now();         // 15:03:21
    LocalTime time1 = LocalTime.now(ZoneId.of("Asia/Seoul"));
    LocalTime time2 = LocalTime.now(Clock.systemDefaultZone());
    ```

- parse("시:분:초") 메서드
  - 시간 문자열로 LocalTime 인스턴스 생성
  - 파싱 할 수 없을 때: DateTimeParseException 예외 발생
  * 첫 번째 인수: 시간 문자열
  * 두 번째 인수: DateTimeFormatter (생략 가능)

  - 예제 코드

    ```java
    LocalTime time = LocalTime.parse("15:03:21", DateTimeFormatter.ofPattern("HH:mm:ss"));
    ```

**[ 시간 정보 조회 메서드 ]**

|메서드|설명|
|--|--|
|int get(TemporalField field)<br/>long getLong(TemporalField field)|해당 시간 객체의 명시된 필드의 값을 int형이나 long형으로 반환|
|int getHour()|해당 시간 객체의 시 값을 반환 (0~23)|
|int getMinute()|해당 시간 객체의 분 값을 반환 (0~59)|
|int getSecond()|해당 시간 객체의 초 값을 반환 (0~59)|
|int getNano()|해당 시간 객체의 나노초 값을 반환 (1~999,999,999)|

<br/>

#### 2.1.3 LocalDateTime

**[ 인스턴스 생성 방법 ]**

- 생성 방법
  1. 직접 생성: of(연, 월, 일, 시, 분, 초, 나노초)
     * 첫 번째 인수: 연
     * 두 번째 인수: 월 (Month 혹은 int 형식)
     * 세 번째 인수: 일
     * 네 번째 인수: 시
     * 다섯 번째 인수: 분
     * 여섯 번째 인수: 초 (생략가능)
     * 일곱 번째 인수: 나노초 (생략가능)

  2. 날짜와 시간 조합
     - of(날짜, 시간)
       * 첫 번째 인수: 날짜 (LocalDate)
       * 두 번째 인수: 시간 (LocalTime)
     - 날짜.atTime(시간)
     - 시간.atTime(날짜)
  
- 인스턴스에서 날짜와 시간을 추출하는 방법
  1. 날짜 추출: 객체.toLocalDate()
  2.  시간 추출: 객체.toLocalTime()

<br/>
<hr/>

### 2.2 Instant

- Instant: 기계적인 관점에서 시간을 표현 즉, 유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지의 시간을 초로 표현
- 나노초(10억분의 1초)의 정밀도 제공

<br/>

- 팩토리 메서드 ofEpochSecond
  - 초를 인자로 주어 인스턴스 생성
  * 첫 번째 인수: 기준 이후의 시간 초
  * 두 번째 인수: 나노초 단위로 시간 보정 (0~999,999,999) (생략 가능)

  <br/>

  - 예제 코드
  
    ```java
    Instant.ofEpochSecond(3);
    Instant.ofEpochSecond(3, 0);
    Instant.ofEpochSecond(2, 1_000_000_000);    // 2초 이후의 1초
    Instant.ofEpochSecond(4, -1_000_000_000);   // 4초 이전의 1초
    ```
    > 위의 4가지 코드는 동일한 Instant를 의미한다.

- 팩토리 메서드 now
  - 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻음
  * 가능한 인수: Clock
  
  <br/>

  - 예제 코드
  
    ```java
    Instant now = Instant.now();
    ```

- 단, 사람이 읽을 수 있는 시간 정보를 제공하지 않음
- Instant에서는 Duration과 Period 클래스를 함께 활용 가능

<br/>
<hr/>

### 2.3 Duration과 Period

- Duration: 시간 객체 사이의 지속시간
- Period: 시간의 간격을 연, 월, 일로 표현

<br/>

#### 2.3.1 Duration

- 정적 팩토리 메서드 between: 두 시간 객체 사이의 지속시간 생성
  - Duration.between(LocalTime, LocalTime)
  - Duration.between(LocalDateTime, LocalDateTime)
  - Duration.between(Instant, Instant)
- Duration 클래스는 초와 나노초로 시간 단위를 표현하기 때문에 between 메서드에 LocalDate를 인자로 줄 수 없음

<br/>

#### 2.3.2 Period

- 정적 팩토리 메서드 between: 두 날짜 객체 사이의 차이 생성
  - Period.between(LocalDate, LocalDate)


- Duration과 Period 제공 공통 메서드

|메서드|설명|
|--|--|
|between|두 시간 사이의 간격 생성|
|from|시간 단위로 간격 생성|
|of|주어진 구성 요소에서 간격 인스턴스 생성|
|parse|문자열을 파싱해 간격 생성|
|addTo|현재 값의 복사본을 생성 후, 지정된 Temporal 객체에 추가|
|long get(TemporalUnit)|현재 간격 정보 값 조회|
|boolean isNegative()|간격이 음수인지 여부|
|boolean isZero()|간격이 0인지 여부|
|plus|현재 값에서 주어진 시간을 더한 값 복사본 생성|
|minus|현재 값에서 주어진 시간을 뺀 값 복사본 생성|
|multipliedBy|현재 값에서 주어진 값을 곱한 값 복사본 생성|
|negated|주어진 값의 부호를 반전한 값 복사본 생성|
|subtractFrom|지정된 Temporal 객체에서 간격을 뺌|

<br/>
<hr/>

## 3. 날짜 조정과 파싱, 그리고 포매팅

### 3.1 날짜 조정

- get과 with 메서드로 Temporal 객체의 필드값을 읽거나 고칠 수 있다. 
  - 고칠 시: 복사본 생성
  
  - 예제 코드
    ```java
    LocalDate date = LocalDate.now(); // 2022-11-28
    date = date.withYear(2021);       // 2021-11-28
    ```
	
- 이외의 LocalDate, LocalTime, LocalDateTime, Instant 등 날짜와 시간을 표현하는 모든 클래스는 아래의 비슷한 메서드를 제공한다.

|메서드|설명|
|--|--|
|from|주어진 Temporal 객체를 이용해 인스턴스 생성|
|now|시스템 시계로 Temporal 객체 생성|
|of|주어진 구성 요소로 Temporal 객체 인스턴스 생성|
|parse|문자열을 파싱해 Temporal 객체 생성|
|atOffset|시간대 오프셋과 Temporal 객체를 합침|
|atZone|시간대와 Temporal 객체를 합침|
|format|지정된 포맷을 이용해 Temporal 객체를 문자열로 변환<br/>(Instant 지원 X)|
|get|Temporal 객체의 상태를 조회|
|plus|특정 시간을 더한 Temporal 객체의 복사본 생성|
|minus|특정 시간을 뺀 Temporal 객체의 복사본 생성|
|with|일부 상태를 바꾼 Temporal 객체의 복사본 생성|

<br/>

- TemporalAdjusters의 팩토리 메서드

|메서드|설명|
|--|--|
|dayOfWeekInMonth|서수 요일에 해당하는 날짜를 반환하는 TemporalAdjusters 반환|
|firstDayOfMonth|현재 달의 첫 번째 날짜를 반환하는 TemporalAdjusters 반환|
|firstDayOfNextMonth|다음 달의 첫 번째 날짜를 반환하는 TemporalAdjusters 반환|
|firstDayOfNextYear|내년의 첫 번째 날짜를 반환하는 TemporalAdjusters 반환|
|firstDayOfYear|올해의 첫 번째 날짜를 반환하는 TemporalAdjusters 반환|
|firstInMonth|현재 달의 첫 번째 요일에 해당하는 날짜를 반환하는 TemporalAdjusters 반환|
|lastDayOfMonth|현재 달의 마지막 날짜를 반환하는 TemporalAdjusters 반환|
|lastDayOfNextMonth|다음 달의 마지막 날짜를 반환하는 TemporalAdjusters 반환|
|lastDayOfYear|올해의 마지막 날짜를 반환하는 TemporalAdjusters 반환|
|lastInMonth|현재 달의 마지막 요일에 해당하는 날짜를 반환하는 TemporalAdjusters 반환|
|next(DayOfWeek)|현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjusters 반환|
|previous(DayOfWeek)|현재 날짜 이후에서 역으로 날짜를 거슬러 올라가며 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjusters 반환|
|nextOrSame(DayOfWeek)|현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjusters 반환 (현재 날짜 포함)|
|previousOrSame(DayOfWeek)|현재 날짜 이후에서 역으로 날짜를 거슬러 올라가며 지정한 요일이 처음으로 나타나는 날짜를 반환하는 TemporalAdjusters 반환 (현재 날짜 포함)|

- 예제 코드
  ```java
  LocalDate date = LocalDate.now();     // 2022-11-28
  LocalDate date1 = date.with(TemporalAdjusters.nextOrSame(DayOfWeek.SATURDAY));    // 2022-12-03
  LocalDate date2 = date.with(TemporalAdjusters.lastDayOfYear());           // 2022-12-31
  ```

<br/>

### 3.2 파싱과 포맷팅
- 파싱: DateTimeFormatter.parse("문자열", DateTimeFormatter)
  * 반환 타입: 날짜 및 시간
- 포맷팅: DateTimeFormatter.format(DateTimeFormatter) 
  * 반환 타입: 문자열

- DateTimeFormatter는 스레드 안정성 보장

<br/>

cf. ZoneId를 이용해 LocalDateTime <-> Instant 변환 방법
  1. LocalDateTime->Instant: LocalDateTime 인스턴스에 toInstant 메서드 호출
     * 첫 번째 인수: ZoneId
  2. Instant->LocalDateTime: LocalDateTime.ofInstant 메서드 호출
     * 첫 번째 인수: Instant 객체
     * 두 번째 인수: ZoneId

> LocalDateTime = LocalDate + LocalTime
> 
> ZoneDateTime = LocalDate + LocalTime + ZoneId
>
> cf. ZoneOffset: 시차

<br/>
<hr/>

지금까지 자바 8의 새롭게 추가된 날짜 및 시간과 관련한 API를 설명하였습니다.

감사합니다:)
