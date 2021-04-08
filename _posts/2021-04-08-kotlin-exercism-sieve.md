---
layout: post
title: "[Kotlin] Exercism Sieve 문제 설명/풀이"
categories: Kotlin
tags: 코틀린_Exercism
---

## Exercism 문제: Sieve
이번 주에는 Exercism의 Sieve 문제를 준비해보았다.
<br/><br/><br/>

### &#91;문제 설명&#93;<br/>
에라토스테네스의 체(Sieve of Eratosthenes)를 사용하여 2에서 주어진 숫자까지의 모든 소수를 찾도록 구현해야하는 문제이다.

에라토스네스의 체는 주어진 한도까지 모든 소수를 찾기 위한 단순한 고대 알고리즘이다.
나눗셈 또는 나머지 연산을 사용하지 않고, 2의 배수부터 시작해 각 소수의 배수를 합성수로 반복적으로 표시함으로써 결론적으로 표시되지 않은 수를 소수로 인식한다.

2부터 시작해서 지정된 한계까지 계속해서 범위를 만들면 된다.

이때 알고리즘은 다음과 같은 반복으로 구성이 된다.
1. 목록에서 사용할 수 있는 다음 미표시된 번호(소수)를 선택한다.
2. 그 숫자의 배수들은 소수가 아니므로 목록에 표시한다.
이러한 반복은 범위의 숫자를 처리할 때까지 이루어지게 된다.

이후 알고리즘이 종료되면 목록에 표시되지 않은 모든 숫자가 소수가 되게 된다.


<br/>
주어진 구현할 코드는 다음과 같다.

```kotlin
object Sieve {

    fun primesUpTo(upperBound: Int): List<Int> {
        TODO("Implement this function to complete the task")
    }
}
```

위의 구현될 코드를 테스트하는 코드는 다음과 같다.
```kotlin
import org.junit.Test
import org.junit.Ignore
import kotlin.test.assertEquals

class SieveTest {

    @Test
    fun noPrimesUnder2() {
        val expectedOutput = emptyList<Int>()

        assertEquals(expectedOutput, Sieve.primesUpTo(1))
    }

    @Test
    fun findFirstPrime() {
        val expectedOutput = listOf(2)

        assertEquals(expectedOutput, Sieve.primesUpTo(2))
    }

    @Test
    fun findPrimesUpTo10() {
        val expectedOutput = listOf(2, 3, 5, 7)

        assertEquals(expectedOutput, Sieve.primesUpTo(10))
    }
    
    @Test
    fun findPrimesUpTo1000() {
        val expectedOutput = listOf(2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61,
                67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173,
                179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 269, 271, 277, 281, 283,
                293, 307, 311, 313, 317, 331, 337, 347, 349, 353, 359, 367, 373, 379, 383, 389, 397, 401, 409, 419, 421,
                431, 433, 439, 443, 449, 457, 461, 463, 467, 479, 487, 491, 499, 503, 509, 521, 523, 541, 547, 557, 563,
                569, 571, 577, 587, 593, 599, 601, 607, 613, 617, 619, 631, 641, 643, 647, 653, 659, 661, 673, 677, 683,
                691, 701, 709, 719, 727, 733, 739, 743, 751, 757, 761, 769, 773, 787, 797, 809, 811, 821, 823, 827, 829,
                839, 853, 857, 859, 863, 877, 881, 883, 887, 907, 911, 919, 929, 937, 941, 947, 953, 967, 971, 977, 983,
                991, 997)

        assertEquals(expectedOutput, Sieve.primesUpTo(1000))
    }
}

```

<br/><br/><br/>

### &#91;문제 풀이&#93;<br/>
내가 문제의 해답으로 작성한 코드는 다음과 같다.

```kotlin
object Sieve {

    fun primesUpTo(upperBound: Int): List<Int> {
        var isprimeArray = Array(upperBound+1, {true})
		var primeList = mutableListOf<Int>()
		
		if (upperBound<2)
			return emptyList<Int>()
		else {
			isprimeArray[0] = false
			isprimeArray[1] = false
			for (i in 2..upperBound) {
				if (isprimeArray[i] == true){		//소수일 때만 반복
					primeList.add(i)	//소수를 저장하는 list에 저장함
					
					//i의 배수들을 소수가 아니라고 설정함
					var j = 2
					while (i*j <= upperBound) {
						isprimeArray[i*j] = false
						j++
					}
				}
			}
			return primeList
		}
    }
}
```

코드에 대해 설명을 하면, 먼저 주어진 한계까지를 index로 갖으며 초기에는 true로 값들이 초기화되는 isprimeArray를 생성함으로써 이 배열에서는 해당 인덱스 숫자가 소수인지 아닌지를 Boolean타입 값으로 저장하도록 한다.
또한 primeList라는 mutableList타입의 리스트를 생성함으로써 추후에 isprimeArray를 참고하여 소수인 숫자만을 원소로 갖게 한다.

이렇게 배열과 리스트를 생성한 후에 만약 인자로 넘겨받은 한계값이 2보다 작을 때에는 소수가 없으므로 emptyList타입의 리스트를 반환하게 한다.

한계값이 2보다 크거나 같을 경우에는 먼저 배열에서 안쓰이지만 인덱스 0과 1도 포함이 되어있기 때문에 해당 인덱스의 값은 소수가 아니라고 false를 저장하게 한다. 
그리고 for문을 통해 주어진 한계까지의 값을 모두 검사하도록 하는데, 
if문을 통해 i를 인덱스로 하는 배열의 원소의 값이 true이면 소수인 것이므로 소수만을 저장하는 primeList에 저장을 하고,
해당 i의 배수들을 소수가 아니라고 설정한다. 
이 알고리즘은 while문을 통해 upperBound까지의 i의 배수들을 인덱스 값으로 하는 원소의 값을 false로 설정하는 것이다.
이렇게 되면 소수가 아닌 값들이 배제됨으로써 for문이 완벽하게 작동이 된다.

이후에 소수의 값만을 갖는 리스트 primeList를 반환함으로써 함수가 올바르게 수행된다.

<br/><br/>
<hr/>
<br/><br/>

지금까지 Exercism의 Sieve문제를 Kotlin으로 풀이하였습니다:)
