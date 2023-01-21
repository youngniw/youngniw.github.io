---
layout: post
title: "[코틀린 쿡북] 13장 코틀린과 구조적 동시성"
categories: Kotlin
tags: [Kotlin, 코틀린]
---

## Chapter 13, 코틀린과 구조적 동시성
코틀린은 개발자가 동시성 코드를 마치 동기 코드처럼 작성할 수 있게 해주는 코루틴을 지원함으로써 동시적 코드 작성이 쉬워지게 된다.<br/><br/>
이번 13장에서는 이러한 코틀린 코루틴과 연관된 주제에 대해 설명한다. 예를 들어, 코루틴 영역과 코루틴 컨텍스트 사용하기, 적절한 코루틴 빌더와 디스패처 선택하기, 코루틴 동작조정 하기 등에 대해 배우게 된다.<br/><br/>
이때 suspend 키워드와 함께 함수를 생성하면 멀티스레딩 코드를 직접 작성하지 않고, 함수를 임시로 정지하고 이후에 다른 스레드에서 이 함수를 다시 재개할 수 있음을 시스템에 알려줄 수 있다.
<br/><br/>

### 13.2 async/await를 withContext로 변경하기
<span style="color:blue"><b>async로 코루틴을 시작하고, 직후에 코루틴이 완료될 동안 기다리는 await코드를 간소화하고 싶은 경우에는 <u>withContext함수</u>를 사용하면 된다.</b></span>

CoroutineScope 클래스에는 withContext라는 확장 함수가 정의되어 있다. 이 함수의 시그니처는 다음과 같다.

```kotlin
suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
): T
```

<u>withContext는 주어진 코루틴 컨텍스트와 함께 명시한 일시정지 블록을 호출하고, 완료될 때까지 일시정지한 후에 그 결과를 리턴한다.</u> 따라서 실제로 async와 바로 다음에 존재하는 await의 조을 대체하기 위해 사용되기도 한다.

다음은 async와 await의 조함을 사용한 코루틴과 withContext를 사용한 코루틴에 대한 예제이다.<br/>
(예제 13-5: async와 await를 withContext로 대체하기)

```kotlin
suspend fun retrieve1(url: String) = coroutineScope {
    async(Dispatchers.IO) {
        println("Retrieving data on $(Thread.currentThread().name)")
        delay(100L)
        "asyncResults"
    }.await()
}

suspend fun retrieve2(url: String) = 
    withContext(Dispatchers.IO) {
        println("Retrieving data on $(Thread.currentThread().name)")
        delay(100L)
        "withContextResults"
    }

fun main() = runBlocking<Unit> {
    val result1 = retrieve1("www.mysite.com")
    val result2 = retrieve2("www.mysite.com")
    println("printing result on ${Thread.currentThread().name} $result1")
    println("printing result on ${Thread.currentThread().name} $result2")
}

/*
실행결과
  Retrieving data on DefaultDispatcher-worker-2
  Retrieving data on DefaultDispatcher-worker-2
  printing result on main withContextResults
  printing result on main asyncResults
-> 순서는 바뀔 수 있음
*/
```

메인 함수는 runBlocking으로 시작하며 두 함수 retrieve1과 retrieve2는 각각 0.1초 동안 지연한 다음 문자열을 반환하는 일을 한다.
두 함수의 차이는 async/await를 사용하거나 withContext를 사용한다는 점이다. 하지만 결론적으로 결과는 유사하게 출력되는 것을 확인할 수 있다.

cf. 실제로 인텔리제이 IDEA가 async/await를 사용한 코드가 있는 것을 인지하면, 이 코드를 withContext로 대체할 것을 제안하고 그 제안을 수락하면 인텔리제이 IDEA가 async/await를 withContext로 변경한다.


<br/><br/>
### 13.3 디스패처 사용하기
<span style="color:blue"><b>I/O 혹은 다른 작업을 위한 전용 스레트 풀을 사용하고자 할 때, <u>Dispatchers 클래스에서 적당한 디스패처를 사용</u>한다.</b></span>

