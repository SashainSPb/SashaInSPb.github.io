---
layout: post
title: "📅 Kotlin 강의노트 - 20강까지"
excerpt: "디모의 Kotlin 강좌 요약본입니다 "
subtitle: "Kotlin youtube lecture"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
date: 2022-1-18
tags: [Kotlin]
---

#### Object
 
  - Singleton Pattern: 클래스의 인스턴스를 단 하나만 만들어 사용하는 아키텍쳐 패턴
  - 오브젝트로 선언된 객체는 최초 사용 시 자동으로 생성
  - 따라서, 공통적으로 사용할 수 있는 공용 속성 및 함수를 묶어서 생성  
  
  - Companion Object: 기존 class 안에 생성하여 인스턴스 간 공용할 수 있는 속성과 함수를 별도 생성
  - Static Member와 비슷 -> 공용으로 사용가능한 속성이나 함수

    ```kotlin
    fun main() {
        var a = FoodPoll("짜장")
        var b = FoodPoll("짬뽕")
        
        a.vote()
        a.vote()
        a.vote()
        a.vote()
        b.vote()
        
        println("짜장면의 카운트 수: ${a.count} \n 짬뽕의 카운트 수: ${b.count}")
    }
    
    class FoodPoll (val name: String) {
        companion object {
            var total = 0
        }
      
        var count = 0
      
        fun vote() {
            total ++
            count ++
        }
    }
    ```

#### 옵저버 패턴

- 옵저버 패턴: 이벤트 리스너랑 같으며 구현 시 두 개의 클래스가 필요. 이벤트를 수신하는 클래스, 이벤트의 발생 
및 전달을 담당하는 클래스임 
- 두 클래스 사이에 인터페이스를 삽입, 이 인터페이스를 옵저버, 리스너라고 부름. 이벤트를 넘겨주는 행위를 callback이라고 함

```kotlin
fun main() {
	EventPrinter().start()
    
}

// 이벤트 리스너, 클래스 두개를 연결함
interface EventListner {
    fun onEvent(count: Int)
} 

// 이벤트를 수신하는 클래스
class Counter(var listener: EventListner) {
   fun count() {
       for(i in 1..100) {
           if(i % 5 == 0) listener.onEvent(i)
       }
   }
}

// 이벤트의 발생 및 전달 담당하는 클래스
// class EventPrinter {

//     fun start() {
//         val counter = Counter(object: EventListner {
//             override fun onEvent(count: Int) {
//                 print("${count}-")
//             }
//         })
//         counter.count()
//     }
// }

// 익명객체를 이용한 리팩토링 
class EventPrinter: EventListner {
    override fun onEvent(count: Int) {
        print("${count}-")
    }
    fun start() {
        val counter = Counter(this)
        counter.count()
    }
}
```
#### polymorphysm(다형성)

- 클래스를 상속하다보면 하위 클래스에서 상위 클래스와 똑같은 이름의 프로퍼티와 메서드를 지정할 일이 생김. **하위 클래스에서 이름은 같지만 호출 
매개변수가 다르거나 전혀 다른 동작의 메서드를 정의.** 클래스의 상속관계에서 오는 인스턴스의 호환성을 적극 활용할 수 있는 기능임
- up/down casting: 하위 인스턴스를 수퍼클래스로 변환하는 행위. 반대의 경우 down-casting이며 as, is 연산자 필요
- as: 변수를 호환되는 자료형으로 변환 후 반환까지 해주는 캐스팅 연산자

    ```kotlin
    var a: Drink = Cola()
    a as Cola
    ```
- is: 변수가 자료형에 호환되는지를 먼저 체크한 후에 변환해주는 연산자. 조건문과 같이 쓰임.

    ```kotlin
    var a:Drink = Cola()
    if(a is Cola) {
    "이 안에서만 a가 콜라"
    }
    ```

```kotlin
fun main() {
   
   var a = Drink() 
   var b: Drink = Cola()
   
   a.drink()
   b.drink()
   
   if(b is Cola){
     b.washDishes()
	 }
   
   var c = b as Cola
   c.washDishes()
   // as를 쓰면 변수 자체도 다운그레이드
   b.washDishes() 
}

open class Drink{
    var name = "음료"
    
    open fun drink() {
        println("${name}을 마십니다")
    }
}

class Cola: Drink() {
    var type = "콜라"
    
    override fun drink() {
        println("${name}중에 ${type}를 마십니다")
    }
    
    fun washDishes() {
        println("${type}로 목욕을 합니다")
    }
}
```