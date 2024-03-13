---
layout: post
title: "📅 Kotlin 강의노트 - 15강까지"
excerpt: "디모의 Kotlin 강좌 요약본입니다 "
subtitle: "Kotlin youtube lecture"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-1-11
tags: [Kotlin]
---

#### 클래스 생성자

```kotlin
fun main() {
    var a = Person("박보영", 1990)
    var b = Person("전정국", 1997)
    var c = Person("장원영", 2004)
    var d = Person("으아아")
    var e = Person("으아아")
    var f = Person("으아아")
}

class Person (var name: String, val birthYear:Int) {
    init {
        println("${this.birthYear}년생 ${this.name}님이 생성되었습니다!")
    }
    constructor(name: String): this(name,1997) { // 생성자
        println("보조 생성자가 사용되었습니다!")
    }
}
```

#### 클래스의 상속
 - 상속이 어떨 때 필요? 기존 클래스를 확장하여 새로운 속성이나 함수를 추가한 클래스를 만들고 싶을 때, 여러 개의 클래스에서 공통되는 클래스를 뽑아 관리하기 용이하게 하고 싶을 때
 - 상속에 대한 두가지 규칙. 첫번째 서브 클래스는 수퍼 클래스에 존재하는 속성과 같은 이름의 속성을 가질 수 없음
 - 서브 클래스가 생성될 때에는 수퍼 클래스의 생성자까지 호출
 - 지나친 상속 구조는 코드를 더 어렵게 만드는 단점

```kotlin
fun main() {
    var a = Animal("별이", 5, "개") // 인스턴스를 생성
    var b = Dog("별이", 5)
    
    a.introduce()
    b.introduce()
    b.bark()
    
    var c = Cat("루이", 1) // 인스턴스를 생성
    c.introduce()
    c.meow()
}

open class Animal (var name:String, var age:Int, var type:String) { // open: 클래스 상속 가능 
    fun introduce() {  // 메서드 선언
        println("저는 ${type} ${name}이고, ${age}살 입니다.")
    }
}

class Dog (name:String, age:Int): Animal(name, age, "개") { // ** 클래스 상속
    fun bark() { // 수퍼 클래스와 구분되는 메서드
        println("bowbow!")
    }
}

class Cat (name:String, age:Int): Animal(name, age, "고양이") {
    fun meow() {
        println("meowmeow~")
    }
}
```

#### 오버라이딩
```kotlin
fun main () {
    var t = Tiger()
    t.eat()
}

open class Animal  {
    open fun eat() { // 상속허용
    println("소고기를 먹습니다")
    }
}
    
// super class에서 끝난 함수를 오버라이딩을 통해 재구현
class Tiger : Animal() {
    override fun eat() {
    println("고기를 먹습니다")
    }
}
```

#### 추상화

- 추상화: 선언부만 있고 기능이 구현되지 않은 추상함수, 추상클래스 두 요소로 구성
- 인터페이스: 속성, 추상함수, 일반함수를 포함. 추상함수는 생성자를 가질 수 있으나 인터페이스는 생성자를 가질 수 없음.
- 인터페이스에서 구현부가 있는 함수는 open 함수, 없는 함수는 abstract 함수로 기본적으로 간주. 별도의 두 키워드가 없어도 서브클래스에서 구현 및 재 정의 가능
- 한번의 여러 인터페이스를 상속받아 서브클래스 생성 가능

```kotlin
fun main () {
    var r = Rabbit()
    r.eat()
    r.sniff()
    println("$r")
}

abstract class Animal { // 추상클래스
    abstract fun eat()
    fun sniff() {
    println("냄시")
    }
}

class Rabbit : Animal() {
    override fun eat() {
    println("당근을 먹습니다")
    }
}
```

- 오버라이딩은 이미 구현이 끝난 함수를 서브 클래스에서 변경할 때
- 추상화는 선언만하고 실제 구현은 서브 클래스에 일임할 때 
- 인터페이스는 서로 다른 기능들을 여러 개 상속해줘야 할 때 필요 

