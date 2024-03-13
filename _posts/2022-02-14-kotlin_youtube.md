---
layout: post
title: "📅 Kotlin 강의노트 - 30강까지"
excerpt: "디모의 Kotlin 강좌 요약본입니다 "
subtitle: "Kotlin youtube lecture"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-2-14
tags: [Kotlin]
---

#### DATA class & ENUM class 
- data class: 데이터를 관리하는데 최적화 된 class이며 5가지 기능을 내부적으로 자동 생성. 배열이나 리스트 등의 데이터 타입을 위해 제공되는 메서드
  - equals: 내용의 동일성을 판단
  - hashcode(): 객체의 내용에서 고유한 코드를 생성
  - toString(): 포함된 속성을 보기쉽게 나타냄
  - copy(): 객체를 복사하여 새 객체를 만들 수 있는 기능 -> 전달인자를 따로 넣어서 일부 속성 변경 가능
  - componentX(): 속성을 순서대로 내용을 반환
  
- enum(erated) class: 열거형 자료의 상태를 각각 나타내기 위한 방법, enum class 클래스 안에 객체들은 관행적으로 대문자로 기술


```kotlin
fun main() {
   	val list = listOf(Data("Sasha", 88),
                     Data("Lisa", 91),
                     Data("Coco", 18))

    for((a, b) in list) { //component1, component2
        println("${a}, ${b}")
    }
    
    var state = State.SING
    println(state)// SING
    
    state = State.SLEEP
    println(state.isSleeping())// true 
    
    state = State.EAT
    println(state.message)// 밥을 먹는다
}

data class Data(val name: String, val id: Int)

enum class State(val message: String) {
    SING("노래를 부른다"),
    EAT("밥을 먹는다"),
    SLEEP("잠을 잡니다"); // 마지막 객체에 세미콜론을 넣어주자
    
    fun isSleeping() = this == State.SLEEP
}
```

#### 컬렉션 클래스: Set & Map

- Set: 순서가 정열되어 있지 않고, 중복이 허용되지 않는 컬렉션. 컨텐츠가 set 안에 있는지 확인
- Map: 객체를 넣을 때 찾을 수 있게 key를 쌍으로 넣어주는 컬렉션

```kotlin
fun main() {
	val a = mutableSetOf("사샤", "리사", "코코")
    
    for(ele in a) {
        println(a)
    }
    
    a.add("발로쟈")
    a.remove("코코")
    a.contains("레라")
	
    val b = mutableMapOf(
    	"사샤" to "프로그래머",
        "리사" to "엔지니어",
        "코코" to "냐옹이") 
    
    for(ele in b) {
        println("${ele.key}:${ele.value}")
    }
    
    b.put("발로쟈", "파일럿")
    println(b) // [(사샤, 프로그래머), (리사, 엔지니어), (코코, 냐옹이), (발로쟈, 파일럿)]
    
    b.remove("프로그래머")
    println(b)
    
    println(b["리사"])
    
}
```

#### 컬렉션 함수 종류

- 컬렉션 함수: 람다함수를 사용하여 컬렉션을 좀 더 편리하게 조작할 수 있는 함수. JS의 고차 함수를 생각하자

```kotlin
fun main() {

  data class Person(val name: String, val birthYear: Int)
  val personList = listOf(Person("사샤", 1988),
    Person("리사", 1991),
    Person("코코", 2018))

  // associateBy: 아이템에서 key를 추출하여 map으로 변환하는 함수 
  println(personList.associateBy{ it.birthYear }) // {1988=Person(name=사샤, birthYear=1988), 1991=Person(name=리사, birthYear=1991), 2018=Person(name=코코, birthYear=2018)}

  // groupBy: key가 같은 아이템끼리 배열로 묶어 map으로 만드는 함수 
  println(personList.groupBy{ it.name }) // {사샤=[Person(name=사샤, birthYear=1988)], 리사=[Person(name=리사, birthYear=1991)], 코코=[Person(name=코코, birthYear=2018)]}

  // partition: 조건을 걸어 boolean 값에 따라 두 컬렉션으로 나누어주는 함수 
  val (over00, under00) = personList.partition { it.birthYear > 2000}
  println(over00) // [Person(name=코코, birthYear=2018)]
  println(under00) // [Person(name=사샤, birthYear=1988), Person(name=리사, birthYear=1991)]

  val numbers = listOf(1, 0 , -3, 14, 100)

  // flatMap: 아이템마다 만들어진 컬렉션을 합쳐서 반환하는 함수 
  println(numbers.flatMap{listOf(it*10, it+10, it/3)}) // [10, 11, 0, 0, 10, 0, -30, 7, -1, 140, 24, 4, 1000, 110, 33]

  // getOrElse: idx 위치에 아이템이 있으면 아이템을 반환하고 아닌 경우 지정한 기본값을 반환하는 함수
  println(numbers.getOrElse(1){ 50 }) // 0
  println(numbers.getOrElse(100) { 50 }) // 50

  // zip: 컬렉션 두 개의 아이템을 1:1로 매칭하여 새 컬렉션을 만들어 줌. 아이템의 개수가 적은 컬렉션이 우선됨  
  println(personList zip numbers) // [(Person(name=사샤, birthYear=1988), 1), (Person(name=리사, birthYear=1991), 0), (Person(name=코코, birthYear=2018), -3)]
}
```

#### 변수의 고급 기술, 상수, lateinit, lazy

- val은 할당된 객체를 바꿀 수 없을 뿐이지 객체 내부의 속성은 변경 가능
- 상수: 컴파일 시점에 결정되어 절대로 바꿀 수 없는 값이며, 기본 자료형만 선언 가능하며 런타임에 생성될 수 있는 일반적인 클래스 객체들은 담을 수 없음. 
대문자와 언더바만 사용
- 상수를 쓰는 이유: 변수의 경우 런타임 시 객체를 생성하기 떄문에 성능 하락 이슈 

