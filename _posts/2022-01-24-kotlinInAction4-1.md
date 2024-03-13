---
layout: post
title: "📅 Kotlin in Action - 4장 클래스, 객체, 인터페이스"
excerpt: "Kotlin in Action 4장 요약 노트입니다."
subtitle: "Kotlin in Action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-1-24
tags: [Kotlin]
---

### 4.1 클래스 계층 정의 

#### 4.1.1 코틀린 인터페이스

- 코틀린 인터페이스 안에는 추상 메서드 뿐만 아니라 구현이 있는 메서드도 정의 가능 
- override 변경자는 실수로 상위 클래스의 메서드를 오버라이드 하는 것을 방지
- 상위 타입의 이름을 꺽쇠 괄호 사이에 넣어서 super를 지정하면, 어떤 상위 타입의 멤버 메서드를 호출할지 지정 가능

```kotlin
fun main(args: Array<String>) {
	val button = Button()
    button.showOff() //  I'm clickable! I'm Focusable!
    button.setFocus(true) // I got focus.
    button.click() // I was clicked!
}

interface Clickable {
	fun click() // 일반 메서드 선언
	fun showOff() = println("I'm clickable!") // 디폴트 구현이 있는 메서드
}

interface Focusable {
	fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
	fun showOff() = println("I'm Focusable!")
}

class Button: Clickable, Focusable {
	override fun click() = println("I was clicked!")
	override fun showOff(){ 
		super<Clickable>.showOff()
		super<Focusable>.showOff() // 
	}
}
```

#### 4.1.2 open, final, abstract 변경자: 기본적으로 final

- 하위 클래스에서 오버라이드하게 의도된 클래스와 메서드가 아니라면 모두 final class로 만들 것 
- 클래스 상속 및 오버라이드 허용을 원하는 메서드나 프로퍼티 앞에는 open 변경자를 붙일 것

```kotlin
open class RichButton: Clickable { // 다른 클래스가 상속 가능하도록 열려있음
  	fun disable() {} // 하위 클래스에서 오버라이드 불가 
    open fun animate() {} // 하위 클래스에서 오버라이드 가능 
    override fun click() {} // 상위 클래스 메서드를 오버라이드
}
```
> **스마트 캐스트 기능을 최대한 이용하기 위해 클래스의 기본적인 상속 가능 상태를 final로 지정**

- 추상 클래스에는 구현이 없는 추상 멤버가 있어 하위 클래스에서 오버라이드 필요 
```kotlin
abstract class Animated { // 추상클래스, 인스턴스를 만들 수 없음
    abstract fun animate() // 구현이 없는 추상 함수. 하위 클래스에서 반드시 오버라이드 필요
    open fun stopAnimating(){} // open으로 오버라이드 허용
    fun animateTwice(){}
}
```
- kotlin 내 상속 제어 변경자

| 변경자        | 오버라이드 가능 유무             |               설명                |
|------------|:------------------------|:-------------------------------:|
| `final`    | 오버라이드 할 수 없음            |         클래스 멤버의 기본 변경자          |
| `open`     | 오버라이드 할 수 있음            |     반드시 open을 명시해야 오버라이드 가능     | 
| `abstract` | 반드시 오버라이드               | 추상 클래스의 멤버에만 사용 가능. 추상 멤버에는 구현X | 
| `override` | 상위 클래스나 인스턴스의 멤버를 오버라이드 |     오버라이드하는 멤버는 기본적으로 열려있음      | 

#### 4.1.3 가시성 변경자: 기본적으로 공개

- Visibility modifier는 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어 -> 해당 클래스에 의존하는 외부 코드를 꺠지 않고 
클래스 내부구현 변경가능 -> encapsulation(캡슐화) 의의

- internal: 코틀린에 도입된 새로운 가시성 변경자. 모듈 내부에서만 볼 수 있으며, 모듈이란 한번에 한꺼번에 컴파일되는 코틀린
파일을 의미 

- 최상위 선언에 대해 private 가시성을 허용하여, 그 선언이 들어있는 파일 내부에서만 사용가능

- kotlin 가시성 변경자

| 변경자               | 클래스 멤버             |             최상위 선언              |
|-------------------|:-------------------|:-------------------------------:|
| `public(default)` | 모든 곳에서 볼 수 있음      |         클래스 멤버의 기본 변경자          |
| `internal`        | 같은 모듈 안에서만 볼 수 있음  |     반드시 open을 명시해야 오버라이드 가능     | 
| `protected`       | 하위 클래스 안에서만 볼 수 있음 | 추상 클래스의 멤버에만 사용 가능. 추상 멤버에는 구현X | 
| `private`         | 같은 클래스 안에서만 볼 수 있음 |     오버라이드하는 멤버는 기본적으로 열려있음      |


