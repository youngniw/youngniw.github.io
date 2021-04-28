---
layout: post
title: "[Kotlin] Exercism의 All Your Base 문제 설명/풀이"
categories: Kotlin
tags: 코틀린_Exercism
---

## Exercism 문제: All Your Base
이번 주에는 Exercism의 "All Your Base" 문제를 준비해보았다.
<br/><br/><br/>

### &#91;문제 설명&#93;<br/>
All Your Base 문제는 어떤 진법<sup>base</sup>으로 표현된 일련의 숫자를 다른 진법<sup>base</sup>로 변환하는 문제이다. <br/>
일반적인 진법 변환을 구현함으로써 a 진법으로 숫자가 표현된 것을 b 진법으로 숫자를 표현하도록 한다.

```
예를 들어 보면,
-> 10진법으로 표현한 숫자 51은 (5*10^1) + (1*10^0)이다.
-> 2진법으로 변환하면 (1*2^5)+(1*2^4)+(0*2^3)+(0*2^2)+(1*2^1)+(1*2^0)로 인해 110011이다.

따라서 다른 기저가 10인 51을 기저 2로 변환했을 때 함수를 통해 반환해야 하는 값을 110011이다.
```


<br/>
주어진 구현할 코드는 다음과 같다.

```kotlin
class BaseConverter {
    // TODO: implement proper constructor to complete the task

    fun convertToBase(newBase: Int): IntArray {
        TODO("Implement the function to complete the task")
    }
}
```

위의 구현될 코드를 테스트하는 코드의 일부는 다음과 같다.
```kotlin
@Test
fun testBinaryToSingleDecimal() {
    val baseConverter = BaseConverter(2, intArrayOf(1, 0, 1))

    val expectedDigits = intArrayOf(5)
    val actualDigits = baseConverter.convertToBase(10)

    assertArrayEquals(
        "Expected digits: ${Arrays.toString(expectedDigits)} but found digits: ${Arrays.toString(actualDigits)}",
        expectedDigits,
        actualDigits)
}

@Test
fun testSingleDecimalToBinary() {
    val baseConverter = BaseConverter(10, intArrayOf(5))

    val expectedDigits = intArrayOf(1, 0, 1)
    val actualDigits = baseConverter.convertToBase(2)

    assertArrayEquals(
        "Expected digits: ${Arrays.toString(expectedDigits)} but found digits: ${Arrays.toString(actualDigits)}",
        expectedDigits,
        actualDigits)
}
```

<br/><br/><br/>

### &#91;문제 풀이&#93;<br/>
내가 문제의 해답으로 작성한 코드는 다음과 같다.

```kotlin
import kotlin.math.*

class BaseConverter(base: Int, numberArray: IntArray) {
	
	init {
        require(base > 1) { "Bases must be at least 2." }
        require(numberArray.isNotEmpty()) { "You must supply at least one digit." }
        require(numberArray.first() != 0 || numberArray.size == 1) { "Digits may not contain leading zeros." }
        require(numberArray.all { it < base }) { "All digits must be strictly less than the base." }
        require(numberArray.all { it >= 0 }) { "Digits may not be negative." }
    }
	
	//10진법의 수로 변환
	val numberBy10 = numberArray.foldIndexed(0) { index, acc, i -> acc + i * base.toDouble().pow(numberArray.size-1-index).toInt() }	

    fun convertToBase(newBase: Int): IntArray {
		require(newBase > 1) { "Bases must be at least 2." }	//예외처리(2보다 작은 기저는 안됨)
		
        if (numberBy10 == 0)
            return intArrayOf(0)	//10진법으로 변환한 수가 0일 때 배열로 0을 반환함

        var tmpNum = numberBy10
        val newBaseNumber = mutableListOf<Int>()	//결과 리스트(결과의 역순으로 원소가 들어감)
		
        while (tmpNum > 0) {		//나머지정리를 통해 새로운 진법으로 변환함
            newBaseNumber.add(tmpNum % newBase)
            tmpNum /= newBase
        }
		
        return newBaseNumber.reversed().toIntArray()
    }
}
```

