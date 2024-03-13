---
layout: post
title: "📅 Kotlin in Action - 8장 고차 함수: 파라미터와 반환 값으로 람다 사용"
excerpt: "Kotlin in Action 8장 요약 노트입니다."
subtitle: "Kotlin in Action"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-4-24
tags: [Kotlin]
---

>목표: 고차 함수로 코드를 더 간결하게 다듬고 코드 중복을 없애는 등 더 나은 추상화를 구축하는 방법을 배워보자.


### 8.1 고차 함수 정의 

  - 고차 함수: 다른 함수를 인자로 받거나 함수를 반환하는 함수, 코틀린에서는 람다나 함수 참조를 인자로 넘기거나 반환할 수 있음

#### 8.1.1 함수 타입

  - 컴파일러는 sum과 action이 함수 타입임을 추론함 

    ```kotlin
        val sum: (Int, Int) -> Int = {x, y -> x + y} // Int 인자 2개를 받아서 Int 값을 반환 
        val action: () -> Unit = {println(42)} // No 인자, No 리턴 함수
    ```

  - 함수 타입을 정의하려면 함수 인자의 타입을 괄호 안에 넣고, 그 뒤에 화살표를 추가한 다음, 함수의 반환 타입을 지정
  - Unit 타입은 의미 있는 값을 반환하지 않음

    ```kotlin
    (Int, String) -> Unit 
    ```

  - 함수 타입에서도 반환 타입을 nullable 타입으로 지정 가능
  - 마찬가지로 nullable 함수 타입 변수 정의 가능. 단, 함수 타입 전체가 nullable 선언하기 위해 함수 타입을 괄호로 감싸고 물음표 기재

    ```kotlin
    var canReturnNull: (Int, Int) -> Int? = {x, y -> null}
    var funOrNull: ((Int, Int) -> Int)? = null
    ```

  - 인자명은 타입 검사 시 무시되며, 람다 정의 시, 인자명이 함수 타입 선언의 파라미터 이름과 일치하지 않아도 됨 

    ```kotlin
    fun performRequest(
        url: String,
        callback: (code: Int, content: String) -> Unit
    ) {/*...*/}
    
    val url = "https://2bytescorp.com"
    performRequest(url) {code, page -> /*...*/} // 2번째 인자명이 page로 설정되도 상관x
    ```

#### 8.1.2 인자로 받은 함수 호출 

  - 함수 이름 뒤에 괄호를 붙이고 괄호 안에 원하는 인자를 comma로 구분해 넣어주자

    ```kotlin
    fun twoAndThree(operation: (Int, Int) -> Int) {
        val result = operation(2, 3)
        println("$result")
    }
    twoAndThree{a,b -> a+b} // 5
    twoAndThree{a,b -> a*b} // 6
    ```

  - filter 메서드 구현하기
    ```kotlin
    fun String.filter(predicate: (Char) -> Boolean): String
    /*  / 수신 객체 타입 /  인자명  / 인자함수의 인자타입 / 인자함수의 반환 타입/   */
    
    // filter 함수를 단순하게 만든 버전 구현하기
    fun String.filter(predicate: (Char) -> Boolean): String {
        val sb = StringBuilder()
        for (i in 0 until length) {
            val element = get(i)
            if (predicate(element)) sb.append(element) // predicate 인자로 전달받은 get()를 호출 
        }
        return sb.toString()
    }
    
    println("ablc".filter {it in 'a'..'z'}) // abc
    ```

#### 8.1.3 자바에서 코트린 함수 타입 사용 (pending)

  - 함수 타입의 변수는 FunctionN 인터페이스를 구현하는 객체를 저장
  
#### 8.1.4 디폴트 값을 지정한 함수 타입 인자나 nullable 함수 타입 인자

  - 인자를 함수 타입으로 선언할 때도 디폴트 값을 정하기 가능
  - nullable 함수 타입 인자 사용하기

    ```kotlin
    fun <T> Collection<T>.joinToString ( // 제네릭 함수
            separator: String = ", ",
            prefix: String = "",
            postfix: String = "",
            transform: ((T) -> String)? = null // nullable 함수 타입의 인자를 선언
    ): String {
        val result = StringBuilder(prefix)
        for((i, e) in this.withindex()) {
            if (i > 0) result.append(separator)
            var str = transform?.invoke(e) // 안전 호출을 사용해 함수 호출
                    ?: e.toString() // 엘비스 연산자를 사용해 람다를 인자로 받지 않은 경우를 처리
            result = append.append(str)
        }
        result.append(postfix)
        return result.toString()
    }
    
    val letters = listOf("Alpha", "Beta")
    println(letters.joinToString()) // Alpha, Beta
    println(letters.joinToString { it.toLowerCase() }) // alpha, beta
    println(letters.joinToString(separator = "! ", postfix = "! ", 
            ...transform = {it.toLowerCase()})) // ALPHA!, BETA!
    ```