코루틴은 CoroutineContext타입의 컨텍스트 내에서 실행된다. 이 코루틴 컨텍스트에는 CoroutineDispatcher클래스의 인스턴스에 해당하는 코루틴 디스패쳐가 포함되어 있다. 
이 디스패처는 코루틴이 어떤 스레드 혹은 스레드 풀에서 코루틴을 실행할지를 결정한다. <br/>

따라서 launch나 async와 같은 빌더를 사용할 때 CoroutineContext 선택 매개변수를 통해 사용하고 싶은 디스패처를 명시할 수 있다. 

코틀린 표준 라이브러리에는 <b>Dispatchers.Default</b>, <b>Dispatchers.IO</b>, <b>Dispatchers.Unconfined</b>의 내장 디스패처가 포함되어 있다.

<ul>
    <li>기본<sup>Default</sup> 디스패처는 평범한 공유 백그라운드 스레드 풀을 사용한다. 이 기본 디스패처는 보통 코루틴이 대규모의 계산 리소스를 소모하는 경우에 사용된다.</li>
    <li>IO 디스패처는 파일 I/O 혹은 I/O 집약적인 블로킹 작업을 제거하기 위해 디자인된 생성된 스레드 주문식 공유 풀을 사용한다.</li>
    <li>Unconfined 디스패처는 일반적으로 애플리케이션 코드에서 사용해서는 안된다.</li>
</ul>

다음은 기본과 IO 디스패처를 사용한 예시이다.<br/>
(예제 13-6: 기본 디스패처와 I/O 디스패처 사용하기)

```kotlin
fun main() = runBlocking<Unit> {
    launchWithIO()
    launchWithDefault()
}

suspend fun launchWithIO() {
    withContext(Dispatcher.IO) {        //IO 디스패처
        delay(100L)
        println("Using Dispatchers.IO")
        println(Thread.currentThread().name)
    }
}

suspend fun launchWithDefault() {
    withContext(Dispatcher.Default) {   //기본 디스패처
        delay(100L)
        println("Using Dispatchers.Default")
        println(Thread.currentThread().name)
    }
}

/*
실행결과
  Using Dispatchers.IO
  DefaultDispatcher-worker-3
  Using Dispatchers.Default
  DefaultDispatcher-worker-2
-> 워커 번호는 다를 수 있음
*/
```

위의 코드를 보면 launchWithIO와 launchWithDefault 함수 모두 withContext의 CoroutineContext 선택 매개변수를 통해 사용하고자 하는 디스패처(IO, 기본)를 명시하였다.<br/><br/>

<b>cf. 안드로이드 디스패처<br/></b>
: 안드로이드 API는 추가로 Dispatcher.Main 디스패처가 들어있다. 이 디스패처는 보통 UI를 구성하는 스레드를 메인으로 사용하는 플랫폼에서 사용되며, 해당 CoroutineScope을 Main UI Thread에서 동작시키도록 한다. 
이 디스패처를 사용하기 위해서는 kotlinx-coroutines-android 의존성을 추가해야 한다. <br/>
또한 안드로이드 컴포넌트 라이브러리에는 수명주기 디스패처도 추가되어 있다.


<br/><br/>
### 13.4 자바 스레드 풀에서 코루틴 실행하기
<span style="color:blue"><b>코루틴을 사용하는 사용자 정의 스레드 풀을 제공하고 싶을 때, <u>자바 ExecutorService의 asCoroutineDispatcher 함수</u>를 사용한다.</b></span>

코틀린 라이브러리는 java.util.concurrent.ExecutorService에 asCoroutineDispatcher 확장 메소드를 추가했다. 
이 asCoroutineDispatcher함수는 ExecutorService의 인스턴스를 ExecutorCoroutineDispatcher의 구현으로 변환한다.

이러한 asCoroutineDispatcher 함수를 사용하고자 할 때는, Executors 클래스로 사용자 정의 스레드 풀을 정의한 다음 디스패처로 사용할 수 있게 변환해야 한다.

