---
layout: post
toc: true
title: "[코틀린 쿡북] 8장 코틀린 대리자"
categories: Kotlin
tags: 코틀린_스터디
---

## Chapter 8, 코틀린 대리자
이번 8장에서는 표준 라이브러리의 lazy, observable, vetoable, notNull 대리자 및 사용자 정의 대리자 등을 사용하는 방법에 대해 설명한다.<br/>
클래스 대리자를 통해 상속을 합성으로 대체할 수 있으며, 속성 대리자를 통해 어떤 속성의 획득자와 설정자를 다른 클래스에 있는 속성의 획득자와 설정자로 대체할 수 있다.
<br/><br/>

### 8.1 대리자를 사용해서 합성 구현하기
다른 클래스의 인스턴스가 포함된 클래스를 만들고 그 클래스에 연산을 위임하고 싶을 때에는 연산을 위임할 메소드가 포함된 인터페이스를 만들고, 클래스에서 해당 인터페이스를 구현한 다음, by 키워드를 사용해 바깥쪽에 래퍼 클래스를 만들면 된다.

이 by 키워드는 포함된 객체에 있는 모든 public 함수를 이 객체를 담고 있는 컨테이너를 통해 노출할 수 있다.
이때 포함된 객체에 있는 모든 public 함수를 이 객체를 담고 있는 컨테이너를 통해 노출하려면, 포함된 객체의 public 메소드의 인터페이스를 생성해야 한다.

예제를 보면, 스마트폰 안에는 전화와 카메라 등 다양한 부속품이 있다. 이때 스마트폰에서 전화와 카메라의 기능을 사용할 수 있어야 한다. 이를 위해 스마트폰을 래퍼 객체로, 내부의 전화와 카메라 부품을 스마트폰에 포함된 객체로 대입해 생각해보자. <br/>
그리고 Phone과 Camera 클래스는 Dialable과 Snappable 인터페이스를 구현해야 한다. 이에 대한 코드는 다음과 같다.
``` kotlin
interface Dialable {
    fun dial(number: String): String
}

class Phone : Dialable {
    override fun dial(number: String) = 
        "Dialing $number..."
}

interface Snappable {
    fun takePicture(): String
}

class Camera : Snappable {
    override fun takePicture() = 
        "Taking picture..."
}
```

그리고 컨테이너인 SmartPhone클래스가 생성자에서 Phone과 Camera를 인스턴스화함으로써 모든 public함수를 Phone과 Camera 인스턴스에 위임하도록 정의할 수 있는데 이에 대한 코드는 다음과 같다.

```kotlin
class SmartPhone {
    private val phone: Dialable = Phone(),
    private val camera: Snappable = Camera(),
} : Dialable by phone, Snappable by camera          //키워드 by를 사용해서 위임
```

이후에 SmartPhone 클래스를 인스턴스화해서 Phone 또는 Camera의 모든 메소드를 호출할 수 있게 된다.

```kotlin
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.Assertions.*

class SmartPhoneTest {
    private val smartPhone: SmartPhone = SmartPhone()       //인자 없이 SmartPhone을 인스턴스화함
    
    @Test
    fun `Dialog delegates to internal phone`() {
        assertEquals("Dialing 555-1234...", smartPhone.dial("555-1234"))    //위임 함수를 호출
    }
    
    @Test
    fun `Taking picture delegates to internal camera`() {
        assertEquals("Taking Picture...", smartPhone.takePicture())         //위임 함수를 호출
    }
}
```
이때 포함된 객체는 컨테이너을 통해 노출되는 것이 아니라 그 객체의 public 함수만이 노출된다. 
또한 포함된 객체 클래스의 모든 함수가 아닌 해당 인터페이스에 선언되어 있고, 이와 일치하는 함수만 컨테이너에서 사용이 가능하다.


<br/><br/>
### 8.2 lazy 대리자 사용하기
어떤 속성이 필요할 때까지 해당 속성의 초기화를 지연하고 싶은 경우에는 코틀린 표준 라이브러리의 lazy 대리자를 사용하면 된다.

먼저 lazy는 해당 변수에 처음 접근할 때까지 계산하지 않고, 처음 접근하게 되면 초기화를 수행한다. 이 lazy 함수의 시그니처는 다음과 같다.
```kotlin
fun <T> lazy(initializer: () -> T): Lazy<T>         //기본 시그니쳐로, 스스로 동기화 함(lock의 인자로 아무것도 제공되지 않음)

fun <T> lazy(       //lazy 인스턴스가 다수의 스레드 간에 초기화를 동기화하는 방법에 대해 명시함
    mode: LazyThreadSafetyMode, initializer: () -> T
): Lazy<T>

fun <T> lazy(lock: Any?, initializer: () -> T): Lazy<T>     //동기화 락을 위해 제공된 객체를 사용함
```
이와 같이 lazy 대리자를 사용하려면 처음 접근이 일어날 때 값을 제한하기 위해 만들어진 초기화 람다(initializer)를 제공해야 한다.
이때 lazy 대리자의 초기화 람다가 예외를 던지게 될 시에는 다음 번에 접근할 때 해당 값의 초기화를 다시 시도하게 된다.<br/>