```kotlin
fun main() {
    var d = Dog()
    d.run()
    d.eat()
}

interface Runner {
    fun run()
}

interface Eater {
    fun eat() {
        println("음식을 먹습니다")
    }
}

class Dog: Runner, Eater {
    override fun run() {
        println("우다다다다다다다다")
    }
    override fun eat() {
        println("우다다다다다 뛰니다아아아앙")
    }
}
```

#### 패키지 이용법
- 패키지: 개발 시에 소스코드의 소속을 지정하기 위한 논리적 단위  
- 패키지 작명: com.youtube.sasha.base 

- 코드 파일을 패키지에 넣는 방법
- 패키지를 명시하지 않으면 자동으로 default 패키지로 묶임

```kotlin
package com.youtube.dimo
import com.youtube.dimo.base 
// 다른 패키지의 변수나 함수, 클래스 등을 그대로 사용 가능
// 이름이 중복될 경우, 풀 패키지 명을 언급
```

- 클래스명과 파일명이 일치하지 않아도 되며, 하나의 파일에 여러 개의 클래스를 넣어도 알아서 컴파일이 가능

#### 변수, 함수, 클래스의 접근 범위와 접근 제한자

 - 접근제한자 (package scope)  
 public: 어떤 패키지에서도 접근 가능  
 internal: 같은 모듈 내에서만 접근 가능  
 private: 같은 파일 내에서만 접근 가능 

 - class scope  
 public: 클래스 외부에서 늘 접근 가능  
 internal: 클래스 내부에서만 접근 가능  
 protected: 클래스 자신과 상속받은 클래스에서 접근 가능


#### 고차함수와 람다함수

- 고차함수: 다른 함수에 인자로 넘길 수 있는 함수. 함수를 클래스의 인스턴스처럼 사용할 수 있는 함수.
- 람다함수: 익명 함수. 람다식은 주로 고차 함수에 인자(argument)로 전달되거나 고차 함수가 돌려주는 결과값으로 쓰인다.

 고차함수와 람다함수의 의의: 함수를 변수로 사용할 수 있으며 이는 향후 컬렉션과 스코프 함수에도 도움 

```kotlin
// 람다 (=익명함수)식 안에 인자의 자료형 기재하여 타입추론 기능을 이용  
fun main() {
    b(::a) 
    val c = {str: String -> println("$str 람다함수")} 
}

fun a(str: String) {
    println("$str 함수 a")
}

fun b (function: (String) -> Unit) {
    function("b가 호출한") 
}
```
- 람다식에 여러 줄 입력 가능 
```kotlin
val c = {str: String -> 
	println("${str}람다함수")
    println("${str}익명함수였네")
    println("${str}단어에 쫄 필요없어")
}
```

#### 스코프 함수

- 인스턴스의 속성이나 함수를 scope 내에서 깔끔하게 분리하여 사용할 수 있게 만들어 코드의 가독성 향상

- apply
  - 인스턴스를 생성하고, 변수에 담기 전 초기화 과정을 수행할 때 사용. 인스턴스 자신을 다시 반환
  - main 함수와 별도의 scope에서 인스턴스의 변수와 함수를 조작하므로 코드가 깔끔해짐

- run
  - 인스턴스 생성 후, 인스턴스의 함수나 속성을 스코프 내에서 사용할 때 사용
  - 참조연산자를 사용하지 않아도 됨. 마지막 구문을 반환

- with
    - run과 같지만, 인스턴스를 인자로 받는다는 차이

- also (=apply) / let (=run)
    - 'it'을 통해서 인스턴스를 사용

```kotlin
fun main() {
	var price = 5000

	var a = Book("사샤의 코틀린", 30000).apply {
		name = "[초특가]" + name
		discount()
	}

	a.run {
		println("상품명: ${name}, 가격: ${price}원")
	} // 상품명: [초특가]사샤의 코틀린, 가격: 28000원

	with(a) {
		println("상품명: ${name}, 가격: ${price}원")
	} // 상품명: [초특가]사샤의 코틀린, 가격: 28000원

	a.let {
		println("상품명: ${it.name}, 가격: ${it.price}원")
	} // 상품명: [초특가]사샤의 코틀린, 가격: 28000원
}

class Book(var name: String, var price: Int) {
	fun discount() {
		price -= 2000
	}
}
// 같은 이름의 함수나 변수가 scope 바깥에 중복되어 있는 경우에 혼란을 방지 
```