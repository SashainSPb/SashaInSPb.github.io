---
layout: post
title: "Kotlin 컬렉션 정리"
excerpt: ""
subtitle: "API docs"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2023-01-20
tags: [Collection, Kotlin]
---

# Kotlin 컬렉션 정리

  여타 다른 프로그래밍 언어와 마찬가지로 코틀린에서도 컬렉션의 개념과 이에 대한 메서드를 지원하고 있습니다. 
  
  기존 JS에서 배열(Array)와 객체(Object)를 통해 컬렉션에 대한 개념에 대해서 인지하고 있었으나, 코틀린에서는 
  
  좀 더 기능이 세분화된 컬렉션 타입을 지원하고 있어 초반에 상당히 헤맸던 기억이 있네요. 이에 Kotlin 공식 
  
  문서 내 컬렉션 파트를 번역하고, 이해한 내용을 재 구성하여 본 페이지를 작성하였습니다.
  
  흔히, 컬렉션은 같은 종류의 원소 / 아이템을 여러 개 가지고 있는 자료형으로 정의할 수 있으며, 코틀린에서는
  
  다음과 같은 세가지 형태로 지원하고 있습니다.

- List: 순서를 갖고 있는 컬렉션이며, 원소는 순서를 나타내는 인덱스를 통해 접근 가능한 특성을 갖고 있습니다. 원소는 중복될 수 있으며, 순서는 중요합니다.

- Set: 각각 구분되는 유일한 원소를 갖고 있는 컬렉션입니다.. 중복은 있을 수 없으며, 순서는 중요하지 않습니다.

- Map(혹은 dictionary): JS의 객체와 비슷하며, 키-값 쌍으로 이루어진 컬렉션입니다. 키는 유일하며 값은 중복으로 가질 수 있습니다.

---