또한 lazy는 LazyThreadSafety타입의 인자를 갖을 수 있는데, 이 인자의 값으로는 SYNCHRONIZED, PUBLICATION, NONE으로 설정할 수 있다.
* SYNCHRONIZED: 오직 하나의 스레드만 Lazy 인스턴스를 초기화할 수 있게 락을 사용
* PUBCLICAION: 초기화 함수가 여러 번 호출 될 수 있지만, 첫 번째 리턴값만 사용
* NONE: 락이 사용되지 않음

&#43; lazy 함수는 최상위 함수인 반면, 이후에 설명할 notNull함수, observable함수, vetoable 대리자들은 Delegates 인스턴스의 일부분이다.


<br/><br/>
### 8.3 값이 널이 될 수 없게 만들기
처음 접근이 일어나기 전에 값이 초기화되지 않았을 때, 예외를 던지고 싶은 경우에는 notNull 함수를 이용한다. 

속성에 처음 접근하기 전에 속성이 사용되면 예외를 던지는 대리자를 제공하는 notNull함수를 사용하여 속성 초기화를 지연시킬 수 있다. 이 함수에 대한 예제는 다음과 같다.
```kotlin
var shouldNotBeNull: String by Delegates.notNull<String>{}
```
속성에 값이 제공되기 전에 접근을 시도한다면 다음의 코드와 같이 IllegalStateException 예외를 던지도록 할 수 있다.
```kotlin
@Test
fun `unintialized value throws exception`() {
    assertThrows<IllegalStateException> { shouldNotBeNull }
}

@Test
fun `initialize value then retrieve it`() {
    shouldNotBeNull = "Hello, World!"
    assertDoesNotThrow { shouldNotBeNull }
    assertEquals("Hello, World!", shouldNotBeNull)
}
```
notNull에 대해 더 자세히 알아보기 위해 코틀린 표준 라이브러리의 Delegates.kt 내의 notNull 구현을 보면 다음의 코드와 같다.
```kotlin
object Delegates {      //Delegates는 싱클톤
    fun <T : Any> notNull(): ReadWriteProperty<Any?, T> = NotNullVar()
    //이외의 함수가 있음
}

private class NotNullVar<T : Any>() : ReadWriteProperty<Any?, T> {      //ReadWriteProperty를 구현한 private 클래스
    private var value: T? = null
    
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException(
            "Property ${Property.name} should be initialized before get.")
    }
    
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = value
    }
}
```
notNull팩토리 메소드는 ReadWriteProperty 인터페이스를 구현하는 private 클래스인 NotNullVar를 인스턴스화한다.
또한 NotNullVar의 setValue함수는 그저 제공된 값을 저장하는 반면에, getValue 함수는 값이 널인 지를 확인하고 엘비스 연산자를 사용하여 널이 아닌 경우에는 그 값을 반환하고 널인 경우에는 IllegalStateException을 던진다.


<br/><br/>
### 8.4 observable과 vetoable 대리자 사용하기
속성의 변경을 가로채서 필요에 따라 변경을 거부하고 싶을 때, 변경 감지에는 observable 함수를 사용하고 변경의 적용 여부를 결정할 때는 vetoable 함수와 람다를 사용한다.

observable 함수와 vetoable 함수의 시그니처는 다음과 같다.
```kotlin
fun <T> observable(
    initialValue: T,
    onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit
): ReadWriteProperty<Any?, T>

fun <T> vetoable(
    initialValue: T,
    onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean
): ReadWriteProperty<Any?, T>
```
두 팩토리 함수 모두 T 타입의 초기값과 람다를 인자로 받고, ReadWriteProperty 인터페이스를 구현한 클래스의 인스턴스를 반환한다.
이 함수들은 다음과 같이 사용할 수 있다.
```kotlin
var watched: Int by Delegates.observable(1) {prop, old, new ->
    println("${prop.name} changed from $old to $new")
}

var checked: Int by Delegates.vetoable(0) {prop, old, new ->
    println("Trying to change ${prop.name} from $old to $new")
    new >= 0
}
```
watched 변수는 값이 1로 초기화됐고 이 변수의 값이 변경될 때마다 이전 값과 새로운 값을 보여주는 내용이 출력된다.
반면 checked 변수는 값이 0으로 초기화됐지만 "new>=0"으로 인해 변경 값(new)이 0이거나 양수일 때만 vetoable의 람다 인자가 true를 반환해 변경이 가능하게 설정되었다.