```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Sasha!")
    protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech() { //public 함수인 giveSpeech 안에서 internal 수신 타입인 TalkativeButton을 참조X
    yell() // yell은 TalkativeButton의 private 멤버로 접근 불가
    whisper() // whisper는 TalkativeButton의 protected 멤버로 접근 불가
}
```
> 컴파일 오류를 해결하려면 확장 함수 giveSpeech 가시성을 internal로 바꾸거나, TalkativeButton 클래스의 가시성을 public으로 변경

- kotlin에서 protected 멤버는 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보임
- kotlin에서 외부 클래스가 내부 클래스나 중첩된 클래스의 private 멤버에 접근할 수 없음

#### 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

- 클래스 안에 다른 클래스 선언이 가능하며 도우미 클래스를 캡슐화하거나 코드 정의를 사용하는 곳 근처에 두고 싶을 때 유용

```kotlin
interface State: Serializable
interface View {
    fun getCurrentState(): State
    fun restoreState(state: State) {}
}

// 중첩 클래스를 사용해 코틀린에서 view 구현
class Button : View {
    override fun getCurrentState(): State = ButtonState()
    override fun restoreState(state: State) {super.restoreState(state)}
    class ButtonState: State {}
}
```

- Inner 클래스 안에서 Outer 클래스의 참조에 접근하는 방법은 다음과 같다

```kotlin
class Outer {
    inner class inner {
        fun getOuterReference() : Outer = this@Outer
    }
}
```

#### 4.1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

- 봉인된 클래스는 클래스의 외부에 자신을 상속한 클래스를 둘 수 없음
- when 식에서 sealed 클래스의 모든 하위 클래스를 처리한다면 else 분기 필요 X

```kotlin
sealed class Expr { // 기반 클래스를 sealed로 봉인
    class Num(val value: Int) : Expr() // 모든 하위 클래스를 중첩 클래스로 나열
    class Sum(val left: Expr, val right: Expr) : Expr
}
    
fun eval(e: Expr): Int = 
    when (e) { // 
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
    }
```

### 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

#### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

- primary constructor(주 생성자): 생성자 파라미터를 지정하고 프로퍼티를 정의하는 목적을 갖고 있음 
- 다음은 User 클래스를 정의하는 세가지 방법임
```kotlin
class User constructor(_nickname: String) { // 파라미터가 하나만 있는 주 생성자
    val nickname: String
    init {
        nickname = _nickname // 초기화 블록
    }
}
```
```kotlin
class User(_nickname: String) { 
    val nickname = _nickname // 주 생성자의 파라미터로 초기화된 프로퍼티
}
```
```kotlin
class User(val nickname: String) // 가장 간결하게 프로퍼티 생성 
```

#### 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

- 자바 상호운용성과 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우에 다수의 부생성자 이용

```kotlin
class MyButton : View {
	constructor(ctx: Context) : super(ctx){} // super() 키워드를 통해 상위 클래스 생성자를 호출
	constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr){}
}
```

#### 4.2.3 인터페이스에 선언된 프로퍼티 구현

- SubscribingUser는 커스텀 게터로 nickname 프로퍼티를 설정, FacebookUser 클래스는 초기화 식으로 nickname 값을 초기화
> **SubscribingUser의 nickname은 매번 호출될 때마다 substringBefore를 호출해 계산하는 커스텀 게터 이용, FacebookUser는
객체 초기화 시 계산한 데이터를 뒷받침하는 필드에 저장했다가 불러오는 방식**

```kotlin
interface User {
    val nickname: String // 추상 프로퍼티 선언
}

class PrivateUser (override val nickname: String) : User
class SubscribingUser (val email: String): User {
	override val nickname: String
	get() = email.substringBefore('@') // 커스텀 게터
}
class FacebookUser(val accountId: Int): User {
	override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식, getFacebookName 함수는 다른 곳에 선언
}

println(PrivateUser("88parksw@gmail.com").nickname)
```

- 인터페이스에는 추상 프로퍼티뿐 아니라 게터와 세터가 있는 프로퍼티를 선언할 수 있음
```kotlin
interface User {
	val email: String
	val nickname: String
	  get() = email.substringBefore('@') // 프로퍼티에 뒷받침하는 필드가 없으나 매번 결과를 계산해 돌려줌
}
```