#### 8.1.5 함수를 함수에서 반환

  - 프로그램의 상태나 다른 조건에 따라 달라질 수 있는 로직 구현

    ```kotlin
    enum class Delivery {STANDARD, EXPEDITED} 
    class Order(val itemCount: Int) 
    fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double { // Order 함수를 받아 Double 함수를 반환
        if (delivery == Delivery.EXPEDITED) {
            return {order -> 6 + 2.1 * order.itemCount} // 람다 반환1
        }
        return {order -> 1.2 * order.itemCount} // 람다 반환2
    }
    
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    println("Shipping costs ${calculator(Order(3))}") // Shipping costs 12.3
    ```

  - 사용자의 UI 입력 창에 입력한 문자열과 매치되는 연락처만 화면에 표시하는 로직 구현.

    ```kotlin
    data class Person(
        val firstName: String,
        val lastName: String, 
        val phoneNumber: String?
    )
    
    class ContactListFilters {
        var prefix: String = ""
        var onlyWithPhoneNumber: Boolean = false 
        
        fun getPredicate(): (Person) -> Boolean { // 함수를 반환하는 함수 정의
            val startsWithPrefix = { p: Person ->
                p.firstName.startsWith(prefix) || p.lastName.startsWith(prefix)
            }
            if (!onlyWithPhoneNumber) { // 함수 타입의 변수를 반환
                return startsWithPrefix
            }
            return { startsWithPrefix(it) && it.phoneNumber = null } // 람다 반환
        }
    }
    
    val contacts = listOf(Person("Sasha", "Park", "123-4567"), Person("Lisa", "Park", "321-3212"))
    val contactListFilters = ContactListFilters()
    with (contactListFilters) {
        prefix = "Sa"
        onlyWithPhoneNumber = true
    }
    println(contacts.filter( ... contactListFilters.getPredicate()))
    // [Person(firstName=Sasha, lastName=Park, phoneNumber=123-4567)]
    ```

#### 8.1.6 람다를 활용한 중복 제거 

  - 사이트 방문 데이터 정의 

    ```kotlin
    data class SiteVisit (
        val path: String,
        val duration: Double,
        val os: OS
    )
    enum class OS {WINDOWS, LINUX, MAC, IOS, ANDROID}
    val log = listOf (
        SiteVisit("/", 34.0, OS.WINDOWS)
        SiteVisit("/", 3.0, OS.MAC),
        SiteVisit("/login", 12.0, OS.IOS),
        SiteVisit("/signup", 76.0, OS.ANDROID),
        SiteVisit("/", 24.0, OS.LINUX)
    )
    
    val averageWindowsDuration = log
        .filter {it.OS == OS.WINDOWS}
        .map(SiteVisit::duration)
        .average()
    
    println(averageWindowsDuration) // 34.0
    ```

  - 고차 함수를 통해 중복 제거

    ```kotlin
    fun List<SiteVisit>.averageDurationFor (predicate: (SiteVisit) -> Boolean =
    filter(predicate).map(SiteVisit::duration).average()
    )
    
    println(log.averageDurationFor {
        ... it.os in setOf(OS.ANDROID, OS.IOS)
    }) // 12.15
    
    println(log.averageDurationFor {
        ... it.os == OS.IOS && it.patj == "/signup"
    }) // null
    ```

### 8.2 인라인 함수: 람다의 부가 비용 없애기

  - 람다가 생성되는 시점마다 새로운 무명 클래스 객체 생기고 실행 시점에 따른 부가 비용이 듦
  - inline 변경자를 함수에 붙이면 컴파일러는 그 함수를 호출하는 문장을 함수 본문에 해당하는 바이트 코드로 변환하여 좀 더 효율적인 사용이 가능

#### 8.2.1 인라이닝이 작동하는 방식

  - inline: 함수 호출 코드를 함수 호출 바이트코드 대신에 함수 본문을 번역한 바이트 코드로 컴파일
  - 람다 본문에 의해 생성되는 바이트코드는 람다 호출코드(synchronized) 일부분으로 간주 -> 무명 클래스로 감싸지 않음, 즉 성능 향상이 있음

    ```kotlin
    inline fun <T> synchronized(lock: Lock, action: () -> T): T {
        lock.lock()
        try {
            return action()
        } finally {
            lock.unlock()
        }
    }
    val 1 = lock()
    synchronized(1)
    ```

#### 8.2.2 인라인 함수의 한계 

  - 인라인 함수의 본문에서 람다 식을 바로 호출하거나 람다 식을 인자로 전달받아 호출하느 경우 이외에는 람다 인라이닝 불가 -> "Illegal usage of inline-parameter"
  - 둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝 적용하고 싶을 시, noinline 변경자를 인자 이름 앞에 붙임

    ```kotlin
    inline fun foo (inlined: () -> Unit, noinline NotInlined: () -> Unit) { ... }
    ```