```kotlin
fun main() {
    val foodCourt = FoodCourt()
    
    foodCourt.searchPrice(FoodCourt.FOOD_CREAM_PASTA) // 크림파스타의 가격은 13000원 입니다!
    foodCourt.searchPrice(FoodCourt.FOOD_STEAK) // 스테이크의 가격은 30000원 입니다!
    foodCourt.searchPrice(FoodCourt.FOOD_PIZZA)	// 피자의 가격은 15000원 입니다!
}

class FoodCourt {
  fun searchPrice(foodName: String) {
    val price = when (foodName) {

      FOOD_CREAM_PASTA -> 13000
      FOOD_STEAK -> 30000
      FOOD_PIZZA -> 15000
      else -> 0
    }
    println("${foodName}의 가격은 ${price}원 입니다!")
  }

  companion object {
    const val FOOD_CREAM_PASTA = "크림파스타"
    const val FOOD_STEAK = "스테이크"
    const val FOOD_PIZZA = "피자"
  }
}
```

- lateinit: 변수에 객체를 할당하는 것을 선언과 동시에 할 수 없을 때 사용, String 클래스에는 사용 가능하나 기본 자료형에는 사용 불가
- lateinit var 변수의 제한사항: 초기값 할당 전까지 변수를 사용할 수 없음 -> 에러 발생
- ::a.isInitialized: lateinit 변수 초기화 사용여부 확인하는 컬렉션 

```kotlin
fun main() {
	val a = LateInitSample()
    println(a.getLateInitText()) // 기본값
    
    a.text = "new assigned"
    println(a.getLateInitText()) // new assigned   
}

class LateInitSample {
    lateinit var text: String
    
    fun getLateInitText(): String {
        if(::text.isInitialized) {
            return text
        } else {
            return "기본값"
        }
    }
}
```

- 지연 대리자 속성(lazy delegate properties): 변수를 사용하는 시점까지 초기화를 자동으로 늦춰주는 기능, 선언 시 즉시 객체를 생성 및 할당하여 변수를 
초기화하는 형태를 갖고 있지만 실제 실행 시에는 변수 사용 시에 초기화가 진행됨. 

```kotlin
fun main() {
	val number: Int by lazy {
        println("초기화를 합니다")
        7
    }
    
    println("start the code") 
    println(number) // 초기화를 합니다, 7 -> 초기화 진행
    println(number) // 7 -> 이미 초기화가 진행되었으므로 람다 함수 lazy 작동x
	    
}
```

#### 비트연산

- 비트연산: 정수변수를 2진법으로 연산할 수 있는 기능. 정수형의 값을 비트단위로 나누어 데이터를 좀 더 작은 단위로 담아 경제성을 확보하는데 그 목적이 있음.
주로 플래그값(여러개의 상태값을 01과 1로 담는 방법)을 처리하거나 네트워크 프토로콜의 데이터 양을 줄이는데 이용됨
- 부호비트: 2진수로 전환된 값의 가장 앞부분 데이터이며 보통 부호를 나타냄

```kotlin
fun main() {
    
    var bitData: Int = 0b1000
    bitData = bitData or(1 shl 2)
    println(bitData.toString(2))
   
    var result = bitData and(1 shl 4) // shift left 부호비트를 제외한 나머지 비트들을 4칸 밀어주자
    println(result.toString(2)) // 2진수 형태로 전환하여 출력
    
    println(result shr 4) // shift right 4칸 밀어주자 
    
    bitData = bitData and((1 shl 4).inv()) // shl을 사용하여 1을 좌측으로 4번 밀어주자
    println(bitData.toString(2))
    
    println((bitData xor(0b10100)).toString(2))
}
```

#### 코루틴을 통한 비동기 처리

- 코루틴: 비동기처리, 메인 루틴과 별도로 진행이 가능한 루틴으로 개발자가 실행, 종료를 마음대로 조정
- 코루틴 scope
  - GlobalScope: 프로그램 어디서나 제어, 동작이 가능한 기본 범위
  - CoroutineScope: 특정한 목적의 디스패처를 지정하여 지정, 동작이 가능

- 루틴의 대기를 위한 추가적인 함수. 코루틴이나 runBlocking처럼 루틴 대기가 가능한 구문에서만 작동
  - delay(miliseconde: Long): ms 단위로 루틴 대기
  - Job.join(): Job의 실행이 끝날때까지 대기하는 함수
  - Deferred.await(): JS의 Promise 객체와 비슷하며 결과값도 반환

- cancel(): 코루틴 내부의 delay() or yield() 함수가 사용된 위치까지 수행된 뒤 종료되거나 isActive 속성이 false가 되는 것을 수동으로 확인 후 종료

```kotlin
import kotlinx.coroutines.*

fun main() {
	runBlocking { // 코루틴이 종료될때까지 메인 루틴을 잠시 대기. 안드로이드에서는 반응이 없을 경우 앱이 강제 종료 
    	val a = launch {
            for(ele in 1..5) {
                println(ele)
                delay(100)
            }
        }
         val b = async {
            "async 종료"
        }
         
        println("Hold async")
        println(b.await())
        
        println("launch 취소")
        a.cancel()
        println("Quit launch")
        
    }
}
```

- withTimeOutOrNull: 제한 시간 내에 수행되면 결과값을 반환하고 아니면 null값을 반환
```kotlin
import kotlinx.coroutines.*

fun main() {
	runBlocking { 
    	var result = withTimeoutOrNull(50) {
            for(ele in 0..10) {
                println(ele)
                delay(10)
            }
            "Finish"
        }
        println(result)
    }
}
```