asCoroutineDispatcher 함수를 사용한 예제 코드는 다음과 같다.<br/>
(예제 13-7: 코루틴 디스패처로서 스레드 풀 사용하기)

```kotlin
import kotlinx.coroutines.*
import java.util.concurrent.Executors

fun main() = runBlocking<Unit> {
    val dispatcher = Executors.newFixedThreadPool(10)   //크기가 10인 스레드 풀 생성
        .asCoroutineDispatcher()
    
    //생성한 스레드 풀을 코루틴을 위한 디스패처로 사용
    withContext(dispatcher) {
        delay(100L)
        println(Thread.currentThread().name)
    }
    
    dispatcher.close()      //스레드 풀 종료
}
```

실행 결과는 시스템이 코루틴을 실행하기 위해 스레드 풀 1의 스레드 2를 선택했음을 나타내는 pool-1-thread-2를 출력한다. <br/>
이때 마지막에 디스패처의 close 함수를 호출하는 이유는, ExecutorService는 close 함수를 호출하지 않으면 계속 실행되기 때문인 즉 main 함수가 절대 종료되지 않기 때문이다.

보통 가져온 ExecutorService를 중단시키기 위해 shutdown 혹은 shutdownNow를 호출한다. 
그러므로 ExecutorService의 참조(래퍼런스)를 보전하고 ExecutorService를 수동으로 종료시킬 수 있는데, 이에 대한 예제 코드는 다음과 같다.<br/>
(예제 13-8: 스레드 풀 종료하기)

```kotlin
val pool = ExecutorService.newFixedThreadPool(10)
withContext(pool.asCoroutineDispatcher()) {
    //이전과 같음
}
pool.shutdown()
```
위의 문제점은 개발자가 shutdown 메소드를 호출해야 하는 사실을 잊어버릴 수 있다는 것이다.
자바에서는 위의 문제를 close 메소드가 있는 AutoCloseable 인터페이스를 구현함으로써 해결하고, 코드를 try-with-resources 블록으로 감쌀 수 있다.
하지만 위의 코틀린에서는 shutdown을 호출해야한다는 것이다. <br/>

따라서 코틀린 라이브러리 개발자들은 ExecutorCoroutineDispatcher 클래스의 구현을 변경하였다. 
이들은 ExecutorCoroutineDispatcher가 Closeable 인터페이스를 구현하도록 리팩토링함으로써 새로운 추상 클래스의 이름을 CloseableCoroutineDispatcher라고 설정했으며, close메소드는 다음의 코드와 같다.

```kotlin
import java.util.concurrent.ExecutorService

abstract class ExecutorCoroutineDispatcher: CoroutineDispatcher(), Closable {
    abstract override fun close()
    abstract val executor: Executor
}

//하위 클래스 안에서
override fun close() {
    (executor as? ExecutorService)?.shutdown()
}
```

이를 통해 자바 ExecutorService를 사용해서 생성한 디스패처는 이제 ExecutorService를 종료할 close 함수를 가지고 있게 된다. 
그러면 shutdown까지 호출을 할 수 있게 되는데, 문제는 코틀린은 자바의 try-with-resources구문을 지원하지 않기에 close함수가 호출되는 것을 보장하지 못한다는 것이다.

```kotlin
inline fun <T : Closable?, R> T.use(block: (T) -> R): R
```
이 use함수는 자바 Closable 인터페이스에 확장 함수로 정의되어 있다. 이 use함수를 사용해 자바 ExecutorService를 종료하는 문제를 해결할 수 있게 된다. 이에 대한 코드는 다음과 같다.<br/>
(예제 13-10: use를 사용해서 디스패처 닫기)

```kotlin
Executors.newFixedThreadPool(10).asCoroutineDispatcher().use{
    withContext(it) {
        delay(100L)
        println(Thread.currentThread().name)
    }
}
```

위의 코드는 use 블록문 끝에서 디스패처를 닫게 될 것이며, 이를 통해 기저의 스레드도 닫히게 된다. 따라서 문제를 해결할 수 있다.


