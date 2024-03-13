---
layout: post
title: "📅 Kotlin in Action - 4장 클래스, 객체, 인터페이스"
excerpt: "Kotlin in Action 4장 요약 노트입니다."
subtitle: "Kotlin in Action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-2-3
tags: [Kotlin]
---

#### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근 

- 특정 값을 저장하되 그 값을 변경하거나 읽을 때마다 정해진 로직을 실행하는 유형의 프로퍼티를 생성

```kotlin
fun main() {
    val user = User("Alitciia")
    user.address = "안양시 동안구 110-5" 
    println(user) 
    println(user.address)
}

class User(val name: String) {
    var address: String = "unspecified"
    	set(value: String) {
            println("""
            	Address was changed for $name:
   				"$field" -> "$value".""".trimIndent())
            field = value
        }
}
```

#### 4.2.5 접근자의 가시성 변경

- 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같으나 get, set 앞에 가시성 변경자를 추가해서 접근자 가시성 변경 가능

```kotlin
fun main() {
	val lengthCounter = LengthCounter()
	lengthCounter.addWord("Hi!")
	println(lengthCounter.counter) // 3
}

class LengthCounter {
	var counter: Int = 0
		private set // 외부 코드에서 단어 길이릐 합을 마음대로 바꾸지 못하게 하기 위해서 내부에서만 변경 가능케 함
	fun addWord(word: String) {
		counter += word.length
	}
}
```

### 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임

- 클래스에 기본적으로 들어있는 메서드들을 오버라이드하여 커스터마이징 가능

#### 4.3.1 모든 클래스가 정의해야 하는 메서드 

- 문자열 표현: toString()

```kotlin
fun main() {
	val client1 = Client("사샤", 4122)
    println(client1) // Client(name = 사샤, postalCode=4122)
}

class Client (val name: String, val postalCode: Int) {
	override fun toString() = "Client(name = $name, postalCode=$postalCode)"   
}
```

- 객체의 동등성: equals(other: Any)
> 
> "=="는 원시 타입일 경우 피연산자의 값이 같은지를 비교, 참조 타입인 경우 피연산자의 주소가 같은지를 비교. 코틀린에서 
> "=="는 내부적으로 equals를 호출해서 객체 비교

```kotlin
fun main() {
	val client1 = Client("오현석", 4122)
    val client2 = Client("오현석", 4122)
    println(client1 == client2) // false
}

class Client (val name: String, val postalCode: Int) {
	override fun toString() = "Client(name = $name, postalCode=$postalCode)"   
}
```

```kotlin
class Client(val name: String, val postalCode: Int) {
	override fun equals(other: Any?) : Boolean {
		if (other == null || other !is Client)
			return false
		return name == other.name && postalCode == other.postalCode
	}
	override fun toString() = "Client(name = $name, postalCode=$postalCode)"
}
```

- 해시 컨테이너: hashCode()
> JVM 언어에서는 equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야 함. 두 객체의 id가 같은지를 검사하는 프로세스

```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}

```

#### 4.3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성 

- data라는 변경자를 클래스 앞에 붙이면 필요한 메서드를 컴파일러가 자동으로 만들어줌. 그리고 이러한 클래스를 데이터 클래스라고 함
- 하기 클래스는 다음과 같은 메서드를 포함. 인스턴스 비교를 위한 equals, 해시 기반 컨테이너에서 키로 사용할 수 있는 hashCode, 클래스의 각 필드를
선언 순서대로 표시하는 문자열 표현을 만들어 주는 toString 

```kotlin
data class Client(val name: String, val postalCode: Int) 
```

- 데이터 클래스와 불변성: copy() 메서드: HashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성이 필수. 클래스 인스턴스를 직접
바꾸는 것 보다는 차라리 복사하여 일부 프로퍼티를 바꾸게 하는 것이 다루기 더 쉬우며 이러한 기능을 제공

```kotlin
fun main() {
      val lee = Client("사샤", 4122)
      println(lee.copy(postalCode = 400))
      Client(name = 사샤, postalCode = 4000)
}
```

> 예전 JPA 엔티티를 data class로 작성했지만, 상속이 까다로운 문제로 인해 일반 class로 변경 
> 

### 4.3.3 클래스 위임: By 키워드 사용

- 대규모 객체지향 시스템을 설계할 때 시스템을 취약하기 만드는 문제는 보통 구현 상속(implementation inheritance)에서 발생 -> 코틀린은 기본적으로 클래스를 final로 취급
- @(Decorator 패턴): 상속을 허용하지 않는 클래스에 새로운 동작을 추가할 때 쓰이는 방법. 하지만 구현 코드량이 많고 복잡함
- 그래서 by 키워드를 이용하여 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 것을 표시

```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
): MutableCollection<T> by innerSet { // MutableCollection의 구현을 innerSet에게 위임
    var objectAdded = 0 
    override fun add(element: T): Boolean { // 위임하지 않고 새로운 구현 제공
        objectAdded++	
        return innerSet.add(element)
    }
    override fun addAll(c: Collection<T>): Boolean { // 위임하지 않고 새로운 구현 제공
        objectAdded += c.size
        return innerSet.addAll(c)
    }	  
}
```

