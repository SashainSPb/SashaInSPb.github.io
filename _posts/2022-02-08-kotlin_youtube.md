---
layout: post
title: "📅 Kotlin 강의노트 - 20강까지"
excerpt: "디모의 Kotlin 강좌 요약본입니다 "
subtitle: "Kotlin youtube lecture"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-2-8
tags: [Kotlin]
---

#### Generic

- 클래스나 함수에서 사용하는 자료형을 외부에서 지정할 수 있는 기능
> 캐스팅 이용 시 프로그램 속도를 제한시킬 수 있음

```kotlin
fun main() {
    UsingGeneric(A()).doShouting() // Sasha는 코틀린에 대해서 더 알고 싶어요!
    UsingGeneric(C()).doShouting() // Lisa는 인공지능을 더 공부합니다!
    
    doShouting(A()) // 캐스팅 없이 A의 객체 그대로 함수에서 사용
}

fun <T: A> doShouting(t: T) {
    t.shout()
}

open class A {
    open fun shout() {
        println("Sasha는 코틀린에 대해서 더 알고 싶어요!") 
    }
}

class C: A() {
    override fun shout() {
        println("Lisa는 인공지능을 더 공부합니다!") 
    }
}

class UsingGeneric<T: A> (val t: T) { // 수퍼 클래스를 A로 제한한 제너릭 T를 선언
    fun doShouting() {
        t.shout()
    }
}
```

#### 리스트

- List: 데이터를 코드에서 지정한 순서대로 저장해두는 클래스. 데이터를 모아 관리하는 컬렉션 클래스의 서브 클래스 중 가장 단순한 형태.
- 리스트의 2가지 형태  
  List<out T>: 생성 시에 넣은 객체를 대체, 추가, 삭제 할 수 없음
  MutableList<T>: 가능

```kotlin
fun main() {
	val family = listOf("사샤", "리사", "코코")

	println(family) // [사샤, 리사, 코코]

	for(ele in family) {
		println("${family}")
	}

	val b = mutableListOf(1, 2, 3)
	b.add(3, 5) // add(삽입하고자 하는 idx, value)
	println(b) // [1, 2, 3, 5]

	b.removeAt(3)
	println(b) // [1, 2, 3]

	b.shuffle()
	println(b) // 랜덤한 값이 반화

	b.sort()
	println(b) // [1, 2, 3]
}
```

#### 문자열 함수 종류

```kotlin
fun main() {
    val test1 = "Test.Kotlin.String"
    println(test1.length) // 18
    println(test1.lowercase()) // test.kotlin.string
    println(test1.uppercase()) // TEST.KOTLIN.STRING
    
    val test2 = test1.split(".")
    println(test2) // [Test, Kotlin, String]
    println(test2.joinToString()) // Test, Kotlin, String
    println(test2.joinToString("-")) // Test-Kotlin-String
    
    println(test1.substring(0..5)) // Test.K
    
    val nullString: String? = null
    val emptyString = ""
    val blankString = " "
    val normalString = "A"
    
    println(nullString.isNullOrEmpty()) // true, blank는 비어있는 것으로 취급X
    println(emptyString.isNullOrEmpty()) // true
    println(blankString.isNullOrEmpty()) // false
    println(normalString.isNullOrEmpty()) // false

    println(nullString.isNullOrBlank()) // true
    println(emptyString.isNullOrBlank()) // true
    println(blankString.isNullOrBlank()) // true, blank 상태도 비어있는 것으로 취급
    println(normalString.isNullOrBlank()) // false
  
    val test3 = "Sasha"
    val test4 = "Lisa"
    
    println(test3.startsWith("Sa")) // true
    println(test4.endsWith("Sa")) // false
    println(test4.contains("sa")) // true
}
```

#### null 값을 처리하는 방법? 동일한지를 확인하는 방법?

- null 상태로 속성이나 함수를 쓰려고 하면 null pointer exception(null 객체를 참조하면 발생하는 오류)이 발생
- null check가 없이는 컴파일 되지 않음

- "?.": null safe operator, 참조연산자 실행 전, 객체가 null인지 확인부터하고 객체가 null이라면 뒷구문을 실행X
- "?:": elvis operator, 객체가 null이 아니라면 그대로 실행하며 반대일 경우 우측의 객체로 대체
- "!!.": non-null assertion operator, 참조연산자 사용 시 null 여부를 컴파일 시 확인하지 않도록 하여 런타임 시 null pointer
  exception이 나도록 일부로 방치하는 연산자

```kotlin
fun main() {
    val a: String? = null
    println(a?.uppercase()) // null
    println(a?:"default".uppercase()) // DEFAULT
    println(a!!.uppercase()) // Exception in thread "main" java.lang.NullPointerException
}
```

- null check를 위해 if문 대신 연산자 + scope 함수 사용하면 편리

```kotlin
fun main() {
    var a: String? = null
    a?.run { // null을 체크하기 위해 스코프함수 사용
        println(uppercase())
        println(lowercase())
    }
}
```
- 내용의 동일성: heap 상에 주소가 달라도 내용이 같다면 두 객체는 같음 a == b
- 객체의 동일성: 서로 다른 변수가 메모리 상에 동일한 객체를 가르킬 때 a === b

```kotlin
fun main() {
    val a = Product("콜라", 1000)
    val b = Product("콜라", 1000)
    var c = a
	var d = Product("사이다", 500)    
    
    println(a == b) // true
    println(a === b) // false 
    
    println(a == c) // true
    println(a === c) // true
    
    println(a == d) // false
    println(a === d) // false

}

class Product(val name: String, val price: Int) {
    override fun equals(other: Any?): Boolean {
        if (other is Product) {
    		return other.name == name && other.price == price 
        } else {
            return false
        }	
    }
}
```

#### 중첩 클래스와 내부 클래스

- 중첩 클래스와 내부 클래스는 클래스 간의 연계성을 표현하여 코드의 가독성 및 작성 편의성을 증대하는데 목적이 있음
- 중첩 클래스: 클래스 서로 간 강하게 연관되어 있다는 의미를 전달하기 위해 만들어진 형식
- 내부 클래스: 혼자서 객체를 만들 수는 없고, 외부 클래스의 객체가 있어야만 생성과 사용이 가능

```kotlin
fun main() {
  Outer.Nested().introduce() // "Nested Class"

  val outer = Outer()
  val inner = outer.Inner()

  inner.introduceInner() // "Inner Class"
  inner.introduceOuter() // "Outer Class"

}

class Outer {
  var text = "Outer Class"

  class Nested() {
    fun introduce() {
      println("Nested Class")
    }
  }

  inner class Inner {
    var text = "Inner Class"

    fun introduceInner() {
      println(text)
    }

    fun introduceOuter() {
      println(this@Outer.text)
    }
  }
}
```