#### 8.2.3 컬렉션 연산 인라이닝

    ```kotlin
    // 람다를 사용해 컬렉션 거르기
    data class Person(val: Nameval : String, val age: Int)
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.filter {it.age < 30}) // Person(name=Alice, age=29)
    
    // 람다식을 사용하지 않고 컬렉션 거르기
    val result = mutableListOf<Person>()
    for (person in people) {
        if (person.age < 30) result.add(person)  
    }
    println(result) // Person(name=Alice, age=29)
    ```

#### 8.2.4 함수를 인라인으로 선언해야 하는 경우

  - 람다를 인자로 받는 함수만 (ex. 표준라이브러리 함수) 성능이 좋아질 가능성이 높음 
  - inline 변경자를 함수에 부팅ㄹ 때는 코드 크기에 주의를 기울일 것 

#### 8.2.5 자원 관리를 위해 인라인된 람다 사용

  - 자원 (ex. 파일, 락, 데이터베이스 트랜잭션 등) 관리 시, 람다로 중복을 없앨 수 있는 메인 패턴임
  - 보통 try/finally 문을 사용하여 자원 관리 패턴을 구현 
    
    ```kotlin
    // 자바 try-with-resource 기능을 제공하는 use 라이브러리 함수
    fun readFirstLineFromFile(path: String) : String {
        BufferedReader(FileReader(path)).use { // BufferedReader 개체를 만들고, use 함수 호출을 통해 람다로 넘김
            br -> return br.readLine() // 
        }
    }
    ```

### 8.3 고차 함수 안에서 흐름 제어
#### 8.3.1 람다 안의 return문: 람다를 둘러싼 함수로부터 반환 

  - 람다 안에서 return을 사용하면 람다로부터만 반환되는게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환
  - non-local return: 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 블록을 반환하게 만드는 return 문
  - 람다를 인자로 받는 함수가 인라인 함수일때만 non-local return 사용 가능 

    ```kotlin
    data class Person(val name: String, val age: Int)
    val people = listOf(Person("Sasha", 33), Person("Lisa", 30))
    
    fun lookForSasha(people: List<Person>) {
        people.forEach { 
            if (it.name == "Sasha") {
                println("Found!")
                return 
            }
        }
        println("Found!")
    }
    ```

### 8.3.2 람다로부터 반환: 레이블을 사용한 return 

  - 람다 식에서도 local return 사용 가능 -> for 루프의 break와 비슷한 역할 
  - local / non-local return 구분을 위해 label@ 사용 
  - 람다 식의 레이블을 명시하면 함수 이름 레이블로 사용 불가, 또한 람다 식에는 레이블이 2개 이상 붙을 수 없음

    ```kotlin
    fun lookForSasha(people: List<Person>) {
        people.forEach label@ { // 람다 label
            if (it.name == "Sasha") {
                println("Found!")
                return@label // return 식 label, 여기서 loop 탈출
            }
        }
        println("Found!") // 항상 해당 줄이 출력
    }
    ```

> #### label이 붙은 this 식  
> 수신 객체 지정 람다 본문에서는 this 참조를 사용해 람다를 만들 때 지정한 수식 객체를 가리킬 수 있음  
    
    ```kotlin
    StringBuilder().apply sb@ { // this@sb를 통해 람다의 묵시적 수신 객체에 접근할 수 있음
        listOf(1, 2, 3).apply { 
            this@sb.append(this.toString())
        }
    }
    ```

### 무명 함수: 기본적으로 로컬 return

  - 장황한 non-local 반환문을 여럿 사용해야 코드 블록을 쉽게 작성하는 방법
  - 무명 함수는 코드 블록을 함수에 넘길 때 사용할 수 있는 다른 방법

    ```kotlin
    fun lookForSasha (people: List<Person>) {
        people.forEach(fun(person) { // 람다 식 대신에 무명함수 사용
            if (it.name == "Sasha") return // return은 가장 가까운 상기 무명함수를 가리킴 
            println("${person.name} is not Sasha!")
        })
    }
    lookForSasha(people) // Sasha is not Sasha!
    ```

  - 무명 함수도 일반 함수와 같은 반환 타입 지정 규칙을 따름 
  - 블록이 분문이 무명 함수는 반환 타입을 명시해야하지만, 식을 본문으로 하는 경우 반환 타입 생략 가능

    ```kotlin 
    people.filter(fun(person):Boolean {
        return person.age < 30
    }) 
    
    people.filter(fun(person) = person.age < 30)
    ```

  - fun 키워드로 정의된 함수를 반환시키는 return 식

    ```kotlin
    fun lookForSasha (people: List<Person>) {
        people.forEach(fun(person) {
            if (person.name == "Sasha") return // return은 가장 가까운 무명함수를 반환함
        })
    }
    
    fun lookForSasha (people: List<Person>) {
        people.forEach {
            if (it.name == "Sasha") return // return은 lookForSasha을 반환
        }
    }
    ```