<br/><br/>
### 13.5 코루틴 취소하기
<span style="color:blue"><b>코루틴 내의 비동기 처리를 취소하고 싶을 때는 <u>launch 빌더 혹은 withTimeout이나 withTimeoutOrNull 같은 함수가 리턴하는 Job 레퍼런스를 사용</u>한다.</b></span>

13.1에서 공부했다 싶이 launch 빌더는 코루틴을 취소하기 위해 사용할 수 있는 Job 타입의 인스턴스를 반환한다. 
이 Job 타입의 인스턴스를 통해 비동기 처리를 취소할 수 있게 되는데, 이에 대한 예제 코드는 다음과 같다.<br/>
(예제 13-11: 잡 취소하기)

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(100) { i ->
            println("job: I'm waiting $i...")
            delay(100L)
        }
    }
    delay(500L)
    println("main: That's enough waiting")
    job.cancel()
    job.join()
    println("main: Done")
}

/*
실행결과
    job: I'm waiting 0...
    job: I'm waiting 1...
    job: I'm waiting 2...
    job: I'm waiting 3...
    job: I'm waiting 4...
    main: That's enough waiting
    main: Done
*/
```

위의 코드에서 보면, launch 빌터는 Job 인스턴스를 반환하고 이를 지역 변수(job)에 할당한다. 그런 후 repeat 함수를 사용해서 100번의 코드가 반복되게 한다.
launch 블록를 제외한 main 함수에서는 delay를 통해 0.5초 쉬고, println을 통해 출력을 하고, cancel()함수를 통해 해당 잡을 취소한다. <br/>
따라서 이 예제 코드를 실행하게 되면 100번의 반복문이 실행되기도 전에 job.cancel()을 통해 해당 job의 코루틴이 종료되고 기존에 수행하고 있는 메인이 수행됨으로써 "main: Done"을 마지막으로 출력하고 프로그램이 종료된다.

cf. join 함수는 해당 잡이 완료될 때까지 기다린 다음 프로그램을 종료한다.<br/>
cf. cancel과 join 호출을 결합한 cancelAndJoin 함수도 사용가능하다.

이때, 시간이 너무 오래 걸리는 이유로 잡을 취소하고자 할 때는 withTimeout 함수를 사용할 수 있다. 이 함수의 시그니처는 다음과 같다.

```kotlin
suspend fun <T> withTimeout(
    timeMillis: Long,
    block: suspend CoroutineScope.() -> T
): T
```

withTimeOut 함수는 코루틴 안의 일시 중단 블록의 코드<sup>block</sup>를 실행하는데, 만약 주어진 시간<sup>timeMillis</sup>을 초과(타임아웃)하면 TimeoutCancellationException 예외를 던지게 된다. 
이 withTimeOut 함수를 이용한 예제 코드는 다음과 같다.<br/>
(예제 13-12: withTimeout 사용하기)

```kotlin
fun main() = runBlocking {
    withTimeout(1000L) {
        repeat(50) { i ->
            println("job: I'm waiting $i...")
            delay(100L)
        }
    }
}