### 4.4 object 키워드: 클래스 선언과 인스턴스 생성

- 클래스를 정의하면서 동시에 인스턴스(객체)를 생성
  - object declaration(객체 선언): singleton을 정의하는 방법 중 하나
- companion object (동반 객체): 인스턴스 메서드는 아니지만 특정 클래스와 관련 있는 메서드와 팩토리 메서드를 담을 때 쓰임
- 객체 식은 자바의 anonymous inner class (무명 내부 클래스) 대신 쓰임

#### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기 

- 코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원. 즉, 객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을
합친 선언
- 객체 선언 안에 프로퍼티, 메서드, 초기화 블록 등을 넣을 수 있으나 주/부 생성자는 객체 선언에 쓸 수 없음

```kotlin
object CaseInsensitiveFileComparator: Comparator<File> { 
    override fun compare(file1: File, file2: File): Int {
			return file1.path.compareTo(file2.path, ignoreCase = true)
	}
}
```
> **Singleton과 의존관계 주입**: 대규모 소프트웨어 시스템에서는 객체 선언이 항상 적합하지는 않음. 특히, 생성을 제어할 수 없고 생성자
파라미터를 지정할 수 없으므로 단위 테스트를 하거나 소프트웨어 시스템의 설정이 달라질 때 객체를 대체하거나 객체의 의존관계를 바꿀 수 없음.

- 중첩 객체를 사용해 Comparator 구현하기

```kotlin
fun main() {
    val persons = listOf(Person("Bob"), Person("Alice"))
    println(persons.sortedWith(Person.NameComparator)) // [Person(name=Alice), Person(name=Bob)]
}

data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int = p1.name.compareTo(p2.name)
    }
}
```

#### 4.4.2 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소 

- 코틀린은 자바의 static 키워드를 지원하지 않음. 대신 패키지 수준의 최상위 함수와 객체 선언 활용
- 하지만 최상위 함수는 private으로 표시된 클래스 비공개 멤버에 접근할 수 없어서, 팩토리 메서드를 이용
- companion 키워드를 붙여 동반 객체로 지정하면 자바의 정적 메서드 호출 / 필드 사용 구문과 같아짐 

```kotlin
fun main() {
    val subscribingUser = User.newSubscribingUser("sasha@2bytescorp.com")
    val facebookUser = User.newFacebookUser(4) // 클래스 이름을 사용해 해당 클래스에 속한 동반 객체의 메서드 호출 가능
    println(subscribingUser.nickname)
}

class User private constructor(val nickname: String) { // 주 생성자를 비공개로 생성
    companion object { // 동반 객체 선언 
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId)) // facebook 사용자 id로 사용자를 만드는 팩토리 메서드 
    }
}
```

#### 4.4.3 동반 객체를 일반 객체처럼 사용

- 클래스 안에 정의되는 동반 객체에 이름을 붙이거나, 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티 정의 가능
- 동반객체: 클래스 안에 정의된 일반 객체

```kotlin
fun main() {
	person = Person.Loader.fromJSON("{name: 'Dmitry'}")
    println(person.name) // Dmitry
}

class Person(val name: String) {
	companion object Loader { // 동반 객체에 이름을 붙임 
        fun fromJSON(jsonText: String): Person = {}
    }
}
```
- 동반 객체에서 인터페이스 구현 가능

```kotlin
interface JSONFactory<T> {
	fun fromJSON(jsonText: String): T
}

class People(val name: String) {
	companion object: JSONFactory<Person> {
        override fun fromJSON(jsontext:String): Person
    }
}
```

- 동반 객체 확장: 클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수 생성 가능

```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object{} // 비어있는 동반 객체를 선언
}

fun Person.Companion.fromJSON(json: String): Person {}

val p = Person.fromJSON(json) // 확장 함수 호출
```

#### 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성

- 무명 객체는 object 키워드를 사용하여 생성하며, 자바의 무명 내부 클래스를 대신함
- 코틀린 객체 식은 다수의 인터페이스 구현, scope 내 변수 변경할 수 있는 등 자바보다 더 많은 기능 제공
- 객체 식은 무명 객체 안에서 여러 메서드를 오버라이드해야 하는 경우에 훨씬 유용
> 객체 선언과 달리 무명 객체는 singleton이 아니며 객체 식이 쓰일때 마다 새로운 인스턴스가 생성

```kotlin
// 무명 객체로 이벤트 리스너 구현하기
window.addMouseListener {
    object: MouseAdapter() { // MouseAdapter를 확장하는 무명 객체 선언
        override fun mouseClicked(e: MouseEvent) {} // MouseAdapter의 메서드를 오버라이드
        override fun mouseEntered(e: MouseEvent) {} // MouseAdapter의 메서드를 오버라이드
    }   
}

// 무명 객체 안에서 로컬 변수 사용
fun countClicks(window: Window) {
  var clickCount = 0
  window.addMouseListener(object : MouseAdapter() { 
      override fun mouseClicked(e: MouseEvent) {
          clickCount ++
      }
  })
}
```