코드에 대해 설명을 하면, 클래스명 바로 옆에 생성자를 썼다. 
이 클래스에 대한 객체를 생성할 때, 정수형의 기저의 값과 해당 기저로 표현한 숫자의 정수형 배열을 인자로 전달하기에 "class BaseConverter(base: Int, numberArray: IntArray)"로 구현하였다.

또한 <b>init</b>를 통해 변환을 하기 전에 변환이 가능한 지를 검사하도록 하였다. <br/>
require함수는 인자로 받은 표현식이 참인지를 확인하며, 참이 아닌 경우 IllegalArgumentException 예외를 발생시킨다.<br/>
이 함수를 이용해 기저<sup>base</sup>의 값이 1보다 큰 지를 확인하며 작다면 메시지로 "Bases must be at least 2."를 전달한다.<br/>
마찬가지로 해당 진법으로 표현한 숫자가 없다면 "You must supply at least one digit."를 메시지로 전달해 하나의 숫자를 제공해야 한다고 전달한다.<br/>
또한 진법의 숫자의 첫번째 값이 0이 아니거나 크기가 1일 때를 제외한 경우에는 "All digits must be strictly less than the base."를 출력한다.(첫번째 값이 0이면서 크기가 1일 때인 값 0은 문제가 없기 때문에 이와 같이 조건을 설정한다.)<br/>
숫자에서 각각의 자릿 수의 값이 base인 기저보다 큰 경우에는 말이 되지 않기 때문에 "All digits must be strictly less than the base."를 전달하며, 마찬가지로 값이 0보다 작으면 가능하지 않으므로 "Digits may not be negative." 메시지를 전달한다.

<b>numberBy10</b>에는 numberArray의 숫자를 10진법으로 계산한 정수를 저장하게 한다.
foldIndexed를 통해 acc값을 0으로 먼저 초기화하고 각 배열의 항목에서 해당 항목의 값에 기저의 (숫자배열크기-1-인덱스 )제곱한 값을 곱함으로써 계산할 수 있다. <br/>
(ex. 숫자배열에 2를 기저로 하는 110이 포함되어 있다면 1*2^2+1*2^1+0*2^0이 되어야 하므로 실제 인덱스 값의 반대로 제곱해야 함)

실제로 새로운 기저<sup>newBase</sup>로 계산한 숫자 값을 반환하는 함수인 <b>convertToBase 함수</b>에서는 require문을 통해 먼저 기저의 값이 1보다 큰 지를 확인한다. <br/>
그리고 만약 이전의 기저로 표현한 숫자의 10진수 값이 0이라면 실제로 숫자가 0이므로 0을 포함한 배열을 반환한다. <br/>
tmpNum을 통해 numberBy10를 newBase로 나누면서 나머지정리를 수행한다.<br/>
while문을 살펴보면 tmpNum에 newBase로 나눈 나머지를 결과 배열에 추가한다.
그리고 이어서 나머지정리를 해야하므로 tmpNum의 값에 newBase를 나눈 몫을 다시 tmpNum이라고 한다.
이렇게 tmpNum이 0보다 클 때까지 수행하게 된다. (몫이 0이면 나머지정리가 완료됨)

이렇게 구해진 newBaseNumber의 숫자 배열은 나머지정리의 반대로 값들이 저장되어 있으므로 해당 배열의 값을 반전해야 한다. 따라서 reversed 함수를 사용하고 toIntArray를 통해 정수형 배열로 반환함으로써 해당 기능을 구현할 수 있다.


<br/><br/>
<hr/>
<br/><br/>

지금까지 Exercism의 "All Your Base"문제를 Kotlin으로 풀이하였습니다:)