위의 watched와 checked 변수를 테스트하는 코드는 다음과 같다.
```kotlin
@Test
fun `watched variable prints old and new values`() {
    assertEquals(1, watched)
    watched *= 2
    assertEquals(2, watched)
}
//콘솔에 "watched changed from 1 to 2"가 출력된다.

@Test
fun `veto values less than zero`() {
    assertAll(
        { assertEquals(0, checked) },
        { checked = 42;     assertEquals(42, checked) },  //값 변경이 허용됨
        { checked = -1;     assertEquals(42, checked) }   //값 변경이 거부됨
    )
}
```

이 함수들의 구현을 자세히 보면, 싱글톤 Delegates 객체 안에서 observable과 vetoable 함수에 대한 코드는 다음과 같다.
```kotlin
object Delegates {      
    //이외의 함수가 있음
    inline fun <T> observable(initialValue: T, 
        crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit
    ): ReadWriteProperty<Any?, T> = 
        object : ObservableProperty<T>(initialValue) {
            override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = 
                onChange(property, oldValue, newValue)
        }
    
    inline fun <T> vetoable(initialValue: T, 
        crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean
    ): ReadWriteProperty<Any?, T> = 
        object : ObservableProperty<T>(initialValue) {
            override fun beforeChange(property: KProperty<*>, oldValue: T, newValue: T): Boolean = 
                onChange(property, oldValue, newValue)
        }
}
```
observable과 vetoable함수 모두 ObservableProperty타입의 객체를 반환한다. 
이 ObservableProperty를 자세히 알아보기 위해 다음과 같은 ObservableProperty 클래스 코드를 살펴보자.

```kotlin
abstract class ObservableProperty<T>(initialValue: T) : ReadWriteProperty<Any?, T> {
    private var value = initialValue
    
    protected open fun beforeChange(property: KProperty<*>, 
        oldValue: T, newValue: T): Boolean = true
    
    protected open fun afterChange(property: KProperty<*>, 
        oldValue: T, newValue: T): Unit {}
    
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value
    }
    
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        val oldValue = this.value
        if (!beforeChange(property, oldValue, value)) {
            return
        }
        this.value = value
        afterChange(property, oldValue, value)
    }
}
```
ObservableProperty 클래스는 모든 제네릭 타입 T의 속성을 저장하고 ReadWriteProperty 인터페이스를 구현하기에 getValue와 setValue 함수를 제공한다.
먼저 getValue에서는 value의 속성만을 반환한다. 반면에 setValue 함수는 현재 값을 저장한 다음 beforeChange함수를 호출함으로써 반환받은 값이 true라면 value 속성은 변경된다.
<br/><br/>
observable함수는 ObservableProperty를 확장하는 객체를 만들고 제공된 onChange 람다의 작업을 수행하기 위해 afterChange함수를 오버라이드한다.
이때 beforeChange함수는 오버라이드하지 않았기 때문에 true를 반환해 해당 속성의 변경을 무조건 수행한다.<br/>
반면에 vetoable 함수는 beforeChange함수를 오버라이드하게 된다.
이 vetoable의 람다 구현은 인자로 제공되며, 람다는 반드시 해당 속성의 변경 여부를 결정할 수 있는 블리언을 리턴함으로써 true일 경우에는 속성 변경이 수행된다.

<br/>
cf. inline과 crossline<br/>
&#62; inline 키워드는 컴파일러가 함수만 호출하는 완전히 새로운 객체를 생성하는 것이 아니라 <u>해당 호출 위치를 실제 소스 코드로 대체하도록 지시</u>한다.
따라서 inline키워드를 추가한 inline 함수는 컴파일 단계에서 정적으로 코드를 포함시키기에 고차함수를 썼을 때보다 성능 상의 문제를 해결할 수 있다고 한다.

가끔 inline 함수는 다른 컨텍스트에서 실행되어야 하는 매개변수로 전달되는 람다이다. 이렇게 '로컬이 아닌' 제어 흐름은 람다 내에서는 허용되지 않기 때문에 crossinline을 써야 하는 경우가 있다.

&#62;crossline 키워드는 함수를 inline으로 선언했더라도 람다 함수에 return을 사용하지 못하도록 해주는 키워드이다. 
보통 매개변수로 받아온 함수를 다시 다른 객체에 대입해야 할 때 즉, 다른 객체에서 사용하는 매개 함수를 선언할 때 사용한다. 


