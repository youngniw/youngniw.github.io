---
layout: post
title: "[Kotlin] Exercism Triangle 문제 설명/풀이"
categories: Kotlin
tags: 코틀린_Exercism
---

## Exercism 문제: Triangle
이번 주에는 Exercism의 Triangle이라는 문제를 준비해보았다.
<br/><br/><br/>

### &#91;문제 설명&#93;<br/>
3개의 변의 길이가 주어질 때, 그 삼각형이 어떤 삼각형인지를 확인하는 클래스를 구현해야 한다.

<b>Equilateral triangle</b>은 세 변의 길이가 모두 같은 삼각형이다. 
또한 <b>Isosceles triangle</b>은 적어도 두 변의 길이가 같은 삼각형을 말한다. (보통은 반드시 두 변의 길이가 같아야 한다고 말하지만 이 문제에서는 적어도 두 변을 말함)
마지막으로 <b>Scalene triangle</b>은 세 변의 길이가 모두 다른 삼각형을 말한다.

<참고> 모양이 삼각형이 되려면 모든 변의 길이가 0보다 커야 하며, 두 변의 길이의 합은 세 번째 변의 길이보다 크거나 같아야 한다.

주어진 코드는 다음과 같다.

```kotlin
class Triangle<out T : Number>(val a: T, val b: T, val c: T) {
	// TODO: Implement proper constructor

	val isEquilateral: Boolean = TODO("Implement this getter to complete the task")
	val isIsosceles: Boolean = TODO("Implement this getter to complete the task")
  	val isScalene: Boolean = TODO("Implement this getter to complete the task")
}
```

<br/><br/><br/>

### &#91;문제 풀이&#93;<br/>
내가 문제의 해답으로 작성한 코드는 다음과 같다.

```kotlin
class Triangle<out T : Number>(val a: T, val b: T, val c: T) {
	init {
		val length1: Double = a.toDouble()		//Number의 가장 큰 범위는 Double
		val length2: Double = b.toDouble()
		val length3: Double = c.toDouble()
		
		//예외처리
		if (length1 <= 0 || length2 <= 0 || length3 <= 0)
	    		throw IllegalArgumentException("The length of all sides must be greater than zero.")
	  	if ((length1 + length2 < length3) || (length1 + length3 < length2) || (length2 + length3 < length1))
	    		throw IllegalArgumentException("The sum of the lengths of both sides must be greater than or equal to the length of the other side.")
	}
	
	//정삼각형(3개의 변이 모두 같음)
  	val isEquilateral: Boolean by lazy { a==b && b==c }
  	//이등변삼각형(적어도 2개의 변이 같음)
	val isIsosceles: Boolean by lazy { a==b || b==c || c==a }	
  	//3개의 변 모두 길이가 다름
	val isScalene: Boolean by lazy { !isIsosceles }
}
```

클래스를 인스턴스화한 객체가 생성될 때 init 영역이나 생성자를 통해 초기화 작업을 수행할 수 있다. 이 문제에서는 이러한 초기화 작업이 필요하다.
문제의 목적은 어떤 삼각형인지를 확인하는 것이지만, 먼저 3개의 변의 길이로 삼각형이 생성되는 지를 확인해야 하므로 init 영역을 구현하였다.

init 영역에는 인자로 넘겨받은 삼각형의 3개의 변의 길이 a, b, c를 각각 Double로 변환하여 length1, length2, length3에 저장한다. 
그리고 if문을 통해 각각의 값이 0보다 큰 지의 여부를 확인해 하나라도 크지 않을 경우에는 throw를 통해 IllegalArgumentException을 넘기게 된다.
또다른 if문을 통해 두 변의 길이의 합이 다른 한 변의 길이보다 작다면 마찬가지로 IllegalArgumentException을 넘기게 된다.

init 영역 이외의 Triangle클래스 내에는 삼각형이 어떤 삼각형인지를 확인할 수 있는 변수를 가져야 한다. 
특정 삼각형인지를 확인할 수 있는 각각의 isEquilateral, isIsosceles, isScalene 변수는 by 키워드와 lazy대리자를 통해 변수의 호출 시점에 초기화를 진행하게 한다.

먼저 isEquilateral 변수는 초기화 시 람다를 통해 세 변의 길이가 모두 같은 지를 확인하고, 모두 같다면 true를 저장하고 그렇지 않다면 false를 저장한다.

isIsosceles 변수는 초기화 시 람다를 통해 적어도 두 변의 길이가 같은 지 확인하고, 그렇다면 true를 저장하고 그렇지 않다면 false를 저장한다.

isScalene 변수는 람다를 통해 세 변의 길이가 모두 다른 지를 확인하여, 모두 같다면 true를 저장하고, 그렇지 않다면 false를 저장한다.
이때 세 변의 길이가 모두 다른 지 확인하는 방법은 각각의 원소를 비교하는 것이 아니라, 적어도 2변이 같은 삼각형이 아니면 되기에 변수 isIsosceles의 반대값을 저장한다.


<br/><br/>
<hr/>
<br/><br/>

지금까지 Exercism의 Triangle문제를 Kotlin으로 풀이하였습니다:)