### 컬렉션 타입

   코틀린에서 구현된 기본적인 컬렉션 타입은 다음과 같이 두가지 특징을 지원하고 있습니다.

  - read-only 인터페이스: 오직 데이터 조회만 가능
  - mutable 인터페이스: 컬렉션 내 쓰기를 가능케 함

  mutable 컬렉션을 변경할 때 val 를 써도 변경 가능함. 단, 재 할당은 불가.

  ```kotlin
  val numbers = mutableListOf("one", "two", "three", "four")
  numbers.add("five")   // this is OK
  println(numbers)
  //numbers = mutableListOf("six", "seven")      // compilation error
  ```

  read-only 컬렉션은 공변성을 가지고 있는데요, 부모 클래스에서 자식 클래스로 상속했을때, 

  List<부모 클래스>가 요구되는 곳에서 List<자식 클래스>를 사용 할 수 있습니다. 다시 말해, 컬렉션은 

  해당 컬렉션의 요소와 동일한 서브 타입관계를 가지고 있어요. Map의 경우 키가 아닌 값과 공변성을 

  가지고 있는 특성을 지닙니다.

  반대로, mutable 컬렉션은 공변성을 가지고 있지 않으며, 공변성을 갖게 된다면 런타임 에러를 

  발생시킵니다. 만일 MutableList<자식 클래스>가 MutableList<부모 클래스>의 서브 타입이라면, 

  부모 클래스를 상속받는 다른 자식 클래스를 넣을 수 있어요. 이는 명시된 타입을 위반하여 에러를 뱉을 것입니다.

  아래는 코틀린 컬렉션 인터페이스이며, 각 구현체에 대해서 좀 더 자세히 알아보겠습니다. 

  ![Untitled](https://1drv.ms/i/c/cdf0612ba3befd25/IQNJm1thM5aQQ5-ZejJ6mWgxAXz8fSbQjzynashPkcOvQdA?width=660)

---

### Collection<T>

   Collection<T>는 컬렉션 계층의 기본이며, **read-only**한 특성을 지닙니다. 또한 Iterable<T> 인터페이스를 상속
  
  받았기 때문에 차례대로 요소에 접근할 수 있는 특성 즉, 순환이 가능함. List와 Set은 Collection을 상속받음.

  ```kotlin
    fun printAll(strings: Collection<String>) {
      for(s in strings) print("$s ")
      println()
    }
  
    fun main() {
      val stringList = listOf("one", "two", "one")
      printAll(stringList) // one two one
  
      val stringSet = setOf("one", "two", "three")
      printAll(stringSet)// one two three
    }
  ```

  MutableCollection<T>는 add와 remove 메서드가 추가된 컬렉션입니다.

  ```kotlin
    fun List<String>.getShortWordsTo(shortWords: MutableList<String>, maxLength: Int) {
      this.filterTo(shortWords) { it.length <= maxLength }
      val articles = setOf("a", "A", "an", "An", "the", "The")
      shortWords -= articles
    }
  
     fun main() {
       val words = "A long time ago in a galaxy far far away".split(" ")
       val shortWords = mutableListOf<String>()
       words.getShortWordsTo(shortWords, 3)
       println(shortWords) // [ago ,in, far, far]
     }
  
  ```

### List<T>

   리스트는 순서를 갖고 있는 요소의 컬렉션이며, 각 요소는 순서가 구분되는 다덱스를 갖고 있습니다.  

  nullable 한 리스트의 요소들은 중복될 수 있고, 같은 사이즈와 구조적으로 동일한 요소가 같은 

  위치에 있는 두 List의 경우 동일한 것으로 간주됩니다.

  ```kotlin
    val bob = Person("Bob", 31)
    val people = listOf(Person("Adam", 20), bob, bob)
    val people2 = listOf(Person("Adam", 20), Person("Bob", 31), bob)
    println(people == people2) // true
    bob.age = 32
    println(people == people2) // false
  ```

 MutableList<T>는 add와 remove 메서드가 추가된 컬렉션입니다.

  ```kotlin
  val numbers = mutableListOf(1, 2, 3, 4)
  numbers.add(5)
  numbers.removeAt(1)
  numbers[0] = 0
  numbers.shuffle()
  println(numbers) // 4,5,0,3을 요소로 갖는 List
  
  ```

  언뜻 배열과 가장 유사한 것으로 여겨지나 큰 차이가 있습니다. 배열의 경우, 초기화 당시에 사이즈가 정의되고 바뀌지 않지만, 
  
  List의 경우 사이즈가 따로 고정되지 않습니다. 코틀린에서는 MutableList의 구현체는 ArrayList이니 참고하세요.


### Set<V>

   Set은 고유한 요소를 저장하며, 순서는 따로 정해져 있지 않습니다. null 또한 고유하게 취급되며, 오직 하나의 null만 
  
  포함할 수 있습니다. List와는 다르게 같은 사이즈와 동일한 요소를 갖고 있는 두 Set 은 같은 것으로 취급됩니다.

  ```kotlin
    val numbers = setOf(1, 2, 3, 4)
    println("Number of elements: ${numbers.size}") // 4
    if (numbers.contains(1)) println("1 is in the set") // 1 is in the set
  
    val numbersBackwards = setOf(4, 3, 2, 1)
    println("The sets are equal: ${numbers == numbersBackwards}") // The sets are equal: true}
  ```

  MutableSet<T>은 add와 remove 메서드가 추가된 컬렉션임. 기본 구현체는 LinkedHashSet 이며,  

  List와 마찬가지로 요소의 삽입 순서를 보존합니다.

  ```kotlin
    val numbers = setOf(1, 2, 3, 4)  // LinkedHashSet is the default implementation
    val numbersBackwards = setOf(4, 3, 2, 1)
  
    println(numbers.first() == numbersBackwards.first()) // false
    println(numbers.first() == numbersBackwards.last()) // true
  ```

  대체 구현체인 HashSet은 요소 순서를 보장하지 않지만, LinkedHashSet에 비해 메모리 이용률이 낮는 특징을 갖고 있습니다.

### Map<K, V>

   Map<K, V>는 컬렉션의 상속자는 아니지만, 코틀린 컬렉션 타입 중 하나입니다. Map은 키-값 형태로 데이터를 

  저장하며, 키는 고유하지만 값은 중복이 가능한 특징을 지님. Map 인터페이스는 키-값을 이용한 유용한 메서드를 제공합니다.  
  
  동일한 키와 값을 가지고 있는 두 Map은 같은 것으로 간주됩니다.

  ```kotlin
    val numberMap = mpaOf("1" to 1, "2" to 2, "3" to 3, "4" to 4)
    println("All keys : ${numberMap.key}") // ["1", "2", "3", "4"]
    println("All values: ${numberMap.values}") [1, 2, 3, 4]
  
    if ("2" in numberMap) println("Value by key \\"key2"\\: ${numbersMap["key2"]}") // "Value by key "2": 2
    if (1 in numbersMap.values) println("The value 1 is in the map") // The value 1 is in the map
    if (numbersMap.containsValue(1)) println("The value 1 is in the map") // The value 1 is in the map
  
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
    val anotherMap = mapOf("key2" to 2, "key1" to 1, "key4" to 1, "key3" to 3)
  
    println("The maps are equal: ${numbersMap == anotherMap}") // The maps are equal: true
  ```

  MutableMap<K, V>은 add와 remove 메서드가 추가된 컬렉션입니다. 기본 구현체는 LinkedHashMap이며, 

  마찬가지로 Map 순환 시 요소 삽입 순서를 보존하는 특성을 지니고 있습니다. HashMap은 순서를 보존하지 않습니다.

  ```kotlin
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap.put("three", 3)
    numbersMap["one"] = 11
  
    println(numbersMap) // { "one": 11, "two": 2, "three": 3}
  
  ```

---

## Ref.

[https://kotlinlang.org/docs/collections-overview.html](https://kotlinlang.org/docs/collections-overview.html)