<br/><br/>
### 8.5 대리자로서 Map 제공하기
코틀린 맵에는 대리자가 되는데 필요한 getValue와 setValue 함수 구현이 있어서, 이를 이용해 여러 값이 들어있는 맵<sup>map</sup>을 제공해 객체를 초기화할 수 있다.

객체 초기화에 필요한 값이 맵 안에 있다면, 해당 클래스 속성을 자동으로 맵에 위임할 수 있다.

```kotlin
data class Project(val map: MutableMap<String, Any?>) {
    val name: String by map         //map 인자에 위임
    var priority: Int by map        //map 인자에 위임
    var completed: Boolean by map   //map 인자에 위임
}

@Test
fun `use map delegate for Project`() {
    val project = Project(
        mutableMapOf(
            "name" to "Learn Kotlin",
            "priority" to 5,
            "completed" to true))
    
    assertAll(
        { assertEquals("Learn Kotlin", project.name) },
        { assertEquals(5, project.priority) },
        { assertTrue(project.completed) }
    )
}
```
이 예제에서 Project클래스 생성자는 MutableMap을 인자로 받고, 해당 맵의 키에 해당하는 값으로 Project 클래스의 모든 속성을 초기화한다.
이 코드가 동작하는 이유는 MutableMap에 ReadWriteProperty 대리자가 되는데 필요한 올바른 시그니처의 setValue와 getValue 확장 함수가 있기 때문이다.

<u>속성을 생성자의 일부로 만들지 않고 맵을 사용하는 이유는 무엇일까?</u><br/>
JSON을 파싱하거나 다른 동적인 작업을 하는 애플리케이션에서 이러한 매커니즘이 발생하기 때문이다. 
이에 대한 예제 코드는 다음과 같다.
```kotlin
private fun getMapFromJSON() =          //JSON 문자열로 속성 맵을 가공
    GSON().fromJSON<MutableMap<String, Any?>>(
    """{ "name":"Learn Kotlin", "priority":5, "completed":true }""",
    MutableMap::class.java)

@Test
fun `create project from map parsed from JSON string`() {
    val project = Project(getMapFromJSON()) //Project 인스턴스화를 위해 맵을 사용
    
    assertAll(
        { assertEquals("Learn Kotlin", project.name) },
        { assertEquals(5, project.priority) },
        { assertTrue(project.completed) }
    )
}
```


<br/><br/>
### 8.6 사용자 정의 대리자 만들기
ReadOnlyProperty 또는 ReadWriteProperty를 구현하는 클래스를 생성함으로써 직접 속성 대리자를 작성해, 어떤 클래스의 속성이 다른 클래스의 획득자와 설정자를 사용하게끔 만들 수 있다.

클래스의 속성은 대부분 지원 필드와 함께 동작하지만, 대신 값을 획득하거나 설정하는 동작을 다른 객체에 위임할 수 있다. 
이때 사용자 정의 속성 대리자를 생성하려면 ReadOnlyProperty 또는 ReadWriteProperty 인터페이스에 존재하는 함수를 제공해야 한다.

다음의 코드는 ReadOnlyProperty와 ReadWriteProperty 인터페이스의 시그니처이다.
```kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```
대리자를 만들려고 ReadOnlyProperty나 ReadWriteProperty 인터페이스를 구현할 필요 없이, 위의 시그니처와 동일한 getValue와 setValue함수만으로도 가능하다.

코틀린 표준 문서의 속성 대리자 절에는 Delegate 클래스가 있는 간단한 대리자에 대한 예시가 있는데, 코드는 다음과 같다.
```kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```
위의 대리자를 사용하려면 위임할 클래스 또는 변수를 생성한 다음 해당 변수를 가져오거나 설정해야 한다. 
```kotlin
class Example {
    var p: String by Delegate()
}

fun main() {
    val e = Example()
    println(e.p)
    e.p = "NEW"
}
//출력
//delegates.Example@4c98385c, thank you for delegating 'p' to me!
//NEW has been assigned to 'p' in delegates.Example@4c98385c.
```


마지막으로 그레이들 빌드 도구는 위임된 속성을 통해 컨테이너와 상호작용을 할 수 있게 도와주는 코틀린 DSL을 제공한다. 
그레이들에는 2개의 주요 속성 영역이 있는데, 그 중 하나는 프로젝트 자체와 연관된 속성의 집합이고, 다른 하나는 프로젝트 전체에서 사용할 수 잇는 extra 속성이 있다. 
이 2가지 타입의 속성을 by 키워드를 사용하여 만들고 접근할 수 있다. 


<br/><br/>
<hr/>
<br/><br/>

지금까지 코틀린 쿡북 책의 8장 정리를 마치도록 하겠습니다.:)