/*
실행결과
    job: I'm waiting 0...
    job: I'm waiting 1...
    job: I'm waiting 2...
    job: I'm waiting 3...
    job: I'm waiting 4...
    job: I'm waiting 5...
    job: I'm waiting 6...
    job: I'm waiting 7...
    job: I'm waiting 8...
    job: I'm waiting 9...
    Exception in thread "main"
        kotlinx.coroutines.TimeoutCancellationException:
    Timed out waiting for 1000 ms
    at kotlinx.coroutines.TimeoutKt.TimeoutCancellationException(Timeout.kt:126)
*/
```

이러한 TimeoutCancellationException을 캐치하거나 타임아웃일 때 예외를 던지는 대신 null을 반환하는 withTimeoutOrNull을 사용할 수도 있다.<br/><br/>

<b>cf. 안드로이드에서 잡 취소하기<br/></b>
: 안드로이드에서도 마찬가지로 잡을 취소할 수 있는데, 일반적인 구현 패턴은 MainActivity가 CoroutineScope를 구현하고, 컨텍스트가 필요할 때 컨텍스트를 제공하며, 컨텍스트를 닫는다.
이에 대한 예제 코드는 다음과 같다.<br/>
(예제 13-13: 안드로이드에서 디스패처 사용하기)

```kotlin
class MainActivity : AppCompatActivity(), CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main + job      //컨텍스트를 생성
    
    private lateinit var job: Job     //준비되면 속성을 초기화
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        job = Job()         //준비됨
    }

    override fun onDestroy() {
        job.cancel()        //액티비티가 소멸되면 잡을 취소
        super.onDestroy()
    }
}
```

위의 코드에서는 취소를 원하는 경우 해당 잡에 접근할 수 있게 나중 초기화 변수<sup>job</sup>를 사용했다. 이를 통해 다음의 예제에서처럼 필요에 따라 간단하게 코루틴을 시작할 수 있게 된다.<br/>
(예제 13-14: 안드로이드에서 코루틴 시작하기)

```kotlin
fun displayData() {
    launch {    //시작
        val data = async(Dispatchers.IO) {  //네트워크 호출을 가능하게 함
            //네트워크를 통해 데이터 얻음
        }
        updateDisplay(data.await())  //UI갱신을 위해 Dispatcher.Main으로 돌아감
    }
}
```

최신 버전의 안드로이드 아키텍처 컴포넌트는 ViewModel이 지워질 때 자동으로 잡을 취소하는 viewModelScope와 같은 추가 코루틴 영역을 제공한다. 이를 이용해 편리하게 안드로이드에서의 코루틴 작성을 할 수 있게 된다.


<br/><br/>
### 13.6 코루틴 디버깅
<span style="color:blue"><b>코루틴의 실행 정보가 더 많이 필요한 경우에는 <u>JVM에서 -Dkotlinx.coroutines.debug 플래그를 사용해서 실행</u>한다.</b></span>

코루틴 라이브러리는 쉬운 디버깅 기능을 포함한다. 따라서 JVM에서 코루틴을 디버그 모드로 실행하고자 한다면 kotlinx.coroutines.debug 시스템 속성을 사용하면 된다.

디버그 모드는 실행된 모든 코루틴에 고유한 이름을 부여한다. 이러한 생성된 이름도 좋지만 코루틴에 직접 이름을 제공할 수도 있다.
코틀린 라이브러리에는 코루틴에 이름을 부여할 수 있는 CoroutineName 클래스가 존재한다. 이 CoroutineName 생성자를 이용해 스레드 이름으로 사용할 수 있는 컨텍스트 원소를 생성할 수 있다.
이에 대한 예제 코드는 다음과 같다.<br/>
(예제 13-15: 코루틴에 이름 부여하기)

```kotlin
suspend fun retrieve1(url: String) = coroutineScope {
    async(Dispatcher.IO + CoroutineName("async")) {     //+를 통해 코루틴 이름을 추가
        println("Retrieving data on $(Thread.currentThread().name)")
        delay(100L)
        "asyncResults"
    }.await()
}

suspend fun retrieve1(url: String) = 
    withContext(Dispatcher.IO + CoroutineName("withContext")) {
        println("Retrieving data on $(Thread.currentThread().name)")
        delay(100L)
        "withContextResults"
    }

fun main() = runBlocking<Unit> {
    val result1 = retrieve1("www.mysite.com")
    val result2 = retrieve2("www.mysite.com")
    println("printing result on ${Thread.currentThread().name} $result1")
    println("printing result on ${Thread.currentThread().name} $result2")
}

/*
출력결과
    Retrieving data on DefaultDispatcher-worker-1 @withContext#1
    Retrieving data on DefaultDispatcher-worker-1 @async#2
    printing result on main @coroutine#1 withContextResults
    printing result on main @coroutine#1 asyncResults
*/
```


<br/><br/>
<hr/>
<br/><br/>

코틀린 쿡북 책의 13장 정리를 마치도록 하겠습니다.:)
