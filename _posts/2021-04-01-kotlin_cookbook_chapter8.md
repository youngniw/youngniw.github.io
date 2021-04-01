---
layout: post
toc: true
title: "[코틀린 쿡북] 8장 코틀린 대리자"
categories: Kotlin
tags: 코틀린_스터디
---

## Chapter 8, 코틀린 대리자
8장에서는 코틀린의 대리자에 대해 설명한다. 

<br/><br/>
### 8.1 대리자를 사용해서 합성 구현하기
다른 클래스의 인스턴스가 포함된 클래스를 만들고 그 클래스에 연산을 위임하고 싶을 때에는...<br/>
연산을 위임할 메소드가 포함된 인터페이스를 만들고, 클래스에서 해당 인터페이스를 구현한 다음, by 키워드를 사용해 바깥쪽에 래퍼 클래스를 만든다.

<br/><br/>

### 8.2 lazy 대리자 사용하기
어떤 속성이 필요할 때까지 해당 속성의 초기화를 지연하고 싶은 경우에는...<br/>
코틀린 표준 라이브러리의 lazy 대리자를 사용하자!

<br/><br/>

### 8.3 값이 널이 될 수 없게 만들기
처음 접근이 일어나기 전에 값이 초기화되지 않았다면 예외를 던지고 싶은 경우에는...<br/>
notNull 함수를 이용해, 값이 설정되지 않았다면 예외를 던지는 대리자를 사용하자!

<br/><br/>

### 8.4 observable과 vetoable 대리자 사용하기
속성의 변경을 가로채서, 필요에 따라 변경을 거부하고 싶은 경우에는...<br/>
변경 감지에는 observable 함수를 사용하고, 변경의 적용 여부를 결정할 때는 vetoable 함수와 람다를 사용하자!

<br/><br/>

### 8.5 대리자로서 Map 제공하기
여러 값이 들어있는 맵<sup>map</sup>을 제공해 객체를 초기화하고 싶은 경우에는...<br/>
코틀린 맵에는 대리자가 되는데 필요한 getValue와 setValue 함수 구현이 있으므로 이 함수들을 사용하자!

<br/><br/>

### 8.6 사용자 정의 대리자 만들기
어떤 클래스의 속성이 다른 클래스의 획득자와 설정자를 사용하게끔 만들고 싶은 경우에는...<br/>
ReadOnlyProperty 또는 ReadWriteProperty를 구현하는 클래스를 생성함으로써 직접 속성 대리자를 작성하자!

<br/><br/><br/>
<hr/>
<br/><br/><br/>

## Exersism 문제 풀이
이번 주에는 Exercism의 &#95;&#95;&#95;&#95;&#95; 문제를 준비해봤다.<